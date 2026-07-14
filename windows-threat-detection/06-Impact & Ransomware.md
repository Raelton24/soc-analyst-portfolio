# Impact & Ransomware Investigation

> **Portfolio Case Study** — Investigating the Final Stage of a Windows Enterprise Attack: Ransomware & Business Impact

---

# Executive Summary

This investigation documents the final stage of a simulated Windows enterprise intrusion, where the attack progresses from initial compromise to business impact through ransomware deployment. Rather than treating ransomware as an isolated event, the investigation reconstructs the complete attack lifecycle and demonstrates how each earlier stage contributed to the attacker's ultimate objective.

The analysis highlights how attackers move from initial access, credential abuse, reconnaissance, data collection, command and control, and persistence before deploying ransomware to encrypt systems and disrupt business operations. The investigation also identifies the earliest opportunities for detection and emphasizes why behavioural monitoring across the attack lifecycle is critical for preventing catastrophic impact.

---

# Incident Overview

Following weeks of undetected malicious activity, the compromised Windows environment experienced widespread file encryption accompanied by ransom notes across multiple systems.

Investigation revealed that ransomware deployment was **not** the beginning of the attack—it was the final stage of a long intrusion in which the attacker had already established persistence, gathered sensitive information, communicated with external infrastructure, and prepared the environment for maximum operational disruption.

The investigation focused on reconstructing the attack timeline, identifying missed detection opportunities, and documenting defensive measures that could have prevented the final impact.

---

# Investigation Objectives

- Reconstruct the complete attack lifecycle.
- Identify attacker objectives.
- Determine the relationship between persistence and ransomware deployment.
- Understand the business impact of ransomware.
- Identify behavioural indicators before encryption.
- Map the attack to MITRE ATT&CK.
- Recommend early-stage detection opportunities.
- Document lessons for SOC operations.

---

# Environment

## Operating System

- Microsoft Windows

## Telemetry Sources

- Windows Security Event Logs
- Sysmon
- PowerShell Operational Logs
- Process Creation Events
- File Creation Events
- Registry Activity
- Network Connections
- DNS Activity

## Investigation Tools

- Event Viewer
- Sysmon
- Windows Security Logs
- Endpoint Telemetry
- MITRE ATT&CK Framework

---

# Complete Attack Timeline

```text
Phishing / RDP Compromise
          │
          ▼
Credential Access
          │
          ▼
Discovery
          │
          ▼
Collection
          │
          ▼
Ingress Tool Transfer
          │
          ▼
Command & Control
          │
          ▼
Persistence
          │
          ▼
Data Exfiltration
          │
          ▼
Ransomware Deployment
          │
          ▼
Business Impact
```

This sequence demonstrates that ransomware is typically the **final objective**, not the initial attack.

---

# Investigation 1 — Initial Access

## Goal

Determine how the attacker first entered the environment.

Common entry vectors included:

- Phishing attachments
- Weak or exposed RDP services
- Vulnerable public-facing applications
- Malicious removable media

Early detection at this stage offers the highest chance of preventing attacker progression.

---

# Investigation 2 — Internal Reconnaissance

## Goal

Understand how the attacker learned about the compromised environment.

Observed activities included:

- Host discovery
- User enumeration
- Network configuration analysis
- Process enumeration
- Software discovery
- File and directory discovery

Reconnaissance enabled the attacker to identify valuable assets and privileged accounts before taking further action.

---

# Investigation 3 — Data Collection & Exfiltration

## Goal

Determine whether sensitive information was stolen before encryption.

Observed behaviour included:

- Collection of business documents.
- Archive creation.
- Temporary staging of files.
- Outbound communication with attacker-controlled infrastructure.

This reflects the modern **double extortion** model, where attackers steal data before encrypting systems.

---

# Investigation 4 — Persistence

## Goal

Identify how long-term access was maintained.

Persistence mechanisms observed included:

- New local user accounts.
- Privileged group membership changes.
- Scheduled Tasks.
- Windows Services.
- Registry Run Keys.
- Startup Folder persistence.

These techniques ensured continued access even if one persistence mechanism was removed.

---

# Investigation 5 — Ransomware Deployment

## Goal

Determine the final impact on the compromised environment.

Typical ransomware behaviour includes:

- Encryption of local files.
- Encryption of shared network resources.
- Deletion of recovery options.
- Disabling security controls.
- Termination of business-critical services.
- Display of ransom notes.
- Demand for cryptocurrency payment.

The investigation concluded that encryption represented the culmination of a much longer intrusion rather than an isolated event.

---

# Business Impact Assessment

Potential consequences of successful ransomware deployment include:

- Loss of business operations.
- Financial losses due to downtime.
- Exposure of confidential information.
- Regulatory penalties.
- Damage to organisational reputation.
- Recovery costs.
- Incident response expenses.
- Customer service disruption.

Modern ransomware attacks frequently combine operational disruption with threats to publicly release stolen information.

---

# Indicators of Compromise (IOCs)

| IOC            | Observation                         |
| -------------- | ----------------------------------- |
| Authentication | Repeated failed logons              |
| Authentication | Successful privileged login         |
| Sysmon         | Process Creation (Event ID 1)       |
| Sysmon         | Network Connections (Event ID 3)    |
| Sysmon         | File Creation (Event ID 11)         |
| Sysmon         | Registry Modification (Event ID 13) |
| Security Logs  | User account creation               |
| Security Logs  | Scheduled Task creation             |
| Security Logs  | Service installation                |
| File System    | Archive creation                    |
| Network        | Command & Control traffic           |
| Endpoint       | Ransom notes                        |
| Endpoint       | Mass file encryption                |

---

# MITRE ATT&CK Mapping

| Tactic            | Technique                         |
| ----------------- | --------------------------------- |
| Initial Access    | Phishing                          |
| Initial Access    | External Remote Services          |
| Credential Access | Brute Force                       |
| Discovery         | System Information Discovery      |
| Discovery         | File and Directory Discovery      |
| Collection        | Data from Local System            |
| Collection        | Archive Collected Data            |
| Command & Control | Application Layer Protocol        |
| Command & Control | Ingress Tool Transfer             |
| Persistence       | Create Account                    |
| Persistence       | Boot or Logon Autostart Execution |
| Exfiltration      | Exfiltration Over Web Service     |
| Impact            | Data Encrypted for Impact         |

---

# Detection Opportunities

The investigation identified numerous opportunities where defenders could have interrupted the attack before ransomware execution:

- Detect repeated failed authentication attempts.
- Alert on successful privileged logons following brute-force activity.
- Monitor execution of Windows discovery commands.
- Detect unusual archive creation.
- Alert on outbound communication to unknown domains.
- Monitor creation of new local accounts.
- Detect additions to privileged groups.
- Alert on Scheduled Task and Windows Service creation.
- Detect Registry Run Key modifications.
- Correlate multiple persistence mechanisms occurring within a short timeframe.
- Detect large outbound data transfers before encryption.
- Monitor abnormal execution of encryption processes.

Early detection at any of these stages would significantly reduce the likelihood of successful ransomware deployment.

---

# SOC Investigation Guide

## Initial Questions

- How did the attacker gain initial access?
- Which accounts were compromised?
- Was sensitive information collected?
- Was data exfiltrated before encryption?
- Which persistence mechanisms were established?
- Which systems were encrypted?
- Were backups affected?
- Which hosts require immediate isolation?

---

## Critical Evidence Sources

### Windows Security Logs

- Authentication Events
- Account Management
- Service Installation
- Scheduled Task Creation

### Sysmon

- Process Creation
- Network Connections
- File Creation
- Registry Modifications

### Endpoint Artifacts

- Ransom Notes
- Encrypted Files
- Scheduled Tasks
- Windows Services
- Registry Run Keys

---

# Containment Priorities

Upon confirmation of ransomware activity, immediate priorities include:

1. Isolate affected endpoints from the network.
2. Disable compromised accounts.
3. Block Command & Control infrastructure.
4. Preserve volatile evidence.
5. Identify the initial point of compromise.
6. Assess the scope of encrypted systems.
7. Determine whether data exfiltration occurred.
8. Begin eradication and recovery using trusted backups.

---

# Skills Demonstrated

- Windows Threat Hunting
- Incident Response
- Endpoint Investigation
- Sysmon Analysis
- Windows Security Log Analysis
- Attack Timeline Reconstruction
- Behavioural Detection
- Ransomware Investigation
- MITRE ATT&CK Mapping
- IOC Identification
- Detection Engineering
- Incident Documentation

---

# Lessons Learned

- Ransomware is rarely the first stage of an attack; it is typically the final objective.
- Successful ransomware deployments are preceded by multiple observable attacker behaviours.
- Behavioural detection throughout the attack lifecycle provides numerous opportunities for early intervention.
- Correlating authentication, process, registry, network, and persistence telemetry significantly improves detection capability.
- Preventing ransomware depends on identifying earlier stages of the intrusion rather than focusing solely on encryption events.

---

# Analyst Reflection

This investigation concluded the Windows Threat Detection series by demonstrating how seemingly independent attacker behaviours combine into a complete intrusion lifecycle. Initial access, credential abuse, reconnaissance, collection, command and control, and persistence were not isolated events—they formed a continuous sequence that ultimately enabled ransomware deployment and business disruption. The strongest lesson from this investigation is that defenders should not wait for ransomware to execute before responding. Every stage preceding impact provides valuable behavioural indicators that, when correlated effectively, allow SOC analysts to detect, contain, and eradicate threats long before encryption or operational disruption occurs.
