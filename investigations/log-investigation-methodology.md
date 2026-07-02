# Log Investigation Methodology

## Overview

This document outlines the structured investigation methodology I developed while studying log analysis fundamentals. Rather than relying solely on automated tools, the investigation process emphasizes analytical thinking, evidence correlation, and repeatable workflows that can be applied across different Security Information and Event Management (SIEM) platforms.

---

# Investigation Objectives

Every log investigation should answer the following questions:

- What happened?
- When did it happen?
- Who performed the action?
- Which systems were affected?
- How did the activity occur?
- Does the activity indicate malicious behavior?

---

# Investigation Workflow

```
Alert Generated
        │
        ▼
Validate Alert
        │
        ▼
Collect Relevant Logs
        │
        ▼
Normalize Timestamps
        │
        ▼
Build Event Timeline
        │
        ▼
Extract Indicators of Compromise (IOCs)
        │
        ▼
Correlate Multiple Log Sources
        │
        ▼
Threat Intelligence Validation
        │
        ▼
Determine Root Cause
        │
        ▼
Assess Impact
        │
        ▼
Document Findings
```

---

# Linux Command-Line Investigation Workflow

During this module I practiced using native Linux utilities instead of relying on a SIEM.

| Command | Investigation Purpose         |
| ------- | ----------------------------- |
| cat     | Display complete log file     |
| less    | Navigate large log files      |
| head    | View beginning of logs        |
| tail    | Review latest events          |
| tail -f | Monitor logs in real time     |
| wc      | Determine log size            |
| cut     | Extract specific fields       |
| sort    | Organize extracted data       |
| uniq    | Remove duplicates             |
| uniq -c | Count repeated values         |
| grep    | Search for indicators         |
| awk     | Filter using field conditions |
| sed     | Transform log output          |

---

# Investigation Example

Example investigation process:

1. Extract all source IP addresses.
2. Count occurrences.
3. Identify the highest-volume IP.
4. Review associated requests.
5. Build a timeline.
6. Validate indicators using threat intelligence.
7. Determine whether activity is benign or malicious.

---

# Detection Patterns Learned

## Authentication

- Multiple failed logins
- Impossible travel
- Logins outside normal hours

## Web Attacks

- SQL Injection
- Cross-Site Scripting (XSS)
- Directory Traversal

## Indicators of Compromise

- Malicious IP addresses
- Domains
- File hashes
- Suspicious user agents

---

# Detection Engineering

This module introduced vendor-neutral detection concepts including:

- Regular Expressions (Regex)
- Sigma Rules
- YARA Rules

These technologies allow analysts to create reusable detections that can later be integrated into SIEM platforms and malware analysis workflows.

---

# Lessons Learned

The biggest lesson from this module is that effective investigations depend more on analytical reasoning than on expensive tools.

A structured workflow, careful log correlation, and understanding attacker behavior are far more valuable than simply searching for keywords.

The command line remains one of the most effective investigation environments because it forces analysts to understand exactly what data they are extracting and why.
