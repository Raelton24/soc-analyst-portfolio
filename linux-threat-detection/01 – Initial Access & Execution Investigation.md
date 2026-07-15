# 01 – Initial Access & Execution Investigation

---

# Executive Summary

This investigation documents my analysis of the **Initial Access and Execution** phase of a Linux intrusion involving a public-facing application.

Using Linux auditd telemetry, authentication logs, and process execution records, I reconstructed how the attacker transitioned from exploiting the vulnerable application to obtaining an interactive reverse shell on the compromised host.

The evidence shows that operating system commands were executed through the application before an outbound reverse shell was established using the `socat` utility. Process ancestry reconstruction confirmed that the activity originated from the compromised application rather than a legitimate SSH session.

This investigation establishes the initial attack timeline and provides the foundation for the remaining phases documented throughout this repository.

---

# Incident Overview

The investigation began after identifying suspicious shell-related process execution originating from the TryPingMe application.

Initial evidence indicated that the application had successfully executed operating system commands before spawning an interactive shell. Shortly afterwards, the `socat` utility established an outbound TCP connection, providing the attacker with remote command execution capabilities.

My objective during this investigation was to determine:

- How the attacker obtained operating system command execution.
- Whether the activity originated from SSH or application exploitation.
- How the reverse shell was established.
- Which evidence supported the reconstructed attack sequence.

---

# Data Sources

The following telemetry sources were used during the investigation.

| Data Source                               | Purpose                                                   |
| ----------------------------------------- | --------------------------------------------------------- |
| Linux auditd                              | Process execution and parent-child process reconstruction |
| Authentication Logs (`/var/log/auth.log`) | Validation of SSH authentication activity                 |
| Process Metadata                          | Process ancestry reconstruction                           |
| Command-Line Arguments                    | Verification of attacker execution                        |

---

# Investigation Methodology

To reconstruct the initial compromise, I began by identifying suspicious process execution before tracing the complete parent-child relationship responsible for spawning the reverse shell.

Rather than relying on isolated process executions, I correlated process identifiers, execution timestamps, authentication records, and command-line arguments to establish the complete execution chain.

The investigation followed the workflow below.

```text
Suspicious Process Detected

↓

Review Process Metadata

↓

Reconstruct Parent Process

↓

Review Authentication Logs

↓

Validate Initial Access Vector

↓

Reconstruct Attack Timeline

↓

Identify Detection Opportunities
```

---

# Investigation Findings

## Reverse Shell Execution

The earliest high-confidence indicator of compromise identified during the investigation was execution of the `socat` binary.

Using auditd process execution telemetry, I located every execution of the binary and reviewed the associated process metadata.

**Command Used**

```bash
ausearch -i -x socat
```

The returned telemetry identified:

- Process Identifier (PID)
- Parent Process Identifier (PPID)
- Executing account
- Command-line arguments
- Execution timestamp

Analysis of the command-line arguments confirmed that `socat` established an outbound TCP connection while spawning an interactive shell, providing the attacker with remote command execution.

**Assessment**

The observed behaviour is consistent with reverse shell establishment following successful exploitation of a public-facing application.

---

## Process Reconstruction

After identifying the reverse shell process, I reconstructed the complete parent-child relationship to determine how execution originated.

Using the parent process identifier obtained from the previous step, I traced execution back through the process tree.

**Command Used**

```bash
ausearch -i --pid <PPID>
```

The reconstructed execution chain was:

```text
TryPingMe Application

↓

Python Interpreter

↓

/bin/sh

↓

socat

↓

Interactive Reverse Shell
```

The reconstructed process ancestry demonstrated that the reverse shell originated from the compromised application rather than an interactive terminal session.

**Assessment**

Process ancestry provided high-confidence evidence linking application exploitation directly to reverse shell establishment.

---

## Authentication Review

To determine whether the attacker authenticated through SSH before obtaining command execution, I reviewed the available authentication logs.

Primary evidence source:

```text
/var/log/auth.log
```

No successful SSH authentication events were identified immediately before execution of the malicious processes.

**Assessment**

Based on the available telemetry, exploitation of the public-facing application represents the most likely initial access vector.

---

# Evidence Supporting Findings

The investigation findings were supported by the following evidence:

- auditd process execution events
- Parent-child process relationships
- Reverse shell command-line arguments
- Authentication log review
- Process execution timestamps

---

# MITRE ATT&CK Mapping

| Tactic              | Technique                                     | Technique ID |
| ------------------- | --------------------------------------------- | ------------ |
| Initial Access      | Exploit Public-Facing Application             | T1190        |
| Execution           | Command and Scripting Interpreter: Unix Shell | T1059.004    |
| Command and Control | Non-Application Layer Protocol                | T1095        |

---

# Detection Engineering Opportunities

## High-Confidence Detections

- Detect public-facing applications spawning `/bin/sh`.
- Detect execution of `socat` by application service accounts.
- Detect outbound reverse shell activity immediately following shell execution.
- Detect abnormal parent-child relationships linking web services to interactive shells.

## Correlation Opportunity

```text
Public-Facing Application

↓

Shell Spawned

↓

Outbound TCP Connection

↓

Interactive Reverse Shell

↓

High-Severity SOC Alert
```

Behavioural correlation significantly increases confidence while reducing false positives associated with individual process executions.

---

# Response Recommendations

## Immediate Response

- Isolate the affected host from the network.
- Terminate active reverse shell processes.
- Preserve volatile evidence before remediation.
- Block outbound communication to attacker-controlled infrastructure.

## Further Investigation

- Review all child processes originating from the compromised application.
- Identify post-exploitation activity performed through the reverse shell.
- Determine whether privilege escalation was attempted.

## Long-Term Hardening

- Patch the vulnerable application.
- Restrict unnecessary shell execution by application services.
- Deploy behavioural detections for abnormal parent-child process relationships.
- Monitor outbound network connections initiated by application service accounts.

---

# Conclusion

Based on the available evidence, I concluded that the attacker successfully exploited a public-facing Linux application to obtain operating system command execution before establishing an outbound reverse shell using `socat`.

Process ancestry reconstruction confirmed that the activity originated from the compromised application rather than a legitimate SSH session. Having established interactive access, the attacker immediately transitioned into host reconnaissance, which forms the focus of the next investigation.
