# 🔵 Part 6: Detection & Analysis

This phase focuses on **manual investigation and analysis** of endpoint telemetry inside Splunk, using attacker-generated activity to understand how malicious behavior appears from a defender’s perspective.

The objective was not to create automated detections, but to:
- validate that attacker activity was observable
- practice SOC-style investigation workflows
- understand how Sysmon telemetry maps to attacker behavior

---

## 🎯 Analysis Objectives

The analysis aimed to answer the following questions:

- Did the payload execute on the endpoint?
- Which process launched it?
- What child processes were spawned?
- Did the activity result in network communication back to the attacker?
- Can the attack chain be reconstructed from logs alone?

---

## 🔎 Investigation Workflow

### 🧭 Step 1: Identify Attacker-Originated Activity

The investigation began by searching for events originating from the Kali attacker machine.

```spl
index=endpoint <KALI_IP_ADDRESS>
```
This was used to confirm:

- inbound connections from the attacker

- correlation between network activity and endpoint events

## 🧬 Step 2: Locate Payload Execution Events

Next, the malware filename generated via msfvenom was used as a pivot point.
```spl
index=endpoint password.pdf.exe
```

This allowed quick identification of:

- process creation events

- file execution activity related to the payload

## 🧾 Step 3: Focus on Process Creation (Sysmon Event ID 1)

After identifying relevant events, attention was narrowed to Sysmon Event ID 1 (Process Creation).

By expanding the event in Splunk, the following was observed:

- password.pdf.exe executed successfully

- the payload spawned a child process

- the child process was cmd.exe

- a unique Process GUID was associated with the execution chain

- The Process GUID was used instead of Process ID to ensure reliable correlation across events.

## 🔗 Step 4: Pivot Using Process GUID

The Process GUID was used to isolate all activity related to that execution chain.
```spl
index=endpoint <PROCESS_GUID>
| table _time ParentImage Image CommandLine
```

This confirmed:

- the parent-child relationship (password.pdf.exe → cmd.exe)

- the command-line context used during execution

- the full execution timeline

## 🌐 Step 5: Network Confirmation

Network telemetry was correlated with process execution to confirm command-and-control behavior.

Reverse shell activity was:

- observed in Metasploit

- confirmed in Splunk telemetry

Example analysis query:
```spl
index=endpoint dest_ip=* dest_port=*
| table _time Image dest_ip dest_port
| sort _time
```

This validated:

- outbound network connections

- alignment with the Metasploit listener

- successful callback from victim to attacker

## 🛡️ Defender Interaction Observations

During testing:

- Windows Defender was manually disabled to allow payload execution

- A warning was displayed prior to execution

- The payload was executed despite the warning

This demonstrated:

- user override of security controls

- how alerts may exist without prevention

- why endpoint telemetry is critical even when defenses are bypassed

## ❌ Challenges Encountered During Analysis

Several issues complicated investigation early on:

- Fields were initially unavailable due to parsing problems

- Raw XML made pivoting impossible

- Incorrect field names resulted in empty tables

- Old events did not reflect updated parsing logic

- Reliance on Process ID proved unreliable compared to Process GUID

These challenges reinforced the importance of:

- field normalization

- understanding event structure

- validating data before analysis

## 🧠 Key Lessons Learned

- Detection starts with investigation, not alerts

- Process GUIDs are more reliable than Process IDs

- Parent-child relationships are critical for understanding attacks

- Network telemetry must be correlated with process execution

- Logs without parsing are operationally useless

- “Malware ran” ≠ “attack understood”

## ✅ Outcome

By the end of this phase:

- Payload execution was fully reconstructed from logs

- Parent-child process relationships were clearly identified

- Network callbacks were confirmed in both Splunk and Metasploit

- Manual investigation techniques mirrored real SOC workflows

- This phase successfully demonstrated how raw telemetry becomes actionable intelligence through structured analysis.
