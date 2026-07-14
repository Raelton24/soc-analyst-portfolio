# Windows Logging Investigation

> **Portfolio Case Study** — Windows Host Telemetry & Threat Detection Investigation

---

# Executive Summary

This investigation documents the analysis of Windows host telemetry used throughout the Windows Threat Detection series to detect attacker activity across the MITRE ATT&CK lifecycle.

Rather than focusing on malware signatures, the investigation examined how Windows Security Logs, Sysmon, and PowerShell Operational Logs provide visibility into attacker behaviour including authentication attempts, process execution, file creation, registry modification, persistence mechanisms, and command execution.

The objective was to understand how defenders can reconstruct attacker activity by correlating multiple telemetry sources into a complete attack timeline.

---

# Incident Overview

A Windows workstation was monitored during multiple simulated attack scenarios representing different stages of the MITRE ATT&CK framework.

Instead of investigating a single malware family, the investigation focused on understanding how different attacker behaviours appear within Windows telemetry and how those events can be correlated to detect an intrusion from initial compromise through persistence and eventual impact.

---

# Investigation Objectives

- Understand Windows logging architecture.
- Identify primary host telemetry sources.
- Examine Windows Security Event Logs.
- Investigate Sysmon telemetry.
- Analyse PowerShell Operational Logs.
- Understand event correlation.
- Build investigation timelines.
- Establish the telemetry foundation used throughout later investigations.

---

# Environment

## Operating System

- Microsoft Windows

## Telemetry Sources

- Windows Security Event Logs
- Sysmon
- PowerShell Operational Logs
- Windows Registry
- Windows Services
- Scheduled Tasks
- Startup Folder
- Run Registry Keys
- Windows File System

## Investigation Tools

- Windows Event Viewer
- PowerShell
- Sysmon
- Windows Security Logs

---

# Investigation Workflow

```
Windows Activity
        │
        ▼
Security Event Logs
        │
        ▼
Sysmon Events
        │
        ▼
PowerShell Logs
        │
        ▼
Artifact Correlation
        │
        ▼
Timeline Reconstruction
        │
        ▼
MITRE ATT&CK Mapping
        │
        ▼
Detection Opportunities
```

---

# Windows Telemetry Investigated

## Windows Security Logs

Primary source for:

- Authentication
- User creation
- Privilege changes
- Service installation
- Scheduled task creation
- Account modifications

---

## Sysmon

Extended endpoint visibility through:

- Process Creation
- Network Connections
- File Creation
- Registry Changes
- DNS Queries
- Driver Loading
- Image Loading

Sysmon significantly improves investigative capability beyond native Windows logging.

---

## PowerShell Operational Logs

PowerShell logging provided visibility into:

- Command execution
- Script execution
- Module loading
- Administrative automation
- Living-off-the-Land techniques

---

# Important Event IDs Observed

## Windows Security Logs

| Event ID | Description                    |
| -------- | ------------------------------ |
| 4624     | Successful Logon               |
| 4625     | Failed Logon                   |
| 4720     | User Account Created           |
| 4724     | Password Reset                 |
| 4732     | User Added to Privileged Group |
| 4697     | Service Installed              |
| 4698     | Scheduled Task Created         |

---

## Sysmon

| Event ID | Description                 |
| -------- | --------------------------- |
| 1        | Process Creation            |
| 3        | Network Connection          |
| 11       | File Creation               |
| 13       | Registry Value Modification |

These events became the primary telemetry used throughout subsequent investigations.

---

# Investigation Findings

The investigation revealed several key observations:

- Windows Security Logs provide authentication and account management visibility.
- Sysmon dramatically increases endpoint visibility by capturing detailed process, file, registry, DNS, and network activity.
- PowerShell logging exposes attacker abuse of native administrative tools.
- No single log source provides sufficient context to investigate an attack independently.
- Correlating multiple telemetry sources is essential for accurate incident reconstruction.

---

# Windows Artifacts Examined

Throughout the investigation, the following artifacts were identified as valuable sources of evidence:

- Security Event Logs
- Sysmon Operational Logs
- PowerShell Logs
- Registry Keys
- Startup Folder
- Scheduled Tasks
- Windows Services
- User Profiles
- Process Trees
- File Metadata

Each artifact contributes unique evidence when reconstructing attacker activity.

---

# Detection Opportunities

The investigation highlighted several behavioural detection opportunities:

- Monitor repeated authentication failures.
- Detect unexpected PowerShell execution.
- Alert on suspicious parent-child process relationships.
- Monitor creation of new services and scheduled tasks.
- Detect registry modifications affecting persistence locations.
- Correlate process execution with outbound network connections.
- Monitor abnormal file creation within sensitive directories.

---

# MITRE ATT&CK Mapping

| Tactic               | Purpose                       |
| -------------------- | ----------------------------- |
| Initial Access       | Entry into the system         |
| Execution            | Command execution             |
| Persistence          | Long-term access              |
| Privilege Escalation | Elevated permissions          |
| Credential Access    | Authentication abuse          |
| Discovery            | Host reconnaissance           |
| Collection           | Data gathering                |
| Command and Control  | Remote attacker communication |
| Exfiltration         | Data theft                    |
| Impact               | Business disruption           |

---

# Skills Demonstrated

- Windows Event Log Analysis
- Sysmon Investigation
- PowerShell Analysis
- Process Investigation
- Authentication Analysis
- Registry Analysis
- Endpoint Forensics
- MITRE ATT&CK Mapping
- Incident Timeline Reconstruction
- Host-Based Threat Detection

---

# Lessons Learned

- Effective Windows investigations require correlating multiple telemetry sources.
- Behavioural analysis is more resilient than relying solely on signatures.
- Sysmon significantly enhances host visibility beyond native Windows logging.
- Windows artifacts provide valuable evidence throughout every stage of the attack lifecycle.
- Early visibility into attacker behaviour increases the likelihood of successful detection and containment.

---

# Analyst Reflection

This investigation established the foundation for every subsequent Windows investigation within this portfolio. Rather than treating Windows logs as isolated records, the analysis demonstrated how Security Logs, Sysmon, and PowerShell telemetry complement one another to reveal attacker behaviour. The most valuable lesson was that successful threat detection depends on correlation and context rather than individual events, enabling defenders to reconstruct the complete attack lifecycle from initial access through impact.
