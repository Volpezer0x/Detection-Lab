# Cybersecurity Home Lab: Detection, Visibility, and Troubleshooting

This repository documents a fully isolated cybersecurity home lab built to simulate realistic attack, defense, and detection scenarios.

The lab emphasizes **visibility**, **troubleshooting**, and **SOC-level investigative skills**, rather than just tool installation.

---

## Core Objectives

- Design a safe environment for offensive testing
- Generate and collect endpoint telemetry using Sysmon
- Ingest and normalize logs in Splunk
- Diagnose visibility failures across network, host, and SIEM layers
- Document mistakes, assumptions, and fixes

---

## Lab Components

| Component | Purpose | Key Notes |
|-----------|---------|-----------|
| Kali Linux | Attack & enumeration | Nmap, Netcat, Metasploit |
| Windows Endpoint | Victim system | Sysmon, Defender, Firewall enabled |
| Splunk Enterprise | Centralized log ingestion | Custom Sysmon TA for field extraction |

---

## Networking

- Internal / host-only network for all VMs  
- Static IPs for consistency  
- Snapshots taken before testing  
- Avoided NAT-only to maintain L3 connectivity for multi-VM testing

---

## Key Troubleshooting Scenarios

1. **Nmap reports all ports “filtered”**
   - Issue: Windows Firewall silently dropping probes
   - Fix: Explicit inbound rules for ICMP/TCP
2. **Splunk ingesting Sysmon logs but fields missing**
   - Issue: XML unflattened, missing TA
   - Fix: Custom props.conf / transforms.conf
3. **PowerShell firewall commands failing**
   - Issue: Incorrect parameter usage
   - Fix: Explicit New-NetFirewallRule commands

---

## Why This Matters

This lab mirrors real SOC challenges:

- Logs exist but aren’t usable  
- Tools appear broken due to configuration  
- Misconfiguration looks like failure  

It shows **analytical thinking, troubleshooting, and system understanding** — exactly what employers seek.

---

## Repository Structure

See the folder breakdown above. Each section is meant to document **why it exists, how it works, and what I learned**.

---

## How to Use

- Read files sequentially for full lab context   
- Review `/07-issues-and-troubleshooting/` for real-world problem-solving examples  

---

## Ongoing Work

- Build detection logic on parsed Sysmon events  
- Add advanced attack scenarios  
- Expand documentation for multi-layer threat modeling
- Mapping generated telemetry to MITRE ATT&CK techniques
to validate detection coverage.
