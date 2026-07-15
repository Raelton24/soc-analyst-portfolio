# Linux Threat Detection Investigations

## Overview

This repository documents a structured investigation into a simulated Linux compromise reconstructed using endpoint telemetry, authentication logs, and Linux auditd events.

Rather than presenting lab walkthroughs or step-by-step exercises, each report focuses on the investigative process used to identify, validate, and document attacker activity across the intrusion lifecycle. The objective is to demonstrate evidence-driven incident response, process reconstruction, behavioural analysis, and mapping to the MITRE ATT&CK framework.

The investigation follows the attack from the initial compromise of a public-facing application through post-exploitation activities, including privilege escalation, credential access, malware deployment, and long-term persistence.

---

# Investigation Objectives

During this investigation I aimed to:

- Reconstruct the complete attack timeline using Linux telemetry.
- Correlate process execution and parent-child relationships.
- Identify attacker behaviour across each phase of the intrusion.
- Validate findings using supporting evidence rather than assumptions.
- Map observed techniques to the MITRE ATT&CK framework.
- Identify behavioural detection opportunities suitable for Security Operations Centers (SOC).
- Produce incident reports following professional Digital Forensics and Incident Response (DFIR) documentation standards.

---

# Investigation Scope

The investigation includes analysis of:

- Linux auditd process execution telemetry
- Authentication logs (`/var/log/auth.log`)
- Parent-child process relationships
- Privilege transitions
- Credential access activity
- Malware staging and execution
- Persistence mechanisms
- MITRE ATT&CK mapping
- Detection engineering opportunities
- Incident response recommendations

---

# Investigation Workflow

```text
Initial Access

↓

Execution

↓

Discovery

↓

Privilege Escalation

↓

Credential Access

↓

Ingress Tool Transfer

↓

Malware Deployment

↓

Persistence

↓

Incident Summary
```

---

# Investigation Reports

| Investigation                                                  | Description                                                                                                                                                |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **01 – Initial Access & Execution**                            | Reconstruction of the initial compromise, reverse shell establishment, and execution chain originating from the public-facing application.                 |
| **02 – Discovery**                                             | Analysis of post-exploitation reconnaissance used to identify user context, operating system information, and accessible resources.                        |
| **03 – Privilege Escalation**                                  | Investigation of the attacker's transition from a compromised service account to the Linux root account through privilege escalation techniques.           |
| **04 – Credential Access, Tool Transfer & Malware Deployment** | Analysis of credential discovery, sensitive file access, external tool transfer, malware preparation, and execution.                                       |
| **05 – Persistence**                                           | Investigation of privileged account creation, SSH key persistence, and Linux startup persistence mechanisms used to maintain long-term access.             |
| **06 – Impact & Incident Summary**                             | Consolidated assessment of the complete intrusion, including attack timeline, MITRE ATT&CK mapping, detection opportunities, and response recommendations. |

---

# MITRE ATT&CK Coverage

The investigation documents techniques across multiple ATT&CK tactics, including:

- Initial Access
- Execution
- Command and Control
- Discovery
- Privilege Escalation
- Credential Access
- Ingress Tool Transfer
- Persistence

---

# Key Skills Demonstrated

This repository demonstrates practical experience in:

- Linux Incident Response
- Digital Forensics & Incident Reconstruction
- Linux auditd Analysis
- Process Tree Reconstruction
- Authentication Log Analysis
- Threat Hunting
- MITRE ATT&CK Mapping
- Detection Engineering
- Behavioural Analytics
- Security Operations (SOC) Investigation
- Technical Report Writing

---

# Detection Engineering Outcomes

Throughout the investigation, multiple behavioural detection opportunities were identified, including:

- Reverse shell detection
- Abnormal parent-child process relationships
- Linux discovery command correlation
- Privilege escalation monitoring
- Credential access detection
- Ingress tool transfer detection
- Malware execution monitoring
- SSH key persistence detection
- Privileged account creation monitoring
- Cron and systemd persistence detection

These detections focus on attacker behaviour rather than static Indicators of Compromise (IoCs), making them more resilient against evolving adversary techniques.

---

# Repository Purpose

This repository forms part of my Security Operations Center (SOC) portfolio and demonstrates my ability to:

- Conduct structured Linux incident investigations.
- Analyse endpoint telemetry to reconstruct attacker activity.
- Produce professional DFIR-style investigation reports.
- Translate technical findings into actionable detection and response recommendations.

The reports are intentionally written as incident investigations rather than training notes, reflecting the style and structure commonly used by professional incident response teams.
