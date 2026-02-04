# Detection Lab Architecture

This document describes the architecture of the cybersecurity home lab used for
endpoint monitoring, attack simulation, and detection engineering.

The lab consists of an attacker machine (Kali Linux), a victim endpoint (Windows),
and a SIEM platform (Splunk Enterprise), all connected through an isolated virtual network.

---

## Logical Architecture Diagram

```mermaid
flowchart LR
    Kali[Kali Linux VM\nAttacker\n\nTools:\n- Nmap\n- Netcat\n- Metasploit]
    
    Win[Windows 10/11 VM\nVictim Endpoint\n\n- Sysmon\n- Windows Defender Firewall\n- Splunk Universal Forwarder]
    
    Splunk[Splunk Enterprise\nDetection & Analysis\n\n- Sysmon TA\n- Field Extraction\n- Index: endpoint]

    Kali -- Reconnaissance / Exploitation --> Win
    Win -- Sysmon + Windows Event Logs --> Splunk

    subgraph Isolated_Virtual_Network
        Kali
        Win
    end
