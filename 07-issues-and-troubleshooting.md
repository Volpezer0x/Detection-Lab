# 🧯 Troubleshooting & Lessons Learned – Detection Lab

This section documents **every issue encountered from initial lab setup to final Splunk field validation**, including misconfigurations, incorrect assumptions, tooling gaps, and parsing problems. Each issue includes symptoms, failed attempts, diagnosis steps, fixes, and lessons learned.

The goal is to help others **recognize the same failure patterns quickly** and avoid repeating the same mistakes. 

⚠ Note that I'm using both the Linux terminal as well as Powershell on Windows.

---

## ❌ ISSUE 1 — Nmap Scan Shows All Ports “Ignored / Filtered”

### Symptom
From Kali Linux:
```bash
nmap 192.168.20.10
```
Result: 
```
All 1000 scanned ports on 192.168.20.10 are in ignored states
```
## Initial Assumption (Incorrect)

- Network routing issue

- VMs not on the same network

## What I Tried That Did NOT Work

- Re-running scans repeatedly

- Changing Nmap flags blindly

- Assuming VirtualBox networking was broken

## Diagnosis

- Windows VM was reachable

- Windows Firewall was silently dropping packets

- No ICMP or TCP RST responses → Nmap marks ports as “filtered”

## Fix

Temporarily disabled Windows Firewall to confirm the hypothesis.

## Lesson Learned

“Filtered” in Nmap usually means silent packet drops, not lack of connectivity.

---

## ❌ ISSUE 2 — Nmap Works Only When Windows Firewall Is Disabled

### Symptom
Firewall OFF:
```bash
nmap -Pn 192.168.20.10
```
Open ports detected Firewall ON:

All ports filtered again

## What Didn’t Work

- Changing scan types alone

- Running scans as root vs non-root

## Diagnosis

Windows Firewall blocks:

- ICMP echo

- TCP SYN probes

- OS fingerprinting traffic

## Fix

Explicitly allow inbound traffic instead of disabling the firewall:
```powershell
New-NetFirewallRule -DisplayName "Allow Test Port" `
  -Protocol TCP `
  -LocalPort 8050 `
  -Direction Inbound `
  -Action Allow
```

## Lesson Learned

Firewalls don’t “block ports” — they block packet states and probes.

---

## ❌ ISSUE 3 — Netcat Works but Nmap Does Not

### Symptom

- Netcat succeeds:
```bash
nc 192.168.20.10 8050
```
- Nmap still reports all ports filtered

## Confusion

In my mind I was thinking if Netcat works, Nmap should work.

## Diagnosis

- Netcat connects to a specific allowed port

- Nmap probes many ports and packet types

- Firewall allows one, drops the rest

## Fix

Understanding expected behavior.

## Lesson Learned

Different tools generate very different traffic patterns.

---

## ❌ ISSUE 4 — PowerShell Firewall Command Fails

### Symptom
```powershell
Set-NetFirewallProfile -AllowInboundEchoRequest $true
```
Error:
```
A parameter cannot be found that matches parameter name 'AllowInboundEchoRequest'
```
## Diagnosis

- Parameter does not exist

- Outdated or incorrect documentation

## Fix

Create explicit ICMP rule:
```powershell
New-NetFirewallRule -Name AllowPing `
  -Protocol ICMPv4 `
  -IcmpType 8 `
  -Action Allow
```
## Lesson Learned

Always validate PowerShell parameters with Get-Help

---

## ❌ ISSUE 5 — Ping Works One Direction Only
### Symptom

- Windows → Kali: ping works

- Kali → Windows: ping fails

## Diagnosis

- Windows firewall blocks inbound ICMP

- Outbound ICMP allowed by default

## Fix

- Allow inbound ICMP on Windows.
```powershell
New-NetFirewallRule -Name AllowPing `
  -Protocol ICMPv4 `
  -IcmpType 8 `
  -Action Allow
```

## Lesson Learned

ICMP is directional and firewall-controlled.

---

## ❌ ISSUE 6 — nc Not Recognized on Windows

### Symptom
When running this from Powershell:
```
nc
```

## Error:
```
'nc' is not recognized as an internal or external command
```
## Diagnosis

- Netcat not installed

- Not included by default in Windows

## Fix

Installed Netcat manually

## Lesson Learned

Windows does not ship with common Linux utilities.
---
## ❌ ISSUE 7 — PowerShell TCP Listener Type Error
### Symptom

```powershell
System.Net.Socket.TcpListener
```

## Error:

```
Unable to find type
```
## Diagnosis

Incorrect .NET namespace.

## Fix

Correct namespace:

```powershell
System.Net.Sockets.TcpListener
```
## Lesson Learned

.NET namespaces must be exact.

---

## ❌ ISSUE 8 — Port Appears in TIME_WAIT but Not LISTENING
### Symptom
```powershell
netstat -ano
```

## Shows:

```text
TIME_WAIT
```
## Diagnosis

- Listener not persistent

- Connection closed immediately

## Fix

Run a persistent listener.

## Lesson Learned

TIME_WAIT ≠ open port.

---
## ❌ ISSUE 9 — nmap -A Fails with Firewall Enabled
### Symptom

```bash
nmap -A 192.168.20.10
```

- OS detection fails

- NSE scripts fail

- Ports filtered

## Diagnosis

`-A` performs:

- OS fingerprinting

- Traceroute

- NSE script execution

- All blocked by firewall.

## Fix

- Allow ICMP

- Allow specific ports

- Accept scan limitations

## Lesson Learned

Aggressive scans require permissive/vulnerable environments.

---

## ❌ ISSUE 10 — Splunk Receives Logs but Fields Missing

### Symptom

- Events visible

- Fields appear as:
```less
Event.System.Execution{@ProcessID}
```
## Diagnosis

- Raw XML ingestion

- No parsing or flattening

## Fix

Enable XML parsing:

```ini
KV_MODE = xml
```
## Lesson Learned

Ingestion does not equal usable data.
---

## ❌ ISSUE 11 — Wrong Source Instead of Sourcetype

### Symptom

Logs appear as:

```
source=WinEventLog:System
```
## Diagnosis

Sysmon logs must use:
```
XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```
## Fix

Match props.conf stanza exactly.

## Lesson Learned

Splunk config matching is literal and case sensitive.

---

## ❌ ISSUE 12 — TA Folder Does Not Exist

### Symptoms Observed
- Sysmon events appeared as **raw XML**
- Fields were deeply nested (e.g. `Event.System.Execution{@ProcessID}`)
- Common fields such as:
  - Image
  - ParentImage
  - CommandLine  
  were **not searchable**
- SPL queries returned empty tables despite visible events

At this stage, **telemetry existed but was unusable**.

## Fix Implemented:

- Created a **custom Sysmon Technical Add-on**  
- Enabled XML parsing
- Flattened critical fields using both **transforms** and **FIELDALIAS** definitions

### Example `props.conf` used in the custom TA:

```ini
[XmlWinEventLog:Microsoft-Windows-Sysmon/Operational]
SHOULD_LINEMERGE = true
KV_MODE = xml
NO_BINARY_CHECK = true
TRUNCATE = 0

# --- Flatten all fields using transforms ---
REPORT-extract-process = extract_process_fields
REPORT-extract-file = extract_file_fields
REPORT-extract-network = extract_network_fields
REPORT-extract-registry = extract_registry_fields
REPORT-extract-dns = extract_dns_fields
REPORT-extract-hashes = extract_hash_fields

# --- Optional FIELDALIASES ---
FIELDALIAS-process_id = process_id AS process_id
FIELDALIAS-process_guid = process_guid AS process_guid
FIELDALIAS-process_exec = process_exec AS process_exec
FIELDALIAS-parent_process_id = parent_process_id AS parent_process_id
FIELDALIAS-parent_process_exec = parent_process_exec AS parent_process_exec
FIELDALIAS-command_line = command_line AS command_line
FIELDALIAS-parent_command_line = parent_command_line AS parent_command_line
FIELDALIAS-user = user AS user
FIELDALIAS-src_ip = src_ip AS src_ip
FIELDALIAS-dest_ip = dest_ip AS dest_ip
FIELDALIAS-src_port = src_port AS src_port
FIELDALIAS-dest_port = dest_port AS dest_port
FIELDALIAS-provider_name = provider_name AS provider_name
FIELDALIAS-provider_guid = provider_guid AS provider_guid
FIELDALIAS-security_userid = security_userid AS security_userid
```

#### >>> Created some telemetry on Linux after creating the custom TA to confirm that the parsing is now correct.

---

## ❌ ISSUE 13 — Fields Still Missing After Config Changes

### Symptom

Aliases defined but fields not visible.

## Diagnosis

- Splunk does not retroactively parse old events

- No new Sysmon activity generated

## Fix

Generate new events via Metasploit

Restart Splunk service

## Lesson Learned

Parsing applies only to new data.

---

## ❌ ISSUE 14 — table Image ParentImage CommandLine Returns Nothing

###Symptom
```spl
| table Image ParentImage CommandLine
```
## Diagnosis

- Fields renamed via FIELDALIAS

- Querying wrong field names

## Fix
```spl
| table _time process_exec parent_process_exec command_line
```
## Lesson Learned

Always query the final field name.

## ❌ ISSUE 15 — Uncertainty Around dest_port

##Symptom

Unsure if `dest_port` exists.

## Diagnosis

- Only present in Sysmon Event ID 3

- Requires network activity

## Fix

Verified via:
```spl
| fields dest_port
```
## Lesson Learned

Field presence depends on event type.

## 📚 Final Meta-Lesson

- Most failures were not tool failures — they were expectation mismatches.

- Firewalls worked as designed

- SIEMs require parsing !!!

- Logs are useless without normalization

- “Working” does not mean “visible”

- I believe this troubleshooting process closely mirrors real SOC work and highlights the importance of understanding how tools actually behave, not how we expect them to behave.
