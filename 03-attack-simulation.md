# 🔴 Part 3: Attack Simulation

This phase simulates **attacker behavior** to generate realistic endpoint telemetry that can later be analyzed from a defender’s perspective.

The objective is not exploitation, but **activity generation** creating the kinds of signals a SOC analyst or detection engineer would expect to see during early-stage attacker behavior.

---

## 🛠️ Techniques Used

The following attacker techniques were intentionally simulated:

- Network scanning using **Nmap**
- Payload generation using **msfvenom**
- Payload delivery over HTTP
- Process execution on the Windows endpoint

Each technique was selected to generate **observable telemetry**, not to evade detection.

---

## 🔍 Nmap Scanning

Initial reconnaissance was performed from the Kali attacker machine against the Windows endpoint.

```bash
nmap -A 192.168.20.10
```
 
## ❌ Issues Encountered

- Initial scans returned no open ports

- Firewall blocked probes

- Fixed via firewall rule adjustments

## ✅Outcome

- Network connections

- Process execution logs

- Firewall interaction
