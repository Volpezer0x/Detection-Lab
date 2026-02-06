# 📡 Part 4: Telemetry Generation (Sysmon)

This phase focuses on generating **high‑fidelity endpoint telemetry** using Sysmon so attacker activity can be reliably observed, searched, and analyzed inside Splunk.

The goal is to move from:

> *“Activity happened”*  
> → *“Activity is observable, structured, and explainable.”*

This phase exposed the critical difference between **log ingestion** and **usable detection data**.

---

## 🎯 Objectives

- Install Sysmon on the Windows endpoint
- Generate endpoint telemetry during attacker activity
- Ingest Sysmon logs into Splunk
- Normalize and flatten fields for effective searching
- Validate that detection-relevant data exists

---

## 🛠️ Sysmon Installation & Configuration

Sysmon was installed on the Windows endpoint using the **Olaf Hartong configuration file sourced from GitHub**.

### Resources:

> Sysmon - https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon*

> Sysmon Configuration - https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml*

> ⚠️ Sysmon configuration Schema version 4.90

What is known:
- The configuration was **not the Sysmon default**
- It was intended to increase telemetry coverage
- Focused on **learning and visibility**, not stealth or performance tuning

Sysmon logging covered common endpoint behaviors such as:
- Process creation
- Network connections
- File creation
- Image loading

This was sufficient to generate meaningful telemetry during attack simulation.

---

## 🔎 Telemetry Generated

During attack simulation, the following Sysmon Event IDs were consistently generated:

| Event ID | Description |
|--------|------------|
| 1 | Process creation |
| 3 | Network connection |
| 11 | File creation |

These events provided visibility into:

- Execution of the msfvenom-generated payload
- Parent–child process relationships
- Outbound network connections initiated by the payload
- File creation activity during payload delivery

---

## ❌ Issues Encountered

Despite logs being present in Splunk, multiple issues prevented meaningful analysis:

### Symptoms Observed
- Sysmon events appeared as **raw XML**
- Fields were deeply nested (e.g. `Event.System.Execution{@ProcessID}`)
- Common fields such as:
  - Image
  - ParentImage
  - CommandLine  
  were **not searchable**
- SPL queries returned empty tables despite visible events

At this stage, **telemetry existed but was unusable**.

---

## 🧯 Root Cause Analysis

### Root Cause

Sysmon logs were ingested **without a proper Technical Add‑on (TA)** to:

- Parse XML
- Extract key-value pairs
- Normalize common endpoint fields

Splunk was storing the data, but **not understanding it**.

---

## 🛠️ Fix Implemented: Custom Sysmon TA

Instead of installing the official Sysmon TA, a **custom Sysmon Technical Add‑on** was created manually to understand how parsing actually works.

### Key Fixes Applied

- Created a **custom Sysmon Technical Add-on**
- Enabled XML parsing
- Flattened critical fields:
  - Image
  - ParentImage
  - CommandLine
  
 *>> Inside Splunk's directory, navagated to `C:\Program Files\Splunk\etc\apps\TA-sysmon-custom\default` and edited `props.conf` to create the custom TA:*

```ini
[XmlWinEventLog:Microsoft-Windows-Sysmon/Operational]
SHOULD_LINEMERGE = true
KV_MODE = xml
NO_BINARY_CHECK = true
TRUNCATE = 0

# --- Flatten all fields using transforms ---
REPORT-extract-process = extract_process_fields
REPORT-extract-file = extract_file_fields
REPORT-extract-network = extract_network_fields
REPORT-extract-registry = extract_registry_fields
REPORT-extract-dns = extract_dns_fields
REPORT-extract-hashes = extract_hash_fields

# --- Optional FIELDALIASES ---
FIELDALIAS-process_id = process_id AS process_id
FIELDALIAS-process_guid = process_guid AS process_guid
FIELDALIAS-process_exec = process_exec AS process_exec
FIELDALIAS-parent_process_id = parent_process_id AS parent_process_id
FIELDALIAS-parent_process_exec = parent_process_exec AS parent_process_exec
FIELDALIAS-command_line = command_line AS command_line
FIELDALIAS-parent_command_line = parent_command_line AS parent_command_line
FIELDALIAS-user = user AS user
FIELDALIAS-src_ip = src_ip AS src_ip
FIELDALIAS-dest_ip = dest_ip AS dest_ip
FIELDALIAS-src_port = src_port AS src_port
FIELDALIAS-dest_port = dest_port AS dest_port
FIELDALIAS-provider_name = provider_name AS provider_name
FIELDALIAS-provider_guid = provider_guid AS provider_guid
FIELDALIAS-security_userid = security_userid AS security_userid
```
> Parsing fixes only applied to **newly ingested events**, requiring attacker activity to be re-executed !!!
---
## 🧪 Validation in Splunk

After applying parsing fixes, Sysmon telemetry became fully searchable and usable.

Validation query:
```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1
| table _time parent_process_exec process_exec command_line src_ip dest_ip src_port dest_port
```
## Validation Results

- Fields populated correctly

- Parent–child relationships clearly visible

- Command-line arguments available

- Network connection fields present

- XML artifacts removed from searches

This confirmed that endpoint telemetry was successfully transformed into actionable detection-ready data.

## 🧠 Lessons Learned

- Logs existing ≠ logs usable

- XML ingestion without parsing is effectively raw storage

- Field normalization is mandatory for detection engineering

- Parsing fixes do not apply retroactively

- Endpoint telemetry matters even when attacks fail

- This phase closely mirrors real SOC challenges where data exists but cannot yet be processed without normalization.
