# Initial Access & Credential Access Investigation

> **Portfolio Case Study** — Investigating Windows Authentication Attacks and Unauthorized Access

---

# Executive Summary

This investigation documents the analysis of a simulated Windows authentication attack involving repeated failed logon attempts followed by a successful Remote Desktop Protocol (RDP) login. The objective was to identify brute-force behaviour, determine whether the attacker successfully authenticated, reconstruct the authentication timeline, and assess evidence of credential abuse.

The investigation focused on Windows Security Event Logs and Sysmon telemetry to identify authentication anomalies, correlate attacker activity, preserve indicators of compromise (IOCs), map observed behaviour to the MITRE ATT&CK framework, and identify opportunities for behavioural detection.

---

# Incident Overview

Security monitoring detected a large number of failed authentication attempts against a Windows endpoint followed by a successful Remote Desktop login.

The investigation aimed to determine:

- Whether the activity represented a brute-force attack.
- Which account was targeted.
- Which source IP performed the attack.
- Whether authentication succeeded.
- What actions occurred immediately after the successful login.
- Whether additional attacker activity indicated progression into later ATT&CK tactics.

---

# Investigation Objectives

- Detect brute-force authentication attempts.
- Identify compromised accounts.
- Determine attacker source IP.
- Correlate failed and successful logons.
- Identify attacker logon type.
- Reconstruct authentication timeline.
- Preserve authentication evidence.
- Recommend behavioural detections.

---

# Environment

## Operating System

- Microsoft Windows

## Data Sources

- Windows Security Event Logs
- Sysmon
- Event Viewer

## Investigation Tools

- Event Viewer
- Windows Security Logs
- Sysmon Logs

---

# Investigation Workflow

```text
Authentication Alert
        │
        ▼
Review Failed Logons
(Event ID 4625)
        │
        ▼
Identify Source IP
        │
        ▼
Locate Successful Logon
(Event ID 4624)
        │
        ▼
Determine Logon Type
        │
        ▼
Correlate Logon Session
(Logon ID)
        │
        ▼
Review Child Processes
(Sysmon)
        │
        ▼
Build Attack Timeline
        │
        ▼
MITRE ATT&CK Mapping
```

---

# Investigation Evidence

## Authentication Failures

The investigation began by reviewing Windows Security Event **4625 (Failed Logon)** events.

Repeated authentication failures against the same account over a short period indicated password guessing activity consistent with a brute-force attack.

Important fields examined included:

- Account Name
- Source Network Address
- Logon Type
- Failure Reason
- Authentication Package
- Workstation Name
- Time Generated

---

## Successful Authentication

After multiple failed attempts, Windows Security Event **4624 (Successful Logon)** confirmed that the attacker successfully authenticated.

The following fields became critical:

- Account Name
- Logon Type
- Source Network Address
- Authentication Package
- Logon ID

The Logon ID was used to correlate subsequent attacker activity across additional Windows events.

---

## Logon Types Investigated

| Logon Type | Meaning              |
| ---------- | -------------------- |
| 2          | Interactive Logon    |
| 3          | Network Logon        |
| 10         | Remote Desktop (RDP) |

Special attention was given to **Logon Type 10**, as it indicates a successful Remote Desktop session.

---

## Authentication Timeline

Typical attack progression observed:

```text
Repeated Failed Logons
        │
        ▼
Successful Authentication
        │
        ▼
Remote Desktop Session
        │
        ▼
Command Execution
        │
        ▼
Discovery
        │
        ▼
Collection
```

---

# Investigation Findings

The investigation identified behavioural characteristics commonly associated with credential attacks:

- High volume of failed logons.
- Repeated authentication attempts targeting a privileged account.
- Successful authentication following multiple failures.
- Remote Desktop access established after successful login.
- Authentication events that served as the starting point for later attacker activity.

These findings demonstrated how authentication telemetry provides the earliest evidence of an active compromise.

---

# Indicators of Compromise (IOCs)

| IOC            | Observation                              |
| -------------- | ---------------------------------------- |
| Authentication | Multiple failed logons                   |
| Authentication | Successful login after repeated failures |
| RDP            | Logon Type 10 observed                   |
| Security Logs  | Event ID 4625                            |
| Security Logs  | Event ID 4624                            |
| Correlation    | Shared Logon ID across attacker activity |

---

# MITRE ATT&CK Mapping

| Tactic            | Technique                |
| ----------------- | ------------------------ |
| Initial Access    | External Remote Services |
| Credential Access | Brute Force              |
| Credential Access | Valid Accounts           |
| Execution         | Remote Services          |

---

# Detection Opportunities

- Alert on repeated Event ID 4625 activity.
- Detect multiple authentication failures from a single source IP.
- Detect successful authentication immediately following numerous failures.
- Alert on privileged account logons originating from uncommon hosts.
- Correlate Event ID 4624 with Sysmon Process Creation events.
- Detect unusual RDP activity outside normal business hours.
- Monitor authentication patterns that deviate from established user baselines.

---

# SOC Investigation Guide

## Initial Questions

- Which account was targeted?
- Which source IP initiated authentication?
- Was authentication successful?
- Which authentication package was used?
- What logon type occurred?
- Were administrative accounts involved?
- Which processes executed after authentication?

## Critical Fields

- Account Name
- Source IP
- Logon Type
- Logon ID
- Authentication Package
- Time Created
- Failure Reason

## Evidence to Preserve

- Security Event Logs
- Sysmon Logs
- RDP Logs
- Authentication Timeline
- Process Tree
- Parent-Child Process Relationships

---

# Skills Demonstrated

- Windows Authentication Investigation
- Security Event Log Analysis
- RDP Investigation
- Credential Access Analysis
- Timeline Reconstruction
- IOC Identification
- Behavioural Detection
- MITRE ATT&CK Mapping
- Incident Documentation
- SOC Investigation Methodology

---

# Lessons Learned

- Authentication events often provide the earliest indication of compromise.
- Successful logons should never be analysed independently from failed attempts.
- Logon IDs enable investigators to reconstruct complete attacker sessions.
- Behavioural correlation provides significantly greater investigative value than analysing individual events in isolation.
- Early detection during Initial Access greatly reduces the likelihood of attacker progression into later stages of the kill chain.

---

# Analyst Reflection

This investigation reinforced the importance of Windows authentication telemetry as the foundation of host-based threat detection. While individual failed logons are common in enterprise environments, correlating repeated authentication failures with a subsequent successful Remote Desktop session revealed clear evidence of malicious behaviour. The investigation demonstrated how Security Event Logs provide the initial context required to reconstruct attacker activity and support rapid incident response before the compromise advances into discovery, persistence, or data theft.
