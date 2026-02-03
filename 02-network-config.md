# 🌐 Part 2: Network Configuration & Isolation

This phase focuses on **safe VM communication** while preventing exposure to the host network.

---

## 🧩 Network Mode Selection

The following modes were evaluated:

| Mode | Result |
|----|------|
| Bridged | Rejected (unsafe for malware testing) |
| NAT | Limited VM-to-VM communication |
| NAT Network | Works but still allows internet |
| Internal Network | Selected |

**Internal Network** was chosen to ensure:
- no internet access
- VM-to-VM communication
- complete host isolation

---

## 🔧 Static IP Configuration

Static IPs were assigned to avoid detection inconsistencies.

| VM | IP |
|----|----|
| Windows | 192.168.20.10 |
| Kali | 192.168.20.11 |
| Splunk | 192.168.20.12 |

Subnet: `255.255.255.0`

---

## 🧪 Connectivity Testing

From Kali:
```bash
ping 192.168.20.10
nmap -sn 192.168.20.0/24
