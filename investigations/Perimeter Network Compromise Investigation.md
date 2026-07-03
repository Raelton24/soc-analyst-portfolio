# Perimeter Compromise Investigation

# Executive Summary

A month-long review of firewall, VPN authentication, and IDS telemetry identified a successful intrusion originating from an external attacker. The investigation reconstructed the complete attack lifecycle, beginning with reconnaissance against exposed perimeter services, progressing through VPN credential compromise, internal lateral movement, command-and-control communication, and attempted data exfiltration.

Multiple log sources were correlated to validate each phase of the intrusion, demonstrating how network telemetry can be used to reconstruct attacker activity and support incident response.

## Incident Overview

A review of perimeter security logs identified suspicious activity originating from an external IP address targeting multiple publicly exposed services. Further investigation revealed a successful compromise of the organization's VPN gateway, followed by lateral movement within the internal network, persistent command-and-control (C2) communications, and indicators of data exfiltration.

The objective of this investigation was to reconstruct the attack timeline using multiple log sources, identify the affected assets, determine the attack techniques used, and document actionable findings.

---

# Environment

## Network Assets

| Host                 | IP Address | Role                          | Criticality |
| -------------------- | ---------- | ----------------------------- | ----------- |
| VPN Gateway          | 10.0.0.50  | Remote Access Gateway         | Critical    |
| Finance Server       | 10.0.0.20  | SMB/File Server               | High        |
| Internal Web Server  | 10.0.0.51  | Application Server            | High        |
| Employee Workstation | 10.0.0.60  | Windows Workstation           | Medium      |
| VPN Client           | 10.8.0.x   | Assigned Internal VPN Address | Critical    |

---

# Investigation Objective

Determine:

- Which external host initiated reconnaissance.
- Which internal systems were targeted.
- Whether VPN authentication was compromised.
- Whether lateral movement occurred.
- Whether persistent C2 communication existed.
- Whether data exfiltration occurred.
- The complete attack timeline.

---

# Investigation Workflow

Firewall Logs
│
▼
Reconnaissance

      │
      ▼

VPN Authentication

      │
      ▼

Successful VPN Login

      │
      ▼

Assigned Internal IP

      │
      ▼

Firewall Correlation

      │
      ▼

IDS Correlation

      │
      ▼

Lateral Movement

      │
      ▼

C2 Beaconing

      │
      ▼

Data Exfiltration

      │
      ▼

Incident Timeline

# Initial Indicators

The investigation began after perimeter monitoring detected repeated blocked firewall connections targeting multiple services.

Additional indicators included:

- Excessive VPN authentication failures.
- IDS alerts for SMB exploitation.
- Repeated outbound beaconing.
- Large outbound HTTP POST requests.

These events suggested a coordinated intrusion rather than isolated malicious activity.

---

# Investigation Process

## Phase 1 — Reconnaissance

Firewall logs were reviewed to identify blocked inbound traffic.

The analysis focused on identifying:

- repeated source IPs
- sequential port scanning
- multiple destination services

### Commands Used

```bash
grep "BLOCK" firewall.log

grep "BLOCK" firewall.log | \
cut -d' ' -f5 | \
cut -d: -f1 | \
sort | uniq -c | sort -nr
```

### Why

These commands isolate blocked traffic, extract source IP addresses, count occurrences, and identify which external host generated the largest number of scanning attempts.

### Findings

- One external IP generated significantly more blocked events than all others.
- Multiple ports were probed against critical infrastructure.
- The VPN gateway was the primary target.

---

## Phase 2 — Target Identification

After identifying the attacker, the next objective was determining which internal assets received the highest number of probes.

### Commands

```bash
grep "<attacker-ip>" firewall.log | \
cut -d' ' -f7 | \
cut -d: -f1 | \
sort | uniq -c | sort -nr
```

### Findings

The attacker focused primarily on:

- VPN Gateway
- Finance Server
- Internal Workstation
- Application Server

This suggested deliberate service enumeration rather than random scanning.

---

## Phase 3 — VPN Credential Attack

VPN authentication logs were reviewed for abnormal authentication activity.

### Commands

```bash
grep FAIL vpn_auth.log

grep FAIL vpn_auth.log | \
cut -d' ' -f3,4 | \
sort | uniq -c | sort -nr

grep "<username>" vpn_auth.log
```

### Findings

Evidence showed:

- repeated authentication failures
- password guessing against a service account
- eventual successful authentication

The successful login assigned an internal VPN address, confirming compromise of valid credentials.

---

## Phase 4 — Initial Access

After successful VPN authentication, the assigned internal VPN address became the primary pivot point for the remainder of the investigation.

### Commands

```bash
grep SUCCESS vpn_auth.log
```

### Findings

The attacker successfully obtained internal network access through the VPN gateway.

---

## Phase 5 — Lateral Movement

Firewall and IDS logs were correlated to identify movement between internal hosts.

### Commands

```bash
grep "SMB" ids_alerts.log

grep "SMB" ids_alerts.log | head
```

### Findings

IDS alerts identified:

- SMB exploitation
- SSH scanning
- RDP brute-force attempts

The attacker attempted to compromise additional internal systems after obtaining VPN access.

---

## Phase 6 — Command and Control

The IDS generated multiple C2 beaconing alerts.

### Commands

```bash
grep "C2" ids_alerts.log

grep "C2" ids_alerts.log | \
cut -d' ' -f19 | \
cut -d: -f1 | \
sort | uniq -c | sort -nr
```

### Findings

One internal host repeatedly communicated with the same external IP.

Characteristics included:

- periodic outbound traffic
- consistent destination
- repeated connections
- persistence over time

These are common indicators of malware beaconing.

---

## Phase 7 — Data Exfiltration

IDS alerts were reviewed for outbound upload activity.

### Commands

```bash
grep "Large Upload" ids_alerts.log

grep "Potential Data Exfiltration" ids_alerts.log
```

### Findings

The compromised host generated repeated large outbound HTTP POST requests.

Combined with persistent C2 traffic, this strongly suggested attempted data exfiltration.

---

# Evidence Collected

## Firewall

- Blocked reconnaissance
- Allowed VPN access
- Internal lateral movement
- Outbound communications

## VPN

- Repeated failed authentication
- Successful login
- Internal VPN assignment

## IDS

- SMB exploitation
- SSH scanning
- RDP brute force
- C2 beaconing
- Data exfiltration alerts

---

# Attack Timeline

| Phase             | Description                                  |
| ----------------- | -------------------------------------------- |
| Reconnaissance    | External attacker scanned exposed services   |
| Credential Access | VPN password guessing                        |
| Initial Access    | Successful VPN authentication                |
| Persistence       | Internal VPN address assigned                |
| Discovery         | Internal service enumeration                 |
| Lateral Movement  | SMB / SSH / RDP activity                     |
| Command & Control | Regular beaconing to external infrastructure |
| Exfiltration      | Large outbound HTTP POST uploads             |

---

# MITRE ATT&CK Mapping

| Tactic              | Technique                             |
| ------------------- | ------------------------------------- |
| Reconnaissance      | Active Scanning (T1595)               |
| Credential Access   | Brute Force (T1110)                   |
| Initial Access      | Valid Accounts (T1078)                |
| Discovery           | Network Service Discovery (T1046)     |
| Lateral Movement    | SMB/Windows Admin Shares (T1021.002)  |
| Lateral Movement    | Remote Services (T1021)               |
| Command and Control | Application Layer Protocol (T1071)    |
| Exfiltration        | Exfiltration Over Web Service (T1567) |

---

# Indicators of Compromise (IOCs)

## External

- Reconnaissance IP
- VPN brute-force IP
- Command-and-Control IP

## Internal

- Compromised VPN client
- Assigned VPN address
- Lateral movement source host

---

# Containment Recommendations

Immediate actions:

- Block malicious external IPs.
- Disable compromised VPN accounts.
- Reset affected credentials.
- Isolate compromised hosts.
- Terminate active VPN sessions.
- Block outbound C2 communications.
- Investigate systems accessed through SMB and RDP.
- Review firewall exposure for unnecessary services.

---

# Detection Opportunities

Improve detections for:

- High-volume port scanning.
- VPN authentication failures.
- Impossible travel logins.
- Multiple VPN logins from a single source.
- SMB lateral movement.
- RDP brute-force attempts.
- Periodic outbound beaconing.
- Large outbound HTTP POST requests.

---

# Detection Logic

The investigation relied on behavioral correlation rather than isolated alerts.

Detection logic included:

- Multiple blocked connections from a single external IP
- Sequential probing across multiple ports
- High-volume VPN authentication failures
- Successful authentication following repeated failures
- New internal VPN IP assignment
- SMB, SSH and RDP activity originating from the assigned VPN client
- Regular outbound communications to a single external IP
- Large outbound HTTP POST uploads

No single alert confirmed compromise. Confidence increased as evidence accumulated across multiple telemetry sources.

# Lessons Learned

This investigation reinforced the importance of correlating multiple telemetry sources rather than relying on a single log type.

Firewall logs identified reconnaissance.

VPN logs confirmed credential compromise.

IDS alerts revealed attacker behavior after initial access.

By pivoting between these datasets, it was possible to reconstruct the complete intrusion lifecycle from reconnaissance through attempted data exfiltration.

---

# Linux Commands Used

| Purpose                    | Command                     |
| -------------------------- | --------------------------- |
| Filter blocked traffic     | `grep "BLOCK" firewall.log` |
| Count source IPs           | `cut`, `sort`, `uniq -c`    |
| Review VPN failures        | `grep FAIL vpn_auth.log`    |
| Find successful VPN logins | `grep SUCCESS vpn_auth.log` |
| Review SMB exploitation    | `grep "SMB" ids_alerts.log` |
| Identify C2                | `grep "C2" ids_alerts.log`  |
| Detect data exfiltration   | `grep "Large Upload"`       |

---

# Skills Demonstrated

- Firewall log analysis
- VPN authentication analysis
- IDS alert investigation
- IOC identification
- Network traffic correlation
- Incident timeline reconstruction
- MITRE ATT&CK mapping
- Linux command-line log analysis
- Threat hunting methodology
- Evidence-based investigation

# Analyst Reflection

This investigation demonstrated how seemingly unrelated security events become meaningful when correlated across multiple log sources.

Rather than relying on a single alert, the investigation followed the attacker's progression through each phase of the intrusion, validating every finding with supporting evidence from firewall, VPN, and IDS telemetry.

The investigation also reinforced a key SOC principle:

> Every alert is a clue, but only correlation reveals the full attack story.

# Tools Used

- Linux
- Bash
- grep
- cut
- uniq
- sort
- head
- tail
- Splunk (optional workflow)
- Firewall Logs
- IDS Logs
- VPN Authentication Logs
