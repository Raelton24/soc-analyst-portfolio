# 02 – Discovery Investigation

---

# Executive Summary

This investigation documents my analysis of the **Discovery** phase of the intrusion following successful establishment of the reverse shell.

Using Linux auditd telemetry and process execution records, I reconstructed the attacker's post-exploitation reconnaissance activities to determine what information was collected before attempting further compromise.

The evidence showed a structured sequence of host discovery commands focused on identifying the current user context, operating system, working directory, and accessible files. While each command is legitimate in isolation, their execution order, timing, and common parent process indicate deliberate reconnaissance conducted immediately after gaining interactive access.

The findings demonstrate that the attacker was preparing the environment for subsequent privilege escalation and post-exploitation activities.

---

# Incident Overview

Following establishment of the reverse shell documented in the previous investigation, I shifted my analysis towards understanding the attacker's immediate objectives.

Rather than performing destructive actions, the attacker first executed a series of reconnaissance commands designed to understand the compromised host.

The purpose of this investigation was to determine:

- What information the attacker attempted to collect.
- Why those commands were executed.
- How the discovery activity was performed.
- Which behavioural detections could identify similar reconnaissance in production environments.

---

# Data Sources

The investigation relied on the following telemetry sources.

| Data Source                        | Purpose                                   |
| ---------------------------------- | ----------------------------------------- |
| Linux auditd                       | Process execution telemetry               |
| Parent-Child Process Relationships | Reconstruction of attacker activity       |
| Command-Line Arguments             | Validation of executed discovery commands |
| Execution Timeline                 | Sequence reconstruction                   |

---

# Investigation Methodology

To reconstruct the reconnaissance phase, I began with the reverse shell process identified during the previous investigation and enumerated every child process it spawned.

Instead of analysing commands individually, I correlated execution order, timestamps, parent-child relationships, and command-line arguments to understand the attacker's objectives and decision-making process.

The investigation followed the workflow below.

```text
Reverse Shell Confirmed

↓

Enumerate Child Processes

↓

Reconstruct Command Sequence

↓

Identify Information Collected

↓

Assess Attacker Objectives

↓

Map to MITRE ATT&CK

↓

Develop Behavioural Detections
```

---

# Investigation Findings

## User Context Enumeration

The first commands executed after the reverse shell was established were used to determine the identity and privilege level of the compromised account.

**Observed Commands**

```bash
whoami

id
```

These commands allowed the attacker to determine:

- Which account had been compromised.
- Group memberships associated with the account.
- Whether elevated privileges were already available.

**Assessment**

This behaviour is consistent with an attacker validating the current execution context before attempting privilege escalation.

---

## Operating System Discovery

After identifying the compromised account, the attacker gathered information about the operating system.

**Observed Command**

```bash
uname
```

The collected information could later be used to identify:

- Kernel version
- Operating system release
- Potential privilege escalation vulnerabilities
- Compatibility with publicly available exploits

**Assessment**

System information discovery is a common precursor to privilege escalation and exploit selection.

---

## Environment Enumeration

The attacker next identified the execution context of the compromised process.

**Observed Command**

```bash
pwd
```

This provided immediate awareness of:

- Current working directory.
- Application execution location.
- Relative paths available for further exploration.

The attacker then enumerated accessible files and directories.

**Observed Command**

```bash
ls
```

Directory enumeration likely assisted in identifying:

- Configuration files
- Credentials
- Application resources
- Sensitive data
- Potential privilege escalation opportunities

**Assessment**

The observed activity demonstrates deliberate environmental reconnaissance rather than routine system administration.

---

## Discovery Sequence Reconstruction

Using auditd telemetry, I reconstructed the complete reconnaissance workflow.

```text
Interactive Reverse Shell

↓

whoami

↓

id

↓

uname

↓

pwd

↓

ls
```

Although every command is a legitimate Linux administration utility, their rapid sequential execution immediately following reverse shell establishment strongly indicates manual attacker reconnaissance.

**Assessment**

The reconstructed sequence demonstrates a structured information-gathering phase that commonly precedes privilege escalation.

---

# Evidence Supporting Findings

The findings were supported by:

- auditd process execution events
- Reverse shell process ancestry
- Child process reconstruction
- Command-line arguments
- Process execution timestamps

---

# MITRE ATT&CK Mapping

| Tactic    | Technique                    | Technique ID |
| --------- | ---------------------------- | ------------ |
| Discovery | System Owner/User Discovery  | T1033        |
| Discovery | System Information Discovery | T1082        |
| Discovery | File and Directory Discovery | T1083        |

---

# Detection Engineering Opportunities

## High-Confidence Detections

- Detect multiple Linux discovery commands executed within a short time window.
- Detect discovery commands executed immediately after reverse shell establishment.
- Detect application service accounts executing reconnaissance commands.
- Detect abnormal parent-child relationships linking web applications to shell-based discovery activity.

## Correlation Opportunity

```text
Reverse Shell

↓

whoami

↓

id

↓

uname

↓

pwd

↓

ls

↓

High-Severity SOC Alert
```

Correlating the complete discovery sequence provides significantly higher confidence than alerting on individual commands.

---

# Response Recommendations

## Immediate Response

- Preserve auditd telemetry associated with the reverse shell.
- Continue monitoring child processes created by the compromised shell.
- Identify evidence of privilege escalation attempts.

## Further Investigation

- Review execution of SUID enumeration commands.
- Investigate searches for sensitive configuration files.
- Monitor access to authentication and credential stores.

## Long-Term Hardening

- Develop behavioural detections for Linux reconnaissance activity.
- Monitor application service accounts for shell execution.
- Alert on sequential execution of discovery commands from non-interactive sessions.

---

# Conclusion

Based on the available telemetry, I concluded that the attacker transitioned from initial compromise into a structured reconnaissance phase immediately after establishing the reverse shell.

The observed discovery commands were executed in a logical sequence designed to identify the compromised account, operating system, execution context, and accessible resources before progressing further into the intrusion. The collected evidence strongly suggests preparation for privilege escalation, which is examined in the next investigation.
