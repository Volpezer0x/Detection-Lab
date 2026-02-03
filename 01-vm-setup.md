# 🖥️ Part 1: Virtual Machine Setup

This phase establishes the core lab environment using **three virtual machines**.

---

## 🧱 Virtual Machines

| VM | Purpose |
|----|--------|
| Kali Linux | Attacker |
| Windows 10 | Victim endpoint |
| Splunk Enterprise | SIEM |

---

## 🛠️ Installation Notes

### Kali Linux
- Used as the attacker platform
- Tools required:
  - nmap
  - msfconsole
  - msfvenom
- Confirm networking tools installed before proceeding

### Windows 10
- Used as victim endpoint
- Sysmon installed later
- Windows Defender kept enabled initially to observe behavior

### Splunk
- Standalone Splunk Enterprise instance
- Dedicated VM to avoid resource contention

---

## ⚠️ Early Issues Encountered

- Windows VM performance degraded due to low RAM
- Splunk indexing lag when sharing resources
- Fixed by allocating additional memory and CPU cores

---

## ✅ Outcome

All VMs boot reliably and are ready for network configuration.
