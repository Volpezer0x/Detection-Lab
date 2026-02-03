# 🖥️ Part 1: Virtual Machine Setup

This section details **how to create and configure virtual machines** for your home lab.

---

## 🛠️ Virtual Machines Deployed

| VM | Purpose |
|----|--------|
| Kali Linux | Attacker machine (Nmap, Metasploit, Netcat) |
| Windows 10 / 11 | Victim endpoint (Sysmon, Defender, Splunk Forwarder) |
| Splunk Enterprise | SIEM for log ingestion and analysis |

---

## ⚙️ Configuration Notes

- Base VMs created in **VirtualBox** (or VMware if preferred)
- Minimal resources allocated for lab purposes
  - 4 GB RAM per VM
  - 2 vCPU
  - 40 GB disk
- Snapshots created after clean install for easy rollback
- ISO sources verified (official Kali, Windows evaluation)

---

## 📌 Key Takeaways

- Keep VMs isolated from your host network initially
- Snapshots are essential before testing
- Document VM IPs, hostnames, and credentials in a **secure notes file**
