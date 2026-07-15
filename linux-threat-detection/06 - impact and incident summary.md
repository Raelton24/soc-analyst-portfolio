# 06 – Impact & Incident Summary

---

# Executive Summary

This report consolidates the findings from my investigation into a simulated Linux compromise involving a public-facing application.

Through analysis of Linux auditd telemetry, authentication logs, process execution records, and file modification events, I reconstructed the complete attack lifecycle from initial compromise through post-exploitation activities.

The investigation confirmed that the attacker successfully exploited a vulnerable public-facing application, established an interactive reverse shell, performed host reconnaissance, escalated privileges to the root account, accessed sensitive credentials, deployed attacker-controlled tooling, and established multiple persistence mechanisms to maintain long-term access.

The observed attack chain closely aligns with techniques commonly employed during real-world Linux intrusions targeting Internet-facing infrastructure.

---

# Incident Summary

The investigation reconstructed a complete compromise of a Linux host from the point of initial access to the establishment of persistent administrative control.

The attacker successfully progressed through multiple stages of the MITRE ATT&CK framework without evidence of interruption.

The reconstructed attack sequence demonstrates a structured and deliberate intrusion rather than opportunistic or automated activity.

The compromise progressed through the following phases:

- Initial Access
- Execution
- Discovery
- Privilege Escalation
- Credential Access
- Ingress Tool Transfer
- Malware Deployment
- Persistence

No evidence of defensive controls interrupting the attack was identified within the available telemetry.

---

# Investigation Scope

The investigation included analysis of:

- Linux auditd telemetry
- Authentication logs
- Process execution events
- Parent-child process relationships
- Privilege transitions
- Sensitive file access
- Tool transfer activity
- Malware execution
- Persistence mechanisms

Each phase of the intrusion was documented separately to preserve investigative integrity and clearly demonstrate how individual attacker actions contributed to the overall compromise.

---

# Attack Timeline

| Phase                | Summary                                                                                           |
| -------------------- | ------------------------------------------------------------------------------------------------- |
| Initial Access       | Public-facing application exploited to obtain command execution.                                  |
| Execution            | Interactive reverse shell established using `socat`.                                              |
| Discovery            | Host reconnaissance performed through user, system, and directory enumeration.                    |
| Privilege Escalation | Attacker successfully transitioned from the compromised service account to the root account.      |
| Credential Access    | Sensitive configuration files containing credentials were accessed.                               |
| Tool Transfer        | Additional attacker tooling downloaded to the compromised host.                                   |
| Malware Deployment   | Downloaded tooling compiled, prepared, and executed.                                              |
| Persistence          | Administrative user account, SSH key persistence, and startup persistence mechanisms established. |

---

# Overall Assessment

Based on the available evidence, I assess that the attacker successfully achieved full administrative control of the compromised Linux system.

The intrusion demonstrated several characteristics commonly associated with hands-on-keyboard post-exploitation activity:

- Structured host reconnaissance.
- Deliberate privilege escalation.
- Credential harvesting.
- Introduction of attacker-controlled tooling.
- Establishment of redundant persistence mechanisms.

Although no destructive activity was observed during the investigation, the attacker possessed sufficient privileges to perform additional actions including:

- Data theft
- Lateral movement
- Credential reuse
- Deployment of ransomware
- Long-term espionage

---

# MITRE ATT&CK Summary

| Tactic                | Representative Techniques                                                                                             |
| --------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Initial Access        | Exploit Public-Facing Application (T1190)                                                                             |
| Execution             | Command and Scripting Interpreter (T1059.004)                                                                         |
| Command and Control   | Non-Application Layer Protocol (T1095)                                                                                |
| Discovery             | System Information Discovery (T1082), File and Directory Discovery (T1083), System Owner/User Discovery (T1033)       |
| Privilege Escalation  | Abuse Elevation Control Mechanism (T1548)                                                                             |
| Credential Access     | Credentials from Password Stores (T1555), Unsecured Credentials (T1552)                                               |
| Ingress Tool Transfer | Ingress Tool Transfer (T1105)                                                                                         |
| Persistence           | Create Account (T1136), SSH Authorized Keys (T1098.004), Scheduled Task/Cron (T1053.003), Systemd Service (T1543.002) |

---

# Detection Engineering Opportunities

Throughout the investigation, several high-confidence behavioural detection opportunities were identified.

Priority detections include:

- Public-facing applications spawning shell processes.
- Reverse shell establishment.
- Sequential Linux discovery commands.
- Privilege escalation by service accounts.
- Access to sensitive configuration files.
- Downloads into temporary directories.
- Compilation and execution of newly created binaries.
- Creation of privileged user accounts.
- Modification of `authorized_keys`.
- Creation of cron jobs and systemd services.

Rather than relying on static indicators of compromise, these detections focus on attacker behaviour and process relationships, making them more resilient against changes in malware or infrastructure.

---

# Response Recommendations

## Immediate Response

- Isolate compromised systems from the network.
- Terminate active attacker processes.
- Remove all identified persistence mechanisms.
- Rotate privileged credentials.
- Preserve forensic evidence.

---

## Threat Hunting

Conduct retrospective hunting for:

- Reverse shell execution.
- Unusual parent-child process relationships.
- Recent privilege escalation activity.
- Access to application configuration files.
- Execution from temporary directories.
- Unauthorized SSH key modifications.
- Newly created privileged accounts.

---

## Long-Term Improvements

To improve detection and response capabilities, I recommend:

- Expanding Linux auditd coverage.
- Deploying behavioural detection rules within SIEM platforms.
- Monitoring privileged account creation.
- Implementing file integrity monitoring for authentication files.
- Restricting execution from temporary directories.
- Developing correlation rules for multi-stage Linux intrusions.

---

# Conclusion

This investigation successfully reconstructed the complete lifecycle of a Linux compromise using endpoint telemetry and system logs.

By correlating process execution events, authentication records, privilege transitions, and file modification activity, I established a clear sequence of attacker actions from initial compromise through long-term persistence.

The investigation demonstrates the importance of behavioural analysis, process ancestry reconstruction, and evidence correlation when investigating Linux intrusions. While individual commands may appear benign in isolation, analysing their execution context provides high-confidence visibility into attacker objectives and significantly improves an organisation's ability to detect and contain advanced post-exploitation activity before it results in business impact.
