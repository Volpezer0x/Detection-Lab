# Detection Lab Architecture

This document describes the architecture of the cybersecurity home lab used for
endpoint monitoring, attack simulation, and detection engineering.

The lab consists of an attacker machine (Kali Linux), a victim endpoint (Windows),
and a SIEM platform (Splunk Enterprise), all connected through an isolated virtual network.

⚠ Note: Splunk Enterprise runs as a dedicated VM within the same isolated network.


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
- Virtualized using: VirtualBox
- Network Mode: **Internal Network**
```

---


All systems are intentionally isolated from the internet to safely simulate malicious activity without external risk.

---

## Data Flow Explanation

```yaml
1. Kali Linux initiates reconnaissance and exploitation activity
2. Windows endpoint receives traffic and generates telemetry
3. Sysmon records:
 - Process creation
 - Network connections
 - Parent-child relationships
4. Splunk Enterprise index created pointing to Sysmon logs
5. Custom Splunk TA created for proper log parsing
6. Splunk parses XML, flattens fields, and enables SPL-based detection
```
---

```yaml

## Detection Engineering Focus

This lab was designed specifically to:

- Validate endpoint visibility
- Understand firewall and telemetry blind spots
- Practice Sysmon XML parsing and field normalization
- Simulate real attacker behavior rather than synthetic logs
```

```yaml
## Why This Architecture Matters

This lab mirrors real-world SOC workflows:

- Attack simulation → telemetry generation
- Endpoint visibility → log ingestion
- Parsing failures → troubleshooting
- Detection validation using real attacker behavior
```
