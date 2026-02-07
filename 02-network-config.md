# 🌐 Part 2: Network Configuration & Isolation

This phase focuses on creating a **safe, isolated network** that allows controlled attacker–victim communication while preventing exposure to the host or internet.

---

## 🧩 Network Mode Selection

Multiple VirtualBox network modes were evaluated to balance **functionality** and **safety**.

| Network Mode | Result | Reason |
|-------------|--------|--------|
| Bridged | ❌ Rejected | Exposes lab traffic to the physical network |
| NAT | ❌ Rejected | Limited VM-to-VM visibility |
| NAT Network | ⚠️ Partial | Allows VM communication but still enables internet |
| Internal Network | ✅ Selected | Full isolation with VM-to-VM communication |

**Internal Network** was selected to ensure:
- No internet connectivity
- Full VM-to-VM communication
- No exposure to the host or external network
- Safe simulation of malicious activity

---

## 🔧 Static IP Configuration

Static IP addressing was used to ensure:
- Consistent scan targets
- Reliable log correlation in Splunk
- Predictable detection results

### Configuration Method

- **Windows 11:** Static IP configured manually via **Network Adapter settings**
- **Kali Linux:** IP configured using **NetworkManager**
- **Gateway:** Not configured (intentional isolation)

### Assigned Addresses

| System | Role | IP Address |
|------|-----|-----------|
| Windows 11 VM | Victim Endpoint + Splunk Enterprise | 192.168.20.10 |
| Kali Linux VM | Attacker | 192.168.20.11 |

**Subnet:** `255.255.255.0`

> Splunk Enterprise is installed **on the Windows VM**, not on a separate system.

---

## 🧪 Connectivity Validation

Basic reachability and host discovery were tested from both sides.

### From Kali Linux
```bash
ping 192.168.20.10
nmap -sn 192.168.20.0/24
```
### From Windows
```powershell
ping 192.168.20.11
```
## ❌ Issues Encountered & Troubleshooting

### Issue 1: ICMP Ping Failed (Kali → Windows)

## Symptom

- Kali could not ping the Windows VM

- nmap -sn returned no live hosts

## Diagnosis

- Verified static IPs on both systems

- Confirmed both VMs were attached to the same Internal Network

- Identified Windows Defender Firewall as the likely blocker

## Root Cause

Windows Defender Firewall blocks ICMP Echo Requests by default

## Fix
```powershell
New-NetFirewallRule -Name AllowICMP -Protocol ICMPv4 -IcmpType 8 -Action Allow
```

## Result

- ICMP traffic allowed

- Kali successfully detected the Windows host

### Issue 2: Nmap Reports All Ports as Filtered

## Symptom

- nmap `-A 192.168.20.10` reported all ports as filtered

- Host was reachable, but no services appeared open

## Diagnosis

- Confirmed Windows services were running

- Verified Kali routing and subnet configuration

- Observed silent packet drops instead of RST responses

## Root Cause

- Windows Defender Firewall silently drops unsolicited inbound traffic

## Fix

- Temporarily adjusted firewall rules to allow testing

- Performed scans to validate visibility

- Re‑enabled firewall rules to preserve realistic detection conditions

## Result

- Controlled visibility during testing

- Firewall left enabled for realistic endpoint behavior

### Issue 3: Service Visibility vs Detection Tradeoff

## Symptom

- Services (including Splunk Web) were not visible during aggressive scans

- Nmap output suggested a hardened endpoint

## Diagnosis

- Confirmed Splunk Web was running locally

- Identified Splunk listening on port 8000

- Recognized firewall behavior as expected in a real environment

## Root Cause

Windows Firewall intentionally blocking external service enumeration

## Fix

- Accepted limited visibility as a feature, not a bug

- Focused on endpoint telemetry instead of exposed services

## Result

- More realistic attack surface

- Better detection engineering outcomes

## ✅ Outcome

- VMs communicate reliably within an isolated network

- No internet or host exposure

- Kali performs reconnaissance against Windows safely

- Windows Defender remains enabled

- Telemetry reflects realistic enterprise conditions

- Network behavior mirrors an internal corporate segment

## 🧠 Lessons Learned

- Isolation is more important than convenience in malware labs

- Firewalls can make systems appear “offline” when they are functioning correctly

- ICMP and service visibility are not guaranteed in real environments

- Detection engineering benefits from restricted, not open, systems
