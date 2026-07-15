# 05 – Persistence Investigation

---

# Executive Summary

This investigation documents my analysis of the **Persistence** phase of the intrusion.

Following successful credential access and malware deployment, I investigated whether the attacker attempted to establish long-term access to the compromised Linux host. The objective was to determine whether access would survive process termination, user logout, or system reboot.

Analysis of Linux auditd telemetry and authentication logs identified multiple persistence mechanisms. The attacker created a privileged user account, modified SSH authentication by adding an authorized key, and established persistence through Linux startup mechanisms.

The evidence demonstrates that the attacker intended to retain long-term administrative access while reducing reliance on the original exploitation vector.

---

# Incident Overview

The previous investigation confirmed that the attacker had successfully obtained root privileges, accessed sensitive credentials, and deployed additional tooling.

At this stage of the intrusion, I shifted my investigation towards identifying mechanisms that would allow the attacker to return to the compromised host after the current session ended.

The objectives of this investigation were to determine:

- Whether new privileged accounts were created.
- Whether SSH authentication was modified.
- Whether startup persistence mechanisms were established.
- Whether the attacker attempted to survive system reboot.
- Which detection opportunities would identify similar persistence activity in production.

---

# Data Sources

The investigation relied on the following evidence sources.

| Data Source                               | Purpose                                           |
| ----------------------------------------- | ------------------------------------------------- |
| Linux auditd                              | Process execution and file modification telemetry |
| Authentication Logs (`/var/log/auth.log`) | User creation and privilege assignment            |
| File Integrity Events                     | SSH key persistence                               |
| Process Metadata                          | Reconstruction of persistence activity            |

---

# Investigation Methodology

Beginning with the final processes identified during malware deployment, I reviewed subsequent authentication events, privileged commands, and file modification activity associated with Linux persistence mechanisms.

Rather than searching for a single persistence technique, I investigated the three most common Linux persistence categories:

- Account persistence
- SSH key persistence
- Startup persistence

The investigation followed the workflow below.

```text
Root Access Confirmed

↓

Review Authentication Logs

↓

Review Privileged Account Activity

↓

Review SSH Key Modifications

↓

Review Startup Persistence

↓

Develop Detection Opportunities
```

---

# Investigation Findings

## Privileged Account Persistence

Authentication logs identified creation of a new local user account followed by assignment of administrative privileges.

**Observed Evidence**

```text
useradd

usermod
```

The authentication logs confirmed that the newly created account was immediately added to the **sudo** group.

This activity provided the attacker with an additional administrative access method independent of the original compromise.

**Assessment**

Creation of a privileged account shortly after compromise represents a high-confidence persistence mechanism.

---

## SSH Key Persistence

Further investigation identified modification of an SSH authorized keys file.

**Observed File**

```text
~/.ssh/authorized_keys
```

Rather than relying on passwords, the attacker added an SSH public key to the authorized keys file, allowing future authentication using the corresponding private key.

Unlike password authentication, malicious SSH keys often remain undetected because they resemble legitimate administrator keys.

**Assessment**

Modification of the authorized keys file established a persistent remote access mechanism capable of surviving password changes.

---

## Startup Persistence

Auditd telemetry identified persistence mechanisms designed to survive system reboot.

Observed activity included creation or modification of Linux startup components such as:

- Cron jobs
- Systemd services

Examples observed during the investigation included:

```text
crontab -e

systemctl

/etc/systemd/system/

/etc/cron.d/
```

These mechanisms ensure attacker-controlled processes automatically execute after reboot without requiring additional exploitation.

**Assessment**

Startup persistence demonstrates an intent to maintain long-term access and continue post-exploitation activity after system restart.

---

# Evidence Supporting Findings

The investigation findings were supported by:

- Authentication log entries
- auditd process execution telemetry
- User creation records
- Group membership modification
- SSH authorized key modifications
- Cron and systemd persistence activity
- File modification timestamps

---

# MITRE ATT&CK Mapping

| Tactic      | Technique             | Technique ID |
| ----------- | --------------------- | ------------ |
| Persistence | Create Account        | T1136        |
| Persistence | SSH Authorized Keys   | T1098.004    |
| Persistence | Scheduled Task / Cron | T1053.003    |
| Persistence | Systemd Service       | T1543.002    |

---

# Detection Engineering Opportunities

## High-Confidence Detections

- Detect execution of `useradd`.
- Detect execution of `usermod` assigning users to the **sudo** group.
- Detect modification of `~/.ssh/authorized_keys`.
- Detect creation or modification of cron jobs.
- Detect creation or modification of systemd service files.
- Detect execution of `systemctl` immediately following privilege escalation.

## Correlation Opportunity

```text
Privilege Escalation

↓

Create Privileged User

↓

Modify authorized_keys

↓

Create Cron Job

↓

Create Systemd Service

↓

Critical Persistence Alert
```

Correlating multiple persistence mechanisms provides significantly higher confidence than alerting on individual administrative commands.

---

# Response Recommendations

## Immediate Response

- Disable attacker-created user accounts.
- Remove unauthorized SSH public keys.
- Disable malicious cron jobs.
- Remove unauthorized systemd services.
- Rotate all privileged credentials.

## Further Investigation

- Review additional administrator accounts.
- Identify all modified authentication files.
- Review scheduled tasks created during the compromise.
- Verify integrity of Linux startup services.

## Long-Term Hardening

- Deploy file integrity monitoring for SSH authentication files.
- Monitor creation of privileged accounts.
- Alert on modifications to cron directories.
- Monitor creation of new systemd services.
- Require administrative approval for privileged account creation.

---

# Conclusion

Based on the available telemetry, I concluded that the attacker established multiple persistence mechanisms to retain administrative access to the compromised Linux host.

The investigation identified privileged account creation, SSH key persistence, and startup persistence through Linux scheduling and service mechanisms. Together, these techniques significantly increased the attacker's ability to survive password resets, user logoff, and system reboot.

With persistence successfully established, the attacker had completed the primary post-exploitation objectives of the intrusion. The final report consolidates the complete attack lifecycle, summarises the incident, and presents the overall assessment of the compromise.
