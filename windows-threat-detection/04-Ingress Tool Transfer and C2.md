# Ingress Tool Transfer & Command and Control Investigation

> **Portfolio Case Study** — Investigating Malware Delivery, Tool Transfer, and Command & Control Activity on Windows Endpoints

---

# Executive Summary

This investigation documents the analysis of attacker behaviour following successful initial access to a Windows endpoint. After obtaining access, the attacker downloaded additional tools, established communication with external infrastructure, and prepared the compromised host for long-term remote control.

The investigation focused on identifying common ingress tool transfer methods, detecting outbound Command and Control (C2) communications, analysing Windows process creation, correlating network activity with endpoint telemetry, and identifying behavioural indicators that distinguish legitimate administrative activity from malicious post-exploitation.

---

# Incident Overview

Following a successful compromise, endpoint monitoring identified multiple attempts to download executable content from external infrastructure using native Windows utilities.

Subsequent telemetry revealed outbound network connections initiated by newly executed processes, indicating that additional malware had been introduced onto the system and was attempting to establish persistent communication with attacker-controlled infrastructure.

The objective of this investigation was to determine how the attacker transferred additional tools, identify C2 communication, reconstruct attacker behaviour, and recommend behavioural detections.

---

# Investigation Objectives

- Detect ingress tool transfer activity.
- Identify abused native Windows utilities.
- Correlate process execution with network activity.
- Identify Command and Control communications.
- Determine downloaded malware location.
- Preserve indicators of compromise.
- Recommend behavioural detections.
- Map activity to MITRE ATT&CK.

---

# Environment

## Operating System

- Microsoft Windows

## Telemetry Sources

- Sysmon
- Windows Security Event Logs
- PowerShell Operational Logs
- DNS Activity
- Network Connections
- File System

## Investigation Tools

- Event Viewer
- Sysmon
- PowerShell Logs

---

# Investigation Workflow

```text
Successful Authentication
        │
        ▼
Download Additional Tools
        │
        ▼
File Creation
        │
        ▼
Malware Execution
        │
        ▼
Outbound Network Connection
        │
        ▼
DNS Resolution
        │
        ▼
Command & Control Beacon
        │
        ▼
Attacker Remote Control
```

---

# Investigation 1 — Ingress Tool Transfer

## Goal

Determine how the attacker transferred additional tools onto the compromised system.

### Common Tool Transfer Methods

| Utility                   | Typical Use                                 |
| ------------------------- | ------------------------------------------- |
| `curl.exe`                | Download payloads                           |
| `certutil.exe`            | Download files using native Windows utility |
| `Invoke-WebRequest (IWR)` | PowerShell-based download                   |
| Web Browser               | Manual download via GUI                     |
| RDP Copy/Paste            | Transfer files over an interactive session  |

These methods are frequently abused because they rely on trusted Windows components already present on most systems.

---

### Typical Commands Observed

Examples include:

- `curl.exe`
- `certutil.exe`
- `Invoke-WebRequest`
- `powershell.exe`

The investigation focused on **behaviour**, not exact command syntax, as attackers often modify command-line arguments to evade simple detections.

---

### Evidence Sources

**Sysmon Event ID 1 — Process Creation**

Important fields:

- Image
- Parent Image
- CommandLine
- User
- Process GUID

**Sysmon Event ID 11 — File Creation**

Important fields:

- TargetFilename
- Image
- User

---

### Investigation Findings

Evidence indicated that native Windows utilities were used to retrieve executable content from external infrastructure.

The downloaded files were written to temporary directories before execution, demonstrating a common second-stage malware deployment technique.

---

# Investigation 2 — Command & Control (C2)

## Goal

Determine whether malware established communication with attacker-controlled infrastructure.

### Behaviour Observed

Following execution of the downloaded payload:

- New outbound network connections appeared.
- DNS lookups preceded external communication.
- Processes initiated periodic communication with remote infrastructure.
- Activity suggested beaconing rather than normal user browsing.

---

### Primary Evidence

**Sysmon Event ID 3**

(Network Connection)

Important fields:

- Image
- Destination IP
- Destination Hostname
- Destination Port
- User
- Process GUID

These fields enabled correlation between the network connection and the originating process.

---

### DNS Investigation

DNS activity was examined to determine:

- Destination domain
- Frequency of lookups
- Newly observed domains
- Repeated beacon intervals

---

### Process Correlation

The following process relationships were investigated:

```text
Explorer.exe
      │
      ▼
PowerShell.exe
      │
      ▼
Downloaded Malware
      │
      ▼
Outbound Network Connection
```

Correlating process creation with network activity significantly increased confidence in identifying malicious communications.

---

# Indicators of Compromise (IOCs)

| IOC         | Observation                                   |
| ----------- | --------------------------------------------- |
| Sysmon      | Event ID 1                                    |
| Sysmon      | Event ID 3                                    |
| Sysmon      | Event ID 11                                   |
| Utility     | `curl.exe` execution                          |
| Utility     | `certutil.exe` execution                      |
| Utility     | PowerShell download activity                  |
| Network     | Outbound connections to uncommon destinations |
| DNS         | Repeated lookups to external domains          |
| File System | Executable created within temporary directory |

---

# MITRE ATT&CK Mapping

| Tactic              | Technique                              |
| ------------------- | -------------------------------------- |
| Command and Control | Application Layer Protocol             |
| Command and Control | Ingress Tool Transfer                  |
| Command and Control | Dynamic Resolution                     |
| Execution           | Command and Scripting Interpreter      |
| Defense Evasion     | Living-off-the-Land Binaries (LOLBins) |

---

# Detection Opportunities

- Alert on execution of `curl.exe`, `certutil.exe`, or PowerShell download utilities from unusual parent processes.
- Detect executable files created within user profile or temporary directories.
- Correlate process creation with immediate outbound network connections.
- Monitor uncommon DNS queries generated by administrative utilities.
- Detect repeated outbound connections to newly observed domains.
- Alert when native Windows utilities download executable content.
- Correlate downloaded files with subsequent execution.

---

# SOC Investigation Guide

## Initial Questions

- Which process initiated the download?
- Which user executed the command?
- Where was the downloaded file stored?
- Was the downloaded file executed?
- Which destination IP or domain was contacted?
- Did DNS activity precede the connection?
- Was the communication periodic (beaconing)?
- Were additional payloads downloaded?

---

## Critical Fields

### Sysmon Event ID 1

- Image
- Parent Image
- CommandLine
- User

### Sysmon Event ID 3

- Image
- Destination IP
- Destination Hostname
- Destination Port
- Process GUID

### Sysmon Event ID 11

- TargetFilename
- Image
- Creation Time

---

# Skills Demonstrated

- Windows Endpoint Investigation
- Sysmon Analysis
- Network Connection Analysis
- DNS Investigation
- Process Correlation
- Command & Control Detection
- Ingress Tool Transfer Investigation
- IOC Identification
- Behavioural Threat Hunting
- MITRE ATT&CK Mapping
- Incident Documentation

---

# Lessons Learned

- Native Windows utilities are frequently abused to introduce second-stage payloads.
- Behavioural analysis is more reliable than detecting individual command-line strings.
- Correlating process creation, file creation, DNS activity, and network connections provides strong evidence of C2 activity.
- Temporary directories are common locations for downloaded malware.
- Early detection of ingress tool transfer can prevent attackers from establishing long-term remote access.

---

# Analyst Reflection

This investigation demonstrated how attackers extend their capabilities after gaining an initial foothold. Rather than relying on custom malware immediately, adversaries frequently abuse trusted Windows utilities to download additional tools and establish Command and Control channels. By correlating endpoint telemetry with network activity, defenders can identify these behaviours before the attacker progresses to persistence, lateral movement, or large-scale data theft. The investigation reinforced the value of behavioural detection over signature-based approaches when analysing post-exploitation activity.
