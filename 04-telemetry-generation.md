# 📡 Part 4: Telemetry Generation (Sysmon)

This phase focuses on generating **high‑fidelity endpoint telemetry** using Sysmon so attacker activity can be reliably observed, searched, and analyzed inside Splunk.

The goal is to move from *“activity happened”* to *“activity is observable and explainable.”*

---

## 🛠️ Sysmon Setup

Sysmon was installed on the Windows endpoint using a **custom configuration** to ensure meaningful visibility.

Logging was enabled for:

- Process creation
- Network connections
- File creation
- Image loading

Sysmon was intentionally configured to favor **clarity and learning** over stealth or performance optimization.

---

## 🔎 Events Observed

The following Sysmon Event IDs were consistently generated during attack simulation:

| Event ID | Description |
|--------|------------|
| 1 | Process creation |
| 3 | Network connection |
| 11 | File creation |

These events provided visibility into:
- how payloads were executed
- which processes spawned child processes
- which binaries initiated network connections

---

## ❌ Issues Encountered

Several telemetry issues were identified during validation:

- Command-line fields were missing in Splunk
- Sysmon logs appeared as raw XML
- Fields were nested and unusable for searches

This made detection logic impossible despite logs being present.

---

## 🧯 Root Cause & Fix

**Root Cause:**  
Sysmon logs were ingested without a proper **Technical Add-on (TA)** for field extraction.

**Fix Implemented:**
- Created a **custom Sysmon TA**
- Enabled XML parsing
- Flattened critical fields (Image, ParentImage, CommandLine)

> Parsing fixes only applied to **newly ingested events**, requiring fresh activity generation.

---

## 🧪 Validation

After fixes:
- Fields populated correctly
- XML artifacts removed
- Searches returned meaningful results

Example validation query:

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| table _time parent_process_exec process_exec command_line
