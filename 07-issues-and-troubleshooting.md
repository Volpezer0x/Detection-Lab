# 🧯 Troubleshooting & Lessons Learned – Detection Lab

This section documents **every issue encountered from initial lab setup to final Splunk field validation**, including misconfigurations, incorrect assumptions, tooling gaps, and parsing problems. Each issue includes symptoms, failed attempts, diagnosis steps, fixes, and lessons learned.

The goal is to help others **recognize the same failure patterns quickly** and avoid repeating the same mistakes. Note that I'm using both the Linux terminal as well as Powershell on Windows.

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

##Diagnosis

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
