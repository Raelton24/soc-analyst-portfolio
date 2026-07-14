# Windows Threat Detection

> **Portfolio Repository** — Investigating the Windows Attack Lifecycle using Host Telemetry, Windows Event Logs, Sysmon, and the MITRE ATT&CK Framework.

---

## Repository Overview

This repository documents a series of Windows threat investigations covering the complete attack lifecycle—from initial compromise through ransomware impact.

Rather than focusing on malware signatures or individual alerts, the investigations emphasize **behaviour-based detection**, demonstrating how endpoint telemetry can be correlated to reconstruct attacker activity, identify indicators of compromise (IOCs), and support incident response.

The investigations are mapped to the **MITRE ATT&CK** framework and reflect the workflow commonly followed by SOC analysts during real-world endpoint investigations.

---

# Investigation Objectives

Throughout this series, the primary objectives were to:

- Investigate Windows endpoint telemetry.
- Analyse attacker behaviour using Windows Security Logs and Sysmon.
- Reconstruct attack timelines.
- Correlate multiple log sources.
- Identify Indicators of Compromise (IOCs).
- Map attacker activity to the MITRE ATT&CK framework.
- Identify behavioural detection opportunities.
- Document incident response methodologies.

---

# Investigation Coverage

This repository includes investigations covering the major stages of a Windows intrusion:

- Windows Logging
- Initial Access
- Credential Access
- Discovery
- Collection
- Ingress Tool Transfer
- Command & Control
- Persistence
- Impact
- Ransomware

Each investigation follows a structured methodology including:

- Executive Summary
- Incident Overview
- Investigation Workflow
- Investigation Findings
- Indicators of Compromise
- MITRE ATT&CK Mapping
- Detection Opportunities
- SOC Investigation Guidance
- Lessons Learned
- Analyst Reflection

---

# Repository Structure

```text
Windows-Threat-Detection/
│
├── README.md
│
├── Investigations/
│   ├── 01-Endpoint Telemetry & Windows Logging Investigation.md
│   ├── 02-Initial-Access-Credential-Access.md
│   ├── 03-Discovery-Collection.md
│   ├── 04-Ingress-Tool-Transfer-Command-Control.md
│   ├── 05-Persistence-Investigation.md
│   └── 06-Impact-Ransomware-Investigation.md
│

```

---

# Telemetry Sources

The investigations leverage multiple Windows telemetry sources to reconstruct attacker behaviour.

## Windows Event Logs

- Authentication Events
- Account Management
- Service Installation
- Scheduled Task Creation

## Sysmon

- Process Creation
- Network Connections
- File Creation
- Registry Modifications
- DNS Activity

## Additional Evidence

- PowerShell Operational Logs
- Windows Services
- Registry
- Startup Folder
- Scheduled Tasks
- File System Artifacts

Correlating these telemetry sources provides significantly greater visibility than analysing individual logs independently.

---

# Investigation Methodology

Each investigation follows a consistent workflow:

```text
Alert
   │
   ▼
Evidence Collection
   │
   ▼
Event Correlation
   │
   ▼
Timeline Reconstruction
   │
   ▼
IOC Identification
   │
   ▼
MITRE ATT&CK Mapping
   │
   ▼
Detection Opportunities
   │
   ▼
Incident Documentation
```

---

# MITRE ATT&CK Coverage

The investigations collectively demonstrate detection opportunities across multiple ATT&CK tactics, including:

- Initial Access
- Execution
- Persistence
- Privilege Escalation
- Credential Access
- Discovery
- Collection
- Command & Control
- Exfiltration
- Impact

Rather than examining each tactic in isolation, the investigations illustrate how attackers progress through the attack lifecycle and how defenders can identify malicious behaviour before business impact occurs.

---

# Skills Demonstrated

- Windows Threat Hunting
- Windows Security Event Log Analysis
- Sysmon Analysis
- Host-Based Forensics
- Endpoint Investigation
- Behavioural Detection
- Incident Response
- IOC Identification
- Timeline Reconstruction
- MITRE ATT&CK Mapping
- Detection Engineering
- Incident Documentation

---

# Key Takeaways

Several important defensive principles emerged throughout this investigation series:

- Effective detection depends on correlating multiple telemetry sources rather than relying on individual events.
- Native Windows tools are frequently abused during post-compromise activity.
- Behaviour-based detection is more resilient than signature-based detection.
- Persistence mechanisms often provide multiple opportunities for early detection.
- Ransomware is typically the final stage of an attack, not the beginning.
- The greatest opportunity to prevent business impact is during the early phases of the intrusion.

---

# Analyst Reflection

This repository represents a complete Windows endpoint investigation journey, demonstrating how individual security events can be correlated into a coherent attack narrative. Rather than focusing solely on isolated alerts, the investigations emphasize understanding attacker behaviour across the entire intrusion lifecycle. The experience reinforced the importance of structured investigations, behavioural analytics, and MITRE ATT&CK mapping in enabling SOC analysts to detect, investigate, and contain threats before they progress to data theft or ransomware deployment.
