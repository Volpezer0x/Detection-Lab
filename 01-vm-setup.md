# 🖥️ Part 1: Virtual Machine Setup

This phase establishes the core lab environment using **two virtual machines** hosted on **VirtualBox**.

---

## 🧱 Virtual Machines

| VM Name     | OS          | Role / Purpose                     | RAM  | Notes                       |
|------------|-------------|------------------------------------|------|-----------------------------|
| Kali Linux | Kali Linux  | Attacker machine (red team tooling)| 4 GB | Tools: Nmap, Netcat, Metasploit, msfvenom |
| Windows 11 | Windows 11  | Victim endpoint + Splunk Enterprise| 4 GB | Sysmon, Defender, Splunk Enterprise installed |

---

## 🛠️ Installation Notes

### Kali Linux
- Used as the attacker platform
- Tools required:
  - nmap
  - metasploit
  - msfconsole
  - msfvenom
  - netcat
- Verified networking tools were installed and functional before proceeding

### Windows 11
- Used as the victim endpoint
- Windows Defender left enabled initially to observe detection behavior
- Sysmon installed in a later phase
- Splunk Enterprise installed locally on this VM

### Splunk Enterprise
- Installed on the Windows 11 VM
- Single-instance deployment (no forwarder/indexer separation)
- Used to collect and analyze endpoint telemetry generated during attacks

---

## ⚠️ Early Issues Encountered

- Performance degradation due to limited memory allocation (4 GB per VM)
- Splunk indexing and search latency when running alongside the Windows endpoint
- Resource contention between Splunk services and endpoint activity

**Mitigation:**
- CPU and memory allocations were reviewed and tuned where possible
- Non-essential background processes were minimized

---

## ✅ Outcome

Both virtual machines boot reliably, communicate over the network, and are ready for attack simulation, telemetry generation, and log analysis in subsequent phases.
