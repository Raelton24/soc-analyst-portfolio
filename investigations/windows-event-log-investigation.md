# Windows Event Log Investigation

## Overview

Windows Event Logs provide critical forensic evidence for detecting, investigating, and reconstructing malicious activity on Windows endpoints. Every authentication attempt, process execution, account modification, PowerShell command, file operation, and network connection can leave valuable artifacts that help Security Operations Center (SOC) analysts understand attacker behavior.

This project demonstrates a structured methodology for investigating Windows endpoint activity by correlating native Windows Security Logs with Sysmon telemetry. The investigation covers authentication events, user account management, process execution, file system activity, registry modifications, network communication, DNS resolution, and PowerShell command history to reconstruct an end-to-end attack timeline.

Rather than analyzing individual events in isolation, this project emphasizes evidence correlation using identifiers such as **Logon ID** and **Process ID**, enabling analysts to connect related activities across multiple log sources and build a complete picture of attacker actions from initial access through post-exploitation.

The investigation methodology documented in this repository reflects practical SOC workflows used during endpoint investigations and incident response.

---

# Environment

| Component                 | Details                                                                                   |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| Operating System          | Microsoft Windows                                                                         |
| Log Sources               | Windows Security Logs, Sysmon, PowerShell History                                         |
| Analysis Tools            | Event Viewer, Sysmon, PowerShell                                                          |
| Investigation Focus       | Authentication, Endpoint Activity, Process Creation, Network Connections, DNS, PowerShell |
| Investigation Methodology | Event Correlation & Timeline Analysis                                                     |
| Threat Framework          | MITRE ATT&CK                                                                              |

---

# Objectives

The objectives of this investigation were to:

- Investigate Windows authentication events.
- Detect brute-force and password spraying attacks.
- Identify unauthorized account creation and privilege escalation.
- Analyze endpoint process execution using Sysmon.
- Investigate malware downloads and execution chains.
- Trace outbound network communication and DNS activity.
- Review PowerShell command history for evidence of post-exploitation.
- Correlate Windows Security Logs, Sysmon events, and PowerShell activity into a unified attack timeline.
- Identify Indicators of Compromise (IOCs).
- Map attacker behavior to the MITRE ATT&CK framework.
- Produce structured investigation documentation using a repeatable SOC methodology.

---

# Skills Demonstrated

This project demonstrates practical SOC analyst skills including:

### Windows Investigation

- Windows Event Log Analysis
- Windows Security Event Analysis
- Authentication Investigation
- User Account Investigation
- Endpoint Investigation
- Incident Documentation

### Endpoint Detection

- Sysmon Analysis
- Process Creation Analysis
- Parent/Child Process Investigation
- Process Tree Reconstruction
- Registry Persistence Analysis
- File System Investigation

### Threat Hunting

- IOC Identification
- Event Correlation
- Timeline Reconstruction
- Threat Hunting Methodology
- Evidence Collection
- Attack Chain Reconstruction

### Network Investigation

- DNS Investigation
- Network Connection Analysis
- Command-and-Control Investigation
- Endpoint Telemetry Analysis

### PowerShell Analysis

- PowerShell Forensics
- Command History Investigation
- Administrative Activity Analysis
- Malware Download Investigation

### Detection Engineering

- Detection Opportunity Identification
- MITRE ATT&CK Mapping
- Investigation Playbook Development

---

# Analyst Methodology

Each investigation follows a structured methodology designed to minimize assumptions, preserve evidence, and accurately reconstruct attacker activity.

The workflow consists of the following phases:

1. Identify the initial alert or suspicious activity.
2. Collect relevant Windows Security Logs and Sysmon events.
3. Correlate authentication, process, and network activity using **Logon ID** and **Process ID**.
4. Validate suspicious artifacts across multiple log sources.
5. Build a chronological timeline of attacker activity.
6. Identify Indicators of Compromise (IOCs).
7. Map observed behavior to the MITRE ATT&CK framework.
8. Document findings and recommend containment, remediation, and future detection opportunities.

This methodology ensures investigations remain evidence-driven rather than assumption-driven.

---

# Investigation Workflow

The investigation follows a structured workflow similar to those used in enterprise Security Operations Centers.

```text
Initial Alert

↓

Authentication Investigation

↓

Session Identification

↓

User Account Review

↓

Process Investigation

↓

File Activity Analysis

↓

Registry Analysis

↓

DNS Investigation

↓

Network Connection Analysis

↓

PowerShell Investigation

↓

IOC Collection

↓

Event Correlation

↓

Timeline Reconstruction

↓

MITRE ATT&CK Mapping

↓

Incident Documentation

↓

Detection Opportunities

↓

Lessons Learned
```

# Log Sources Investigated

Successful Windows investigations rely on correlating multiple telemetry sources rather than relying on a single event. Each log source contributes a different piece of the attack timeline and, when combined, provides the context required to accurately reconstruct attacker behavior.

---

## Windows Security Log

The Windows Security Log provides visibility into authentication activity, user account management, privilege changes, and other security-related events generated by the operating system.

### Primary Event IDs Investigated

| Event ID | Description                        | Investigation Purpose                                                              |
| -------- | ---------------------------------- | ---------------------------------------------------------------------------------- |
| 4624     | Successful Logon                   | Identify successful authentication events and establish user sessions.             |
| 4625     | Failed Logon                       | Detect brute-force attacks, password spraying, and failed authentication attempts. |
| 4720     | User Account Created               | Identify unauthorized account creation used for persistence.                       |
| 4722     | User Account Enabled               | Detect re-enabled user accounts that may have been abused.                         |
| 4723     | Password Changed                   | Investigate user-initiated password modifications.                                 |
| 4724     | Password Reset                     | Identify administrative password resets that could indicate account compromise.    |
| 4725     | User Account Disabled              | Determine whether accounts were intentionally disabled during an attack.           |
| 4726     | User Account Deleted               | Identify attempts to remove evidence or disable access.                            |
| 4732     | Member Added to Security Group     | Detect privilege escalation through group membership changes.                      |
| 4733     | Member Removed from Security Group | Identify unauthorized removal from privileged groups.                              |
| 4738     | User Account Changed               | Review modifications made to existing user accounts.                               |

---

## Sysmon

Sysmon significantly extends endpoint visibility beyond the default Windows Security Logs by recording detailed process, file system, registry, DNS, and network activity.

### Primary Event IDs Investigated

| Event ID | Description        | Investigation Purpose                                                                       |
| -------- | ------------------ | ------------------------------------------------------------------------------------------- |
| 1        | Process Creation   | Analyze executable launches, command lines, hashes, and parent-child process relationships. |
| 3        | Network Connection | Identify outbound communication, command-and-control traffic, and suspicious destinations.  |
| 11       | File Creation      | Detect downloaded payloads, dropped malware, and file system modifications.                 |
| 13       | Registry Value Set | Identify persistence mechanisms and registry modifications.                                 |
| 22       | DNS Query          | Investigate domain lookups performed by suspicious processes.                               |

---

## PowerShell History

PowerShell command history provides visibility into interactive commands executed by users or attackers after PowerShell has been launched.

Unlike process creation logs, the history file records individual commands entered during a PowerShell session, making it valuable for reconstructing post-exploitation activity.

### Primary Investigation Areas

- System discovery
- Malware download commands
- Registry modifications
- Service configuration changes
- Administrative actions
- File manipulation
- Evidence removal attempts

---

# Investigation Domains

The investigation was divided into several functional domains, each contributing a different perspective to the overall attack timeline.

---

## Authentication Analysis

Authentication events establish the starting point of an investigation.

The primary objectives include identifying:

- Source IP address
- Target account
- Authentication type
- Workstation name
- Authentication success or failure
- Logon Type
- Logon ID

The **Logon ID** becomes the primary correlation value used throughout the investigation.

---

## User Account Investigation

User management events reveal whether an attacker attempted to establish persistence or elevate privileges through account manipulation.

Areas reviewed include:

- New account creation
- Account enablement
- Password resets
- Password changes
- Account deletion
- Group membership modifications
- Privilege escalation

Special attention is given to accounts added to privileged local groups such as Administrators or Remote Desktop Users.

---

## Process Investigation

Process creation events provide the context required to understand what executed on the endpoint.

Each suspicious process is evaluated by reviewing:

- Executable path
- Command line
- Parent process
- Process hashes
- Integrity level
- User context
- Process ID
- Parent Process ID

Process lineage is reconstructed to identify how malicious activity originated and propagated.

---

## File System Investigation

File creation telemetry helps determine how malware entered the endpoint and where it was stored.

Investigation focuses include:

- Download directories
- Temporary folders
- Alternate Data Streams (Zone.Identifier)
- Download source information
- Malware staging
- Persistence files

Zone.Identifier artifacts are particularly valuable because they often preserve the original download URL.

---

## Registry Investigation

Registry modifications are reviewed to determine whether malware established persistence or modified system configuration.

Common persistence locations include:

- Run Keys
- RunOnce Keys
- Services
- Startup configuration
- Windows policies
- Security-related registry values

Registry modifications are correlated with Sysmon Process Creation events to identify the process responsible for each change.

---

## Network Investigation

Network telemetry provides visibility into outbound communications initiated by suspicious processes.

Areas investigated include:

- Destination IP addresses
- Destination ports
- Protocols
- Command-and-Control traffic
- External communication
- Beaconing behavior

Network events are correlated with DNS activity to identify attacker infrastructure.

---

## DNS Investigation

DNS telemetry often provides the earliest indication of malicious communication.

Investigations focus on:

- Suspicious domains
- Failed lookups
- Domain generation patterns
- Newly observed domains
- Domain-to-IP correlation

DNS activity is correlated with outbound network connections to reconstruct external communications.

---

## PowerShell Investigation

PowerShell investigations focus on understanding what actions occurred after PowerShell was launched.

Rather than simply confirming that PowerShell executed, investigators analyze the command history to determine the attacker's objectives.

Examples include:

- Malware downloads
- System reconnaissance
- Credential discovery
- Registry modifications
- Service manipulation
- Network configuration changes
- File system operations

PowerShell activity is correlated with Sysmon and Windows Security Logs to establish execution context.

---

# Evidence & Indicators of Compromise (IOCs)

Throughout the investigation, multiple forensic artifacts are collected to support analysis, validate findings, and enable enterprise-wide threat hunting.

## Authentication Evidence

- Source IP Address
- Username
- Workstation Name
- Logon Type
- Logon ID

---

## Process Evidence

- Executable Path
- Parent Process
- Process ID
- Parent Process ID
- Command Line
- SHA256 Hash
- MD5 Hash
- Integrity Level

---

## File Evidence

- Downloaded Executables
- File Creation Time
- Download Directory
- Zone.Identifier
- HostUrl
- File Hashes

---

## Registry Evidence

- Modified Registry Key
- Registry Value
- Persistence Location
- Executing Process

---

## Network Evidence

- Destination IP Address
- Destination Port
- Protocol
- DNS Domain
- Network Connections

---

## PowerShell Evidence

- Command History
- Executed Commands
- User Context
- Associated Logon ID

---

## Timeline Evidence

- Initial Access
- Authentication Events
- Account Changes
- Process Execution
- File Creation
- Registry Modifications
- DNS Activity
- Network Connections
- PowerShell Activity

---

# MITRE ATT&CK Mapping

Observed attacker behaviors were mapped to the MITRE ATT&CK framework to standardize investigation findings, improve reporting, and identify opportunities for future detection engineering.

| Technique | Description                       |
| --------- | --------------------------------- |
| T1078     | Valid Accounts                    |
| T1021     | Remote Services                   |
| T1110.001 | Password Guessing                 |
| T1110.003 | Password Spraying                 |
| T1136     | Create Account                    |
| T1098     | Account Manipulation              |
| T1059     | Command and Scripting Interpreter |
| T1059.001 | PowerShell                        |
| T1105     | Ingress Tool Transfer             |
| T1112     | Modify Registry                   |
| T1547     | Boot or Logon Autostart Execution |
| T1071     | Application Layer Protocol        |
| T1071.004 | DNS                               |

# Detection Opportunities

One of the primary outcomes of any SOC investigation is identifying opportunities to improve future detection capabilities. Rather than simply documenting attacker activity, analysts should determine how similar behavior can be detected earlier through improved monitoring, correlation rules, and endpoint visibility.

Based on the investigation performed in this project, the following detection opportunities were identified.

---

## Authentication Detection

Develop detections for authentication anomalies such as:

- Multiple failed RDP logons originating from the same source IP address.
- Password spraying attempts targeting multiple user accounts.
- Successful authentication immediately following repeated failed logon attempts.
- Authentication originating from unexpected source IP addresses.
- Logons occurring outside approved working hours.
- Authentication from unusual workstation names.
- Remote logons using privileged accounts.

---

## User Account Monitoring

Monitor Windows Security events for unauthorized account activity including:

- New local user account creation.
- Account enablement after long periods of inactivity.
- Administrative password resets.
- Unauthorized account deletion.
- Unexpected user account modifications.
- Additions to privileged local groups such as:
  - Administrators
  - Remote Desktop Users
  - Backup Operators

Correlating these events with authentication activity provides valuable context for detecting persistence and privilege escalation.

---

## Process Monitoring

Monitor endpoint process creation for indicators including:

- Executables launched from user download directories.
- Processes executing from temporary directories.
- Randomly named executables.
- Parent-child process anomalies.
- Unsigned executables.
- Suspicious command-line arguments.
- Hashes matching known malicious samples.

Process lineage should always be reconstructed before reaching investigative conclusions.

---

## File System Monitoring

High-value detections include:

- Executable files written to user download folders.
- Creation of suspicious files within Windows Temp directories.
- Alternate Data Stream (Zone.Identifier) creation.
- Files dropped into Startup folders.
- Unexpected executable creation by web browsers.

Monitoring downloaded executables provides valuable visibility into initial malware delivery.

---

## Registry Monitoring

Monitor registry modifications that may indicate persistence, privilege escalation, or system tampering.

Priority registry locations include:

- Run Keys
- RunOnce Keys
- Services
- Startup configuration
- Windows Policies
- Security configuration

Registry events should always be correlated with the originating process.

---

## Network Monitoring

Detect suspicious outbound communication including:

- Connections to previously unseen external IP addresses.
- Outbound traffic initiated by recently executed processes.
- Communication with uncommon ports.
- Beaconing behavior.
- Unexpected external connections initiated by user applications.

Network telemetry becomes significantly more valuable when correlated with process creation events.

---

## DNS Monitoring

Develop detections for:

- Newly observed domains.
- Failed DNS lookups.
- Domain Generation Algorithm (DGA)-like patterns.
- High-frequency DNS requests.
- Domains associated with malicious infrastructure.
- DNS requests originating from suspicious processes.

DNS telemetry often provides one of the earliest indicators of compromise during malware execution.

---

## PowerShell Monitoring

PowerShell remains one of the most commonly abused administrative tools in Windows environments.

Monitoring priorities include:

- Malware download commands.
- Encoded PowerShell commands.
- Service manipulation.
- Registry modifications.
- System discovery commands.
- Credential access attempts.
- Network configuration changes.

Where possible, combine PowerShell history with enhanced logging solutions such as Script Block Logging and Sysmon.

---

## Correlation Opportunities

Individual alerts rarely provide enough context to accurately determine malicious activity.

High-confidence detections should correlate multiple telemetry sources, including:

- Windows Security Logs
- Sysmon Process Creation
- DNS Queries
- Network Connections
- Registry Modifications
- PowerShell Activity

Correlation significantly reduces false positives while improving investigation quality.

---

# Lessons Learned

This investigation reinforced several important principles that apply to Windows endpoint investigations and Security Operations Center workflows.

## Windows Security Logs Provide the Foundation

Authentication logs establish who accessed a system, when they authenticated, and how they gained access. These events often provide the initial pivot point for an investigation.

---

## Sysmon Greatly Improves Endpoint Visibility

Native Windows Security Logs alone do not provide complete visibility into endpoint activity.

Sysmon supplements Windows logging by capturing:

- Process creation
- File activity
- Registry modifications
- DNS queries
- Network connections

Together, these logs provide a much more complete picture of attacker behavior.

---

## Event Correlation Is More Valuable Than Individual Events

Individual events rarely explain an attack.

Meaningful investigations require correlating multiple log sources using identifiers such as:

- Logon ID
- Process ID
- Parent Process ID

Correlation transforms isolated events into a coherent attack narrative.

---

## Process Lineage Matters

Understanding how a suspicious process originated is often more valuable than simply identifying the process itself.

Parent-child relationships provide valuable context that helps distinguish legitimate administrative activity from malicious execution chains.

---

## PowerShell Can Reveal Post-Exploitation Activity

PowerShell history provides valuable insight into attacker objectives after initial compromise.

Even when process creation logs only indicate that PowerShell executed, command history may reveal:

- Malware downloads
- Service modifications
- Registry changes
- System discovery
- Administrative actions

---

## Investigation Should Be Evidence-Driven

Assumptions should never replace evidence.

Every conclusion reached during an investigation should be supported by:

- Log correlation
- Process analysis
- Timeline reconstruction
- Supporting forensic artifacts

---

## Documentation Is Part of Incident Response

An investigation is incomplete without clear documentation.

Well-structured documentation enables:

- Knowledge sharing
- Faster future investigations
- Detection engineering improvements
- Incident response coordination

---

# Future Enhancements

This repository will continue evolving alongside future Windows investigation projects.

Planned improvements include:

- Integrating Windows telemetry into a Wazuh SIEM deployment.
- Developing Sigma detection rules for key Windows Security and Sysmon events.
- Building Splunk SPL queries for authentication, Sysmon, and PowerShell investigations.
- Simulating Windows attack scenarios within a dedicated home lab environment.
- Expanding investigations to Active Directory authentication and domain environments.
- Automating IOC extraction and timeline generation.
- Developing custom detection content based on observed attack techniques.
- Mapping future investigations to additional MITRE ATT&CK techniques.

---

# Conclusion

Effective endpoint investigations depend on the ability to correlate multiple sources of telemetry rather than relying on isolated events. Windows Security Logs establish the foundation for understanding authentication and account activity, while Sysmon extends visibility into process execution, file creation, registry modifications, DNS resolution, and network communication. PowerShell history further enhances investigations by exposing post-exploitation commands that may not be visible through process creation logs alone.

By combining these complementary sources of evidence, analysts can reconstruct attacker behavior from initial access through execution, persistence, and command-and-control activity. This structured, evidence-driven approach enables accurate incident analysis, improves detection engineering, and strengthens future threat hunting efforts.

The methodology documented in this repository reflects practical SOC investigation workflows that can be applied to enterprise environments, security monitoring platforms, and home lab simulations. As additional Windows security topics are explored, this repository will continue to evolve as a living reference for endpoint investigations, event correlation, and incident response.
