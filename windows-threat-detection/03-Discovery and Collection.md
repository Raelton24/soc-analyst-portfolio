# Discovery & Collection Investigation

> **Portfolio Case Study** — Investigating Host Reconnaissance and Data Collection on Windows Endpoints

---

# Executive Summary

This investigation documents the analysis of attacker behaviour during the **Discovery** and **Collection** phases of a Windows compromise. After successfully obtaining access to a Windows endpoint, the attacker performed system reconnaissance to understand the environment before identifying, collecting, and preparing sensitive data for exfiltration.

The investigation focused on identifying reconnaissance commands, file discovery activity, archive creation, PowerShell execution, and evidence of automated data collection using Windows Security Logs and Sysmon telemetry.

---

# Incident Overview

Following successful authentication, monitoring detected unusual command execution and file system activity originating from the compromised host.

The attacker began enumerating system information, identifying users, locating sensitive documents, and staging files into temporary directories before compressing them for later exfiltration.

The objective of this investigation was to reconstruct the attacker's actions, determine what information was collected, identify affected assets, and document opportunities for behavioural detection.

---

# Investigation Objectives

- Identify host reconnaissance activity.
- Detect command-line discovery techniques.
- Identify file and directory enumeration.
- Detect collection of sensitive documents.
- Detect archive creation.
- Identify automated data stealer behaviour.
- Preserve indicators of compromise.
- Map observed behaviour to MITRE ATT&CK.

---

# Environment

## Operating System

- Microsoft Windows

## Telemetry Sources

- Sysmon
- Windows Security Event Logs
- PowerShell Operational Logs
- File System Artifacts

## Investigation Tools

- Event Viewer
- Sysmon Logs
- PowerShell Logs

---

# Investigation Workflow

```text
Successful Authentication
        │
        ▼
Host Enumeration
        │
        ▼
File Discovery
        │
        ▼
Sensitive File Access
        │
        ▼
Data Staging
        │
        ▼
Archive Creation
        │
        ▼
Preparation for Exfiltration
        │
        ▼
MITRE ATT&CK Mapping
```

---

# Investigation 1 — System Discovery

## Goal

Determine how the attacker gathered information about the compromised host.

### Common Discovery Commands

| Command                         | Purpose                              |
| ------------------------------- | ------------------------------------ |
| `whoami`                        | Identify current user                |
| `hostname`                      | Determine system name                |
| `systeminfo`                    | Collect operating system information |
| `ipconfig /all`                 | Network configuration                |
| `tasklist`                      | Running processes                    |
| `net user`                      | Enumerate local users                |
| `net localgroup administrators` | Identify privileged users            |
| `dir`                           | Browse directories                   |
| `tree`                          | Directory structure                  |
| `wmic`                          | System inventory                     |

---

### Sysmon Evidence

Primary telemetry:

**Sysmon Event ID 1**

(Process Creation)

Important fields examined:

- Image
- Parent Image
- CommandLine
- User
- Current Directory
- Process GUID
- Parent Process GUID

---

### Investigation Findings

Evidence showed repeated execution of Windows-native administrative utilities (Living-off-the-Land Binaries) to enumerate:

- Host identity
- Operating system
- Running processes
- Installed software
- User accounts
- Network configuration

The behaviour was consistent with attacker reconnaissance immediately following initial access.

---

# Investigation 2 — File & Directory Discovery

## Goal

Determine how the attacker identified valuable information.

### Observed Behaviour

The attacker searched recursively for valuable files including:

- PDF
- DOCX
- XLSX
- TXT
- Configuration files
- Financial documents

---

### Common PowerShell Activity

Examples of observed behaviour included:

- Recursive directory searches
- File filtering
- User profile enumeration
- Searching desktop and document folders

---

### Evidence Sources

Primary telemetry:

- Sysmon Event ID 1
- PowerShell Operational Logs

Critical fields:

- CommandLine
- Image
- Parent Process
- User
- Working Directory

---

# Investigation 3 — Collection

## Goal

Determine how sensitive information was gathered prior to exfiltration.

### Collection Behaviour

Observed attacker activity included:

- Opening sensitive documents.
- Copying files into temporary staging directories.
- Collecting application data.
- Gathering browser information.
- Preparing data for compression.

---

### File Creation Activity

Primary telemetry:

**Sysmon Event ID 11**

(File Creation)

Important fields:

- TargetFilename
- Image
- User
- Creation Time

---

### Archive Creation

Collected files were consolidated into compressed archives before exfiltration.

Common techniques observed:

| Tool             | Purpose                       |
| ---------------- | ----------------------------- |
| Compress-Archive | Native PowerShell compression |
| 7-Zip            | Archive creation              |
| ZIP              | Data staging                  |

Archive creation often represented the final step before outbound data transfer.

---

# Investigation 4 — Automated Data Stealer Behaviour

Unlike human operators, automated information stealers performed collection without relying on interactive command-line utilities.

Observed objectives included:

- Browser credentials
- Browser sessions
- VPN profiles
- Cryptocurrency wallets
- Messaging applications
- Clipboard data
- Screenshots
- User documents

These activities were implemented through native application code rather than visible command execution, making behavioural detection significantly more important than signature-based detection.

---

# Indicators of Compromise

| IOC           | Observation                       |
| ------------- | --------------------------------- |
| Sysmon        | Event ID 1                        |
| Sysmon        | Event ID 11                       |
| PowerShell    | Recursive file searches           |
| File System   | Temporary staging directories     |
| Archive       | ZIP file creation                 |
| LOLBins       | Native Windows discovery commands |
| User Activity | Access to sensitive documents     |

---

# MITRE ATT&CK Mapping

| Tactic     | Technique                    |
| ---------- | ---------------------------- |
| Discovery  | System Information Discovery |
| Discovery  | File and Directory Discovery |
| Discovery  | Account Discovery            |
| Discovery  | Process Discovery            |
| Collection | Data from Local System       |
| Collection | Automated Collection         |
| Collection | Archive Collected Data       |
| Collection | Clipboard Data               |

---

# Detection Opportunities

- Alert on excessive execution of Windows discovery commands.
- Detect recursive PowerShell searches targeting user directories.
- Monitor access to sensitive business document types.
- Detect creation of large archive files in temporary directories.
- Alert on execution of compression utilities immediately after file discovery.
- Correlate document access with subsequent archive creation.
- Detect unusual clipboard access by uncommon processes.
- Monitor repeated access to browser profile directories.

---

# SOC Investigation Guide

## Initial Questions

- What commands were executed?
- Which user executed them?
- Were administrative tools abused?
- Which directories were searched?
- Which files were accessed?
- Was data copied to a staging directory?
- Was an archive created?
- Which process created the archive?

---

## Critical Fields

### Sysmon Event ID 1

- Image
- Parent Image
- CommandLine
- User

### Sysmon Event ID 11

- TargetFilename
- Image
- User
- Creation Time

### PowerShell Logs

- Script Block
- Command
- User
- Process ID

---

# Skills Demonstrated

- Windows Host Investigation
- Sysmon Analysis
- PowerShell Investigation
- File System Forensics
- Process Analysis
- Living-off-the-Land Detection
- Behavioural Threat Hunting
- IOC Identification
- MITRE ATT&CK Mapping
- Incident Documentation

---

# Lessons Learned

- Discovery activity often provides the earliest indication of attacker intent after compromise.
- Native Windows administration tools are frequently abused during reconnaissance.
- Archive creation is a strong behavioural indicator when correlated with document discovery.
- Automated data stealers rely heavily on native API calls, reducing command-line visibility.
- Correlating process execution, file creation, and PowerShell activity significantly improves investigative confidence.

---

# Analyst Reflection

This investigation demonstrated how attackers transition from gaining access to understanding and exploiting a compromised environment. Rather than relying on malware-specific indicators, the analysis focused on behavioural patterns including system enumeration, document discovery, file staging, and archive creation. These activities collectively reveal the attacker's objective of preparing sensitive information for exfiltration and reinforce the importance of correlating endpoint telemetry to identify malicious behaviour before data leaves the network.
