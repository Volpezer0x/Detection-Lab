# 🛡️ Cybersecurity Home Detection Lab  
## Detection, Visibility, and Troubleshooting

This repository documents a **fully isolated cybersecurity home lab** designed to simulate realistic attack activity and analyze how that activity is observed, ingested, and investigated.

The lab prioritizes **visibility, troubleshooting, and SOC-level investigative thinking**, not just tool installation or exploit execution.

---

## 🎯 Core Objectives

- Design a safe, isolated environment for controlled attack simulation
- Generate high-fidelity endpoint telemetry using Sysmon
- Ingest and normalize logs in Splunk
- Identify and resolve visibility failures across host, network, and SIEM layers
- Document assumptions, mistakes, and fixes encountered during analysis

---

## 🧩 Lab Components

| Component | Purpose | Key Notes |
|---------|---------|----------|
| Kali Linux | Attack simulation & enumeration | Nmap, Netcat, Metasploit |
| Windows Endpoint | Victim system | Sysmon (Olaf Hartong config), Defender, Firewall |
| Splunk Enterprise | Centralized log analysis | Custom Sysmon TA for field extraction |

---

## 🌐 Networking Design

- Internal / host-only virtual network  
- Static IP addressing for consistency  
- VM snapshots taken before testing  
- NAT-only networking avoided to preserve L3 visibility and realistic scanning behavior  

---

## 🧯 Key Troubleshooting Scenarios

1. **Nmap reports all ports as “filtered”**  
   - Root cause: Windows Firewall silently dropping probes  
   - Resolution: Explicit inbound ICMP/TCP firewall rules  

2. **Sysmon logs present but unusable in Splunk**  
   - Root cause: Raw XML ingestion without field extraction  
   - Resolution: Custom Sysmon Technical Add-on (`props.conf`, `transforms.conf`, field flattening)  

3. **PowerShell firewall commands failing**  
   - Root cause: Incorrect parameter usage  
   - Resolution: Correct `New-NetFirewallRule` syntax and validation  

---

## 🧠 Why This Lab Matters

This lab reflects common SOC realities:

- Logs exist, but detection fails  
- Tools appear broken due to configuration gaps  
- Misconfiguration masquerades as visibility loss  

The focus is on **understanding why data looks the way it does**, not simply that it exists.

---

## 🗂️ Repository Structure

Each section documents:
- **why** it exists  
- **how** it works  
- **what** was learned  

Files are organized to follow the investigation lifecycle rather than tool order.

---

## ▶️ How to Read This Lab

- Start with the lab scope and architecture setup  
- Proceed sequentially through the numbered sections  
- Review <a href="https://github.com/Volpezer0x/Detection-Lab">07-issues-and-troubleshooting</a> for real-world failure analysis  
---

## 🚧 Ongoing Work

- Build detection logic using parsed Sysmon telemetry  
- Add additional attack scenarios  
- Map observed behavior to MITRE ATT&CK techniques  
- Validate detection coverage and investigative workflows  
