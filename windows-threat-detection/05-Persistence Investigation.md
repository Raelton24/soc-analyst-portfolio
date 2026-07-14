# Persistence Investigation

> **Portfolio Case Study** — Investigating Windows Persistence Mechanisms Following Host Compromise

---

# Executive Summary

This investigation documents the analysis of multiple Windows persistence mechanisms observed following a successful host compromise. After establishing initial access and communicating with external infrastructure, the attacker implemented several persistence techniques to maintain long-term access, survive system reboots, and reduce the likelihood of losing control of the compromised endpoint.

The investigation focused on identifying malicious account creation, privileged group membership changes, password resets, Windows Services, Scheduled Tasks, Registry Run Keys, and Startup Folder persistence. Evidence from Windows Security Logs and Sysmon telemetry was correlated to reconstruct attacker activity and identify opportunities for behavioural detection.

---

# Incident Overview

Following successful authentication and post-exploitation activity, endpoint telemetry revealed multiple system modifications intended to provide long-term persistence.

The attacker established several independent persistence mechanisms to ensure continued access even if one technique was discovered or removed.

The investigation aimed to determine:

- Which persistence mechanisms were created.
- Which user accounts were modified.
- Which Windows components were abused.
- Which telemetry sources recorded the activity.
- How these behaviours could be detected in an enterprise SOC.

---

# Investigation Objectives

- Detect malicious user account creation.
- Identify privilege escalation through group membership changes.
- Detect unauthorized password resets.
- Identify malicious Windows Services.
- Detect Scheduled Task persistence.
- Investigate Registry Run Key modifications.
- Detect Startup Folder persistence.
- Correlate persistence mechanisms into a unified attack timeline.

---

# Environment

## Operating System

- Microsoft Windows

## Telemetry Sources

- Windows Security Event Logs
- Sysmon
- PowerShell Operational Logs
- Registry
- Windows Services
- Task Scheduler
- Startup Folder
- File System

## Investigation Tools

- Event Viewer
- Sysmon
- Windows Security Logs
- Registry Editor
- Services Console
- Task Scheduler

---

# Investigation Workflow

```text
Successful Compromise
        │
        ▼
Account Manipulation
        │
        ▼
Privilege Assignment
        │
        ▼
Windows Service Creation
        │
        ▼
Scheduled Task Creation
        │
        ▼
Registry Run Key Modification
        │
        ▼
Startup Folder Persistence
        │
        ▼
Long-Term Access Established
```

---

# Investigation 1 — Backdoored User Accounts

## Goal

Determine whether the attacker created additional local accounts to maintain access.

### Primary Evidence

**Windows Security Event ID 4720**

(User Account Created)

Important fields:

- Subject User
- Target User
- Computer Name
- Time Created
- Security Identifier (SID)

---

### Investigation Findings

A newly created local account represented an attempt to establish an alternative method of access independent of the originally compromised credentials.

Investigators should always verify:

- Who created the account.
- Whether the creation was authorized.
- Which session performed the action.
- Whether the account was subsequently used for authentication.

---

# Investigation 2 — Privileged Group Membership

## Goal

Determine whether attacker-controlled accounts received elevated privileges.

### Primary Evidence

**Windows Security Event ID 4732**

(Member Added to Local Security Group)

Important fields:

- Group Name
- Member Added
- Subject User
- Computer Name

---

### Investigation Findings

Attackers commonly add newly created accounts to privileged groups such as:

- Administrators
- Remote Desktop Users

Granting elevated privileges allows continued administrative access even if the originally compromised account is disabled.

---

# Investigation 3 — Password Reset Activity

## Goal

Identify unauthorized credential changes.

### Primary Evidence

**Windows Security Event ID 4724**

(Password Reset Attempt)

Important fields:

- Subject User
- Target User
- Time Created
- Computer Name

---

### Investigation Findings

Instead of creating new accounts, attackers may reset passwords for dormant or unused accounts to avoid generating obvious indicators such as account creation events.

---

# Investigation 4 — Windows Service Persistence

## Goal

Determine whether malicious services were created to launch malware automatically.

### Evidence Sources

**Windows Security Event ID 4697**

(Service Installed)

**Sysmon Event ID 1**

(Process Creation)

---

### Important Fields

- Service Name
- Service File Name
- Image Path
- Process Image
- Parent Process
- Command Line

---

### Investigation Findings

Malicious services provide persistence by launching attacker-controlled binaries during system startup.

Investigators should verify:

- Service executable location.
- Digital signature.
- Parent process.
- Startup type.
- Creation time.

---

# Investigation 5 — Scheduled Task Persistence

## Goal

Identify malicious Scheduled Tasks created after compromise.

### Primary Evidence

**Windows Security Event ID 4698**

(Scheduled Task Created)

**Sysmon Event ID 1**

(Process Creation)

---

### Investigation Findings

Scheduled Tasks remain one of the most common Windows persistence mechanisms because they are easy to configure and blend with legitimate administrative activity.

Particular attention should be paid to:

- Tasks executing PowerShell.
- Tasks launching executables from temporary folders.
- Tasks running with SYSTEM privileges.
- Tasks configured to execute at logon or startup.

---

# Investigation 6 — Registry Run Keys

## Goal

Identify malware configured to execute automatically during user logon.

### Primary Evidence

**Sysmon Event ID 13**

(Registry Value Modification)

---

### Registry Locations

Per-user:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

System-wide:

```text
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

---

### Investigation Findings

Registry Run Keys provide reliable user-level persistence and are frequently abused by commodity malware, information stealers, and remote access trojans.

---

# Investigation 7 — Startup Folder Persistence

## Goal

Determine whether malware was copied into Startup folders.

### Primary Evidence

**Sysmon Event ID 11**

(File Creation)

---

### Startup Locations

Current User:

```text
C:\Users\<User>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

All Users:

```text
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```

---

### Investigation Findings

The Startup Folder remains an effective persistence location because executables placed within it launch automatically when the user logs on.

Although legitimate software rarely uses this directory today, many malware families continue to abuse it due to its simplicity.

---

# Indicators of Compromise (IOCs)

| IOC             | Observation               |
| --------------- | ------------------------- |
| Security Log    | Event ID 4720             |
| Security Log    | Event ID 4724             |
| Security Log    | Event ID 4732             |
| Security Log    | Event ID 4697             |
| Security Log    | Event ID 4698             |
| Sysmon          | Event ID 1                |
| Sysmon          | Event ID 11               |
| Sysmon          | Event ID 13               |
| Registry        | Run Key modification      |
| File System     | Startup Folder executable |
| Services        | Newly installed service   |
| Scheduled Tasks | New startup task          |

---

# MITRE ATT&CK Mapping

| Tactic      | Technique                         |
| ----------- | --------------------------------- |
| Persistence | Create Account                    |
| Persistence | Account Manipulation              |
| Persistence | Boot or Logon Autostart Execution |
| Persistence | Scheduled Task/Job                |
| Persistence | Create or Modify System Process   |
| Persistence | External Remote Services          |

---

# Detection Opportunities

- Alert on creation of new local accounts.
- Detect additions to the Administrators or Remote Desktop Users groups.
- Monitor password reset events involving privileged accounts.
- Detect creation of new Windows Services outside approved maintenance windows.
- Alert on Scheduled Tasks executing PowerShell or temporary-directory executables.
- Monitor Registry Run Key modifications.
- Detect executable creation within Startup folders.
- Correlate persistence events occurring shortly after successful authentication.
- Alert when multiple persistence mechanisms are established within a short timeframe.

---

# SOC Investigation Guide

## Initial Questions

- Was a new account created?
- Were existing accounts modified?
- Were administrator privileges assigned?
- Was a password reset performed?
- Were new Windows Services installed?
- Were Scheduled Tasks created?
- Were Registry Run Keys modified?
- Was malware copied into a Startup folder?
- Which process initiated each persistence action?

---

## Critical Fields

### Windows Security Logs

- Subject User
- Target User
- Group Name
- Service Name
- Task Name
- Time Created

### Sysmon Event ID 1

- Image
- Parent Image
- CommandLine

### Sysmon Event ID 11

- TargetFilename

### Sysmon Event ID 13

- TargetObject
- Details

---

# Skills Demonstrated

- Windows Persistence Investigation
- Windows Security Log Analysis
- Sysmon Analysis
- Registry Forensics
- Service Analysis
- Scheduled Task Analysis
- Account Investigation
- Behavioural Detection
- MITRE ATT&CK Mapping
- Incident Documentation

---

# Lessons Learned

- Attackers rarely rely on a single persistence mechanism.
- Multiple independent persistence techniques significantly increase attacker resilience.
- Behavioural correlation provides greater detection capability than monitoring individual events.
- Windows Security Logs and Sysmon complement each other when investigating persistence.
- Detecting persistence early prevents attackers from re-establishing access after remediation.

---

# Analyst Reflection

This investigation demonstrated that persistence is not a single event but a collection of behaviours designed to ensure long-term access to a compromised system. By correlating account manipulation, privilege changes, service creation, scheduled tasks, registry modifications, and Startup Folder activity, investigators can reconstruct how an attacker attempts to survive remediation efforts. The investigation reinforced the importance of behavioural monitoring across multiple Windows telemetry sources, enabling defenders to identify persistence mechanisms before they are leveraged in future stages of the intrusion.
