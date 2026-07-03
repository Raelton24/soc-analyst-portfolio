# Network Discovery Detection Investigation

## Executive Summary

This investigation analyzes multiple forms of network reconnaissance observed within enterprise firewall and Zeek connection logs. The objective was to determine whether observed scanning activity represented benign network administration or malicious reconnaissance preceding an intrusion attempt.

During the investigation, evidence of external reconnaissance, internal network discovery, horizontal scanning, vertical scanning, ICMP host discovery, and TCP SYN scanning was identified and analyzed. Each activity was correlated to attacker objectives using network telemetry and mapped to the MITRE ATT&CK framework.

The investigation focuses on analyst reasoning, evidence correlation, and defensive decision-making rather than simply identifying scan types.

---

# Incident Overview

Network discovery is typically one of the earliest observable stages of an intrusion.

Before exploitation occurs, attackers attempt to answer several critical questions:

- Which hosts are online?
- Which services are exposed?
- Which ports are open?
- Which systems are likely vulnerable?

The collected firewall and Zeek logs contained multiple reconnaissance techniques that demonstrated how attackers enumerate enterprise environments before attempting exploitation.

---

# Environment

## Log Sources

- Firewall Logs
- Zeek Connection Logs

## Operating Environment

Linux-based investigation workstation

## Analysis Utilities

- grep
- head
- less
- cut
- wc

---

# Investigation Objective

The investigation sought to determine:

- Whether scanning originated internally or externally.
- Whether activity represented horizontal or vertical scanning.
- Which scanning techniques were used.
- Whether activity indicated early reconnaissance or post-compromise discovery.
- Appropriate SOC response priorities.

---

# Initial Indicators

Initial review identified:

- Multiple failed TCP connection attempts
- Repeated destination port probing
- Multiple destination hosts contacted within short time intervals
- ICMP host discovery activity
- Zeek connection states consistent with SYN scanning

These behaviors suggested systematic reconnaissance rather than legitimate user activity.

---

# Investigation Workflow

1. Review exported firewall logs.
2. Identify source and destination IP addresses.
3. Differentiate public and private addressing.
4. Classify scanning as internal or external.
5. Analyze destination port behavior.
6. Distinguish horizontal and vertical scans.
7. Review Zeek `conn_state`.
8. Identify scan mechanics.
9. Map activity to MITRE ATT&CK.
10. Recommend containment and detection improvements.

---

# Investigation Process

## Phase 1 – External Reconnaissance

### Objective

Determine whether public-facing assets were being enumerated.

### Evidence

External IP:

```
203.0.113.25
```

Target:

```
192.168.230.145
```

### Analyst Reasoning

The attacker remained outside the organization and attempted to enumerate exposed services on perimeter assets.

This activity aligns with the Reconnaissance phase of the attack lifecycle.

---

## Phase 2 – Internal Discovery

### Objective

Determine whether scanning originated inside the network.

### Evidence

Source:

```
192.168.230.127
```

Destination:

```
192.168.230.x
```

### Analyst Reasoning

Private-to-private communication performing systematic scanning strongly suggests an attacker has already established a foothold within the environment.

At this stage, investigation priority increases significantly because the adversary may be preparing for lateral movement.

---

## Phase 3 – Horizontal Scan Detection

### Evidence Pattern

- Same Source IP
- Same Destination Port
- Multiple Destination Hosts

### Analyst Assessment

The attacker searched the network for systems exposing a specific service.

This behavior is commonly associated with exploitation campaigns targeting known vulnerabilities.

---

## Phase 4 – Vertical Scan Detection

### Evidence Pattern

- Same Source IP
- Same Destination Host
- Multiple Destination Ports

### Analyst Assessment

The attacker profiled an individual system to identify all available services before selecting an attack path.

---

## Phase 5 – Scan Mechanics

### ICMP Ping Sweep

Purpose:

Identify live hosts across the subnet.

Detection:

Large numbers of ICMP Echo Requests sent sequentially.

---

### TCP SYN Scan

Evidence:

Zeek connection state:

```
S0
```

Analyst Interpretation:

SYN packets were transmitted without completing the TCP handshake.

This behavior is consistent with TCP SYN scanning.

---

### UDP Scan

The investigation searched for UDP reconnaissance.

No evidence of UDP scanning was identified.

---

# Evidence Collected

| Evidence        | Finding         |
| --------------- | --------------- |
| External Source | 203.0.113.25    |
| Internal Source | 192.168.230.127 |
| Ping Sweep      | Confirmed       |
| TCP SYN Scan    | Confirmed       |
| UDP Scan        | Not Observed    |
| Horizontal Scan | Confirmed       |
| Vertical Scan   | Confirmed       |

---

# Attack Timeline

1. External reconnaissance begins.
2. Public-facing services enumerated.
3. Internal host performs subnet discovery.
4. Horizontal scanning identifies systems exposing target services.
5. Vertical scanning profiles high-value hosts.
6. SYN scanning confirms open ports without completing TCP sessions.

---

# MITRE ATT&CK Mapping

| Technique                 | ATT&CK ID |
| ------------------------- | --------- |
| Active Scanning           | T1595     |
| Network Service Discovery | T1046     |

---

# Indicators of Compromise

- Repeated SYN packets
- Numerous failed connection attempts
- Sequential host enumeration
- High volume of port probing
- Repeated destination changes
- Incomplete TCP handshakes
- Unexpected internal scanning

---

# Containment Recommendations

Immediate:

- Validate whether scanning source is authorized.
- Isolate suspicious internal hosts.
- Review EDR telemetry.
- Block confirmed malicious external sources where appropriate.

Long-Term:

- Reduce exposed attack surface.
- Restrict unnecessary services.
- Patch vulnerable systems.
- Implement network segmentation.
- Maintain an inventory of authorized vulnerability scanners.
- Tune SIEM detection rules to suppress expected scanning activity.

---

# Detection Opportunities

Potential detections include:

- ICMP sweep detection
- Horizontal scan detection
- Vertical scan detection
- TCP SYN scan detection
- Internal reconnaissance detection
- Excessive failed connection attempts
- Scanning outside approved maintenance windows

---

# Detection Logic

### Horizontal Scan

```
One Source IP

↓

Many Destination Hosts

↓

Same Destination Port
```

---

### Vertical Scan

```
One Source IP

↓

One Destination Host

↓

Many Destination Ports
```

---

### SYN Scan

```
Repeated SYN packets

AND

No completed handshake

THEN

Alert
```

---

# Skills Demonstrated

- Network log analysis
- Firewall investigation
- Zeek analysis
- Threat hunting
- Pattern recognition
- Network protocol analysis
- MITRE ATT&CK mapping
- Incident triage
- Linux log analysis
- SOC investigation methodology

---

# Tools Used

- Zeek
- Kibana
- Linux CLI
- Firewall Logs

---

# Investigation Commands Reference

```bash
head log-session-2.csv

head -n2 log-session-1.csv

grep "192.168.230.127" log-session-2.csv | wc -l

grep "203.0.113.25" log-session-1.csv

less log-session-1.csv

cut -d',' -f2,4,5 log-session-2.csv
```

---

# Lessons Learned

This investigation reinforced that network discovery is rarely identified from a single log entry. Effective detection relies on correlating repeated behaviors across multiple events and understanding attacker objectives rather than focusing solely on individual packets.

Understanding the relationship between scan patterns, protocol behavior, and connection states enables analysts to identify reconnaissance early in the intrusion lifecycle and respond before exploitation occurs.

---

# Analyst Reflection

One of the most valuable lessons from this investigation was recognizing that context determines severity.

An external scan often represents early reconnaissance and can usually be mitigated at the perimeter. Internal scanning, however, indicates that an attacker may already possess an initial foothold and is actively mapping the environment in preparation for lateral movement.

Throughout the investigation, analyst decisions were driven by evidence correlation rather than assumptions. By combining firewall logs, Zeek connection states, network addressing, and scan mechanics, it was possible to reconstruct the attacker's objectives and recommend practical defensive improvements.

This investigation demonstrates a repeatable SOC workflow for analyzing network discovery activity and translating raw network telemetry into actionable security findings.
