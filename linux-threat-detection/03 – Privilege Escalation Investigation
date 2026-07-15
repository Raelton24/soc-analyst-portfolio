# 03 – Privilege Escalation Investigation

---

# Executive Summary

This investigation documents my analysis of the **Privilege Escalation** phase of the intrusion.

Following completion of host reconnaissance, I investigated evidence indicating that the attacker attempted to elevate privileges from the compromised application service account to the Linux root account.

Analysis of Linux auditd telemetry revealed execution of privilege escalation techniques immediately after system enumeration. By reconstructing process ancestry and comparing user contexts before and after the exploit execution, I confirmed that the attacker successfully transitioned from the compromised service account to **root**.

The investigation also identified subsequent access to sensitive files and credential stores, confirming that the privilege escalation was successful and enabled progression into the next phase of the attack.

---

# Incident Overview

The previous investigation established that the attacker had completed host reconnaissance after gaining an interactive reverse shell.

Based on the information collected during discovery, I investigated whether privilege escalation techniques were executed to obtain unrestricted access to the operating system.

The objectives of this investigation were to determine:

- Whether privilege escalation occurred.
- Which technique was used.
- Whether the escalation succeeded.
- What evidence demonstrated successful elevation.
- What attacker activity immediately followed root compromise.

---

# Data Sources

The following telemetry sources were used during the investigation.

| Data Source                        | Purpose                                         |
| ---------------------------------- | ----------------------------------------------- |
| Linux auditd                       | Process execution telemetry                     |
| Parent-Child Process Relationships | Privilege escalation reconstruction             |
| User Context (UID)                 | Verification of privilege transition            |
| Command-Line Arguments             | Identification of privilege escalation commands |

---

# Investigation Methodology

I began by reviewing all child processes created after the attacker completed system reconnaissance.

Rather than searching for a specific exploit, I analysed the execution sequence surrounding privilege escalation activity, reconstructed the associated process tree, and compared the effective user context before and after the suspicious process execution.

The investigation followed the workflow below.

```text
Discovery Completed

↓

Identify Suspicious Process

↓

Reconstruct Process Tree

↓

Compare User Context

↓

Confirm Privilege Escalation

↓

Review Post-Escalation Activity

↓

Identify Detection Opportunities
```

---

# Investigation Findings

## Discovery of Sensitive Files

Before attempting privilege escalation, the attacker searched the compromised system for sensitive information that could assist further compromise.

One of the observed commands searched recursively for files containing the keyword **pass**.

**Observed Command**

```bash
grep -iR pass .
```

The search identified an application configuration file containing sensitive credentials.

**Assessment**

The observed behaviour is consistent with credential discovery following initial host reconnaissance.

---

## Privilege Escalation Execution

Shortly after locating sensitive information, auditd telemetry recorded execution of the privilege escalation command.

**Observed Command**

```bash
su root
```

This represented the first direct attempt to transition from the compromised service account to the Linux root account.

Rather than assuming the escalation succeeded, I validated the result by reviewing the user context associated with subsequent child processes.

**Assessment**

Execution of the command alone does not confirm compromise. Additional evidence is required to verify successful privilege escalation.

---

## Verification of Root Access

To determine whether the privilege escalation succeeded, I compared the effective user associated with processes executed before and after the privilege escalation event.

Prior to escalation:

```text
UID = svctrypingme
```

Following execution of the privilege escalation command:

```text
UID = root
```

The change in user context confirmed that the attacker had successfully obtained root privileges.

This conclusion was supported by subsequent execution of commands operating under the root account.

**Assessment**

Comparison of effective user context provides high-confidence evidence confirming successful privilege escalation.

---

## Access to Sensitive Configuration Files

After obtaining root privileges, the attacker accessed application configuration files containing authentication material.

One of the observed commands accessed:

```bash
cat .env.local
```

Review of the configuration file revealed application credentials that would not normally be accessible to the compromised service account.

**Assessment**

Access to sensitive configuration files immediately after privilege escalation demonstrates that the attacker was leveraging elevated privileges to expand access to protected resources.

---

# Evidence Supporting Findings

The investigation findings were supported by:

- auditd process execution events
- User context (UID) comparison
- Parent-child process reconstruction
- Privilege escalation command execution
- Access to protected configuration files
- Process execution timestamps

---

# MITRE ATT&CK Mapping

| Tactic               | Technique                         | Technique ID |
| -------------------- | --------------------------------- | ------------ |
| Privilege Escalation | Abuse Elevation Control Mechanism | T1548        |
| Credential Access    | Credentials from Password Stores  | T1555        |
| Discovery            | File and Directory Discovery      | T1083        |

---

# Detection Engineering Opportunities

## High-Confidence Detections

- Detect execution of `su root` by non-administrative service accounts.
- Detect privilege escalation immediately following Linux discovery activity.
- Detect sudden changes in effective user context (UID).
- Detect access to protected configuration files immediately after privilege escalation.

## Correlation Opportunity

```text
Reverse Shell

↓

Discovery Commands

↓

grep -iR pass .

↓

su root

↓

UID Changes to Root

↓

Protected File Access

↓

Critical SOC Alert
```

Correlating privilege escalation with the surrounding reconnaissance activity significantly improves detection confidence while reducing false positives.

---

# Response Recommendations

## Immediate Response

- Terminate active root sessions.
- Preserve auditd telemetry associated with privilege escalation.
- Reset exposed privileged credentials.
- Isolate the compromised host.

## Further Investigation

- Review all commands executed under the root account.
- Identify files modified after privilege escalation.
- Determine whether persistence mechanisms were established.

## Long-Term Hardening

- Restrict unnecessary use of `su`.
- Monitor privileged account transitions.
- Alert on privilege escalation originating from application service accounts.
- Protect sensitive configuration files containing credentials.

---

# Conclusion

Based on the available telemetry, I concluded that the attacker successfully escalated privileges from the compromised application service account to the Linux root account.

The investigation confirmed the privilege transition through comparison of effective user contexts and subsequent execution of privileged commands. Once unrestricted access had been obtained, the attacker immediately began accessing sensitive configuration files and credentials, setting the stage for credential abuse, malware deployment, and persistence mechanisms examined in the next investigation.
