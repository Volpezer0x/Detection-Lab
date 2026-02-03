# 🌐 Part 2: Network Configuration

This section explains **how to safely connect your virtual machines** while minimizing host risk.

---

## 🧩 Network Modes

| VM Platform | Recommended Mode |
|------------|----------------|
| VirtualBox | Internal Network / Host-only (for malware analysis) |
| VMware | LAN Segment / Host-only |
| Both | NAT (only if internet access required for testing tools) |

---

## 🔧 IP Assignment

- Assign **static IPs** for all lab machines to maintain consistency:
  - Kali Linux: 192.168.20.11
  - Windows Endpoint: 192.168.20.10
  - Splunk: 192.168.20.12

- Avoid bridged mode when executing potentially malicious payloads
- Use `ping` and `nc` to verify connectivity within the isolated lab

---

## ✅ Best Practices

- Always double-check network isolation
- Keep a separate “lab only” network
- Consider VLANs or virtual switches for segmentation
