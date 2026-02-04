# Detection Lab Architecture

This document describes the architecture of the cybersecurity home lab used for
endpoint monitoring, attack simulation, and detection engineering.

The lab consists of an attacker machine (Kali Linux), a victim endpoint (Windows),
and a SIEM platform (Splunk Enterprise), all connected through an isolated virtual network.

---

## Logical Architecture Diagram

```text
+------------------------+
|     Kali Linux VM      |
|       (Attacker)       |
|                        |
| Tools:                 |
|  - Nmap                |
|  - Netcat              |
|  - Metasploit          |
+-----------+------------+
            |
            | Recon / Exploit Traffic
            | (TCP, ICMP, NSE, Payloads)
            v
+------------------------+
|      Windows VM        |
|   (Victim Endpoint)    |
|                        |
|  - Sysmon              |
|  - Windows Firewall    |
|  - Defender            |
|  - PowerShell          |
|  - Splunk Forwarder    |
+-----------+------------+
            |
            | Sysmon + WinEvent Logs
            | (XML -> Flattened Fields)
            v
+------------------------+
|   Splunk Enterprise    |
|        (SIEM)          |
|                        |
|  - Custom Sysmon TA    |
|  - Field Normalization |
|  - Index: endpoint     |
|  - SPL Analysis        |
+------------------------+
```


---

## Network Topology
```yaml
- Virtualized using: VirtualBox / VMware
- Network Mode: **Host-Only** or **Internal Network**
- Example Subnet:
```

---


All systems are isolated from the internet to safely simulate malicious activity.

---

## Data Flow Explanation

```yaml
1. Kali Linux initiates reconnaissance and exploitation activity
2. Windows endpoint receives traffic and generates telemetry
3. Sysmon records:
 - Process creation
 - Network connections
 - Parent-child relationships
4. Splunk Universal Forwarder sends logs to Splunk Enterprise
5. Splunk parses XML, flattens fields, and enables SPL-based detection

---

## Why This Architecture Matters

This lab mirrors real-world SOC workflows:

- Attack simulation → telemetry generation
- Endpoint visibility → log ingestion
- Parsing failures → troubleshooting
- Detection validation using real attacker behavior
```
