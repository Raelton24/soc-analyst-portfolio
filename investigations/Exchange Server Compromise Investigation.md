# Exchange Server Compromise Investigation Using Elastic SIEM

## Executive Summary

A security investigation was initiated following multiple high-severity alerts indicating suspicious web activity targeting an internet-facing Microsoft Exchange server. Correlation across IIS web logs, Windows Security events, Sysmon telemetry, and PowerShell logging confirmed a successful server compromise originating from the external IP address **203.0.113.55**.

The attacker exploited the Exchange web application to deploy a web shell, executed reconnaissance commands, authenticated using the local Administrator account, established persistence through a scheduled task and a newly created service account, elevated privileges, and staged sensitive business data for collection using WinRAR.

The investigation concluded with high confidence that the observed activity represented a full post-exploitation intrusion rather than isolated suspicious events.

---

# Investigation Scope

**Objective**

Determine whether multiple security alerts represented legitimate administrative activity or an active compromise.

**Primary Log Sources**

- IIS Web Server Logs
- Windows Security Logs
- Sysmon Operational Logs
- PowerShell Script Block Logging

**Affected Host**

```
winserv2019.some.corp
```

---

# Alert Overview

The investigation began after multiple alerts identified suspicious activity involving a single external source.

| Time (UTC) | Alert                                               |
| ---------- | --------------------------------------------------- |
| 04:38      | Web Requests Indicating File Upload                 |
| 04:45      | GET Requests to ASPX File with Query Parameters     |
| 05:11      | Administrator Authentication Outside Business Hours |
| 05:13      | New User Account Created                            |
| 05:13      | Suspicious Command-Line Activity                    |

Rather than investigating each alert independently, related events were correlated to determine whether they formed a single intrusion.

---

# Investigation Findings

## Phase 1 — Initial Access

Analysis of IIS web server logs identified repeated POST requests to the Exchange endpoint:

```
/proxyLogon.ecp
```

originating from

```
203.0.113.55
```

The requests were performed using an automated client rather than a browser.

Evidence indicated an attempted exploitation of Microsoft Exchange through the ProxyLogon attack chain.

---

## Phase 2 — Web Shell Activity

Shortly after the exploitation attempt, HTTP GET requests targeted:

```
/errorEE.aspx
```

using the parameter

```
cmd=
```

Multiple successful HTTP 200 responses confirmed remote command execution through the deployed web shell.

Observed commands included:

- whoami
- hostname
- ipconfig /all
- netstat
- net user
- net localgroup administrators
- net group "Domain Admins"
- net share
- schtasks
- sc query
- Get-Process lsass

This sequence demonstrated systematic host and network reconnaissance.

---

## Phase 3 — Persistence

The attacker established persistence by creating the scheduled task:

```
WinUpdate
```

configured to execute every five minutes.

The task periodically issued outbound HTTP requests to attacker-controlled infrastructure, providing a persistent communication channel.

---

## Phase 4 — Credential Access

Subsequent web shell activity executed:

```
rundll32.exe comsvcs.dll MiniDump
```

against the LSASS process.

The resulting memory dump was written to:

```
C:\Windows\Temp\lsass.dmp
```

This activity strongly indicates an attempt to obtain cached credentials from system memory.

---

## Phase 5 — Administrator Authentication

Windows Security Event ID **4624** confirmed a successful Administrator logon immediately following the credential access activity.

The source IP matched the address previously observed exploiting the web application:

```
203.0.113.55
```

This correlation linked the initial compromise directly to privileged interactive access.

---

## Phase 6 — Backdoor Account Creation

Windows Account Management events confirmed creation of the account:

```
svc_backup
```

The naming convention closely resembled a legitimate service account, reducing the likelihood of immediate administrative detection.

Subsequent Security Group Management events showed the account being added to multiple privileged groups, including the local Administrators group and Remote Desktop Users.

This activity established an additional persistence mechanism independent of the original web shell.

---

## Phase 7 — Post-Compromise Activity

Sysmon process creation events showed the Administrator account launching command prompt sessions to modify local group memberships.

PowerShell Script Block Logging further confirmed continued reconnaissance and privilege validation after persistence had been established.

The combined telemetry demonstrated that the attacker remained actively engaged on the compromised host following successful authentication.

---

## Phase 8 — Data Collection

No alert was generated for the final activity.

Sysmon telemetry identified execution of:

```
C:\Program Files\WinRAR\Rar.exe
```

by the compromised account:

```
svc_backup
```

The archive created was:

```
finance_it_archive.rar
```

The archive contained data collected from:

```
C:\Users\asmith\Documents\
```

and

```
C:\IT\Admin\Scripts\
```

The archive was password protected before being written to:

```
C:\Temp\
```

Although WinRAR is legitimate software, its execution immediately following account creation, privilege modification, and credential access strongly indicates data staging prior to exfiltration.

---

# Attack Timeline

| Time  | Activity                                |
| ----- | --------------------------------------- |
| 04:38 | ProxyLogon exploitation attempt         |
| 04:45 | Web shell command execution             |
| 04:48 | Scheduled task created for persistence  |
| 05:05 | LSASS memory dump performed             |
| 05:11 | Administrator authenticated             |
| 05:13 | `svc_backup` account created            |
| 05:13 | Privileged group memberships modified   |
| 05:16 | Additional PowerShell activity observed |
| 05:17 | Sensitive business data archived        |

---

# Indicators of Compromise

## IP Address

```
203.0.113.55
```

## Web Shell

```
errorEE.aspx
```

## Exploited Endpoint

```
proxyLogon.ecp
```

## Scheduled Task

```
WinUpdate
```

## Malicious User

```
svc_backup
```

## Archive

```
finance_it_archive.rar
```

## Credential Dump

```
lsass.dmp
```

---

# MITRE ATT&CK Mapping

| Tactic               | Technique                                 |
| -------------------- | ----------------------------------------- |
| Initial Access       | Exploit Public-Facing Application (T1190) |
| Execution            | Command and Scripting Interpreter (T1059) |
| Persistence          | Scheduled Task (T1053.005)                |
| Persistence          | Create Account (T1136)                    |
| Privilege Escalation | Account Manipulation (T1098)              |
| Credential Access    | OS Credential Dumping (T1003.001)         |
| Discovery            | System, Account, and Network Discovery    |
| Collection           | Data from Local System (T1005)            |
| Collection           | Archive Collected Data (T1560.001)        |

---

# Analyst Assessment

Correlation across web server logs, Windows Security events, Sysmon telemetry, and PowerShell logging confirmed a complete post-exploitation attack lifecycle. The evidence demonstrates successful exploitation of the Exchange server, establishment of multiple persistence mechanisms, credential theft, privileged account abuse, and preparation of sensitive organizational data for exfiltration.

No single alert was sufficient to describe the incident. Confidence in the assessment was achieved by correlating independent telemetry sources into a unified attack timeline, allowing each stage of the intrusion to be validated with supporting evidence.

---

# Recommendations

- Immediately isolate the affected Exchange server from the network.
- Remove the `svc_backup` account and review all recently created or modified privileged accounts.
- Delete unauthorized scheduled tasks and investigate additional persistence mechanisms.
- Reset credentials for privileged accounts, particularly those that may have been exposed through the LSASS memory dump.
- Review outbound network activity for evidence of data exfiltration.
- Patch and validate the Exchange environment against known vulnerabilities.
- Conduct enterprise-wide threat hunting for the identified indicators of compromise and related attacker activity.
