# 🔎 Lab Scope & Context

This lab focuses on **endpoint detection engineering** and defensive investigation workflows, not exploit development or red-team operations.

The primary objective is to understand:
- how attacker actions generate endpoint telemetry
- how Windows logs activity at the process and command-line level
- how telemetry is ingested, parsed, and normalized by a SIEM
- where detection visibility breaks down and why
- how an analyst pivots from raw events to investigative context

This lab intentionally prioritizes **telemetry understanding and log analysis** over bypassing security controls.

---

## ✅ In Scope

- Endpoint telemetry generation (Windows + Sysmon)
- Process creation, command-line, and parent/child relationships
- Network and execution artifacts generated during attack simulation
- Sysmon configuration and tuning
- Splunk ingestion, sourcetype handling, and SPL querying
- Manual investigation and event pivoting
- Basic attack simulation using controlled tooling

---

## 🎯 Threat Model (Simplified)

- Single endpoint compromise
- User-level execution
- No evasion or obfuscation
- Focus on post-execution visibility


## ❌ Out of Scope

- Exploit development or weaponization
- Internet-facing infrastructure
- Persistence mechanisms
- Live malware deployment in production environments
- Detection engineering at scale
- Production hardening or SOC automation

---

## 🧪 Lab Environment Notes

- All testing is performed in an **isolated lab network**
- No EDR platform is used beyond native Windows Defender
- No prebuilt Splunk Sysmon TA is assumed
- Detection logic is exploratory rather than production-ready

> The goal is not perfect detection, but understanding **why detection succeeds or fails**.
