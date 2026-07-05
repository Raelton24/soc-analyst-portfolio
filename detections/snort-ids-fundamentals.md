# Snort IDS Fundamentals: Rule Development & PCAP Analysis

## Executive Summary

Intrusion Detection Systems (IDS) play a critical role in network defense by providing visibility into malicious or suspicious activity that bypasses preventive security controls such as firewalls. Unlike firewalls, IDS solutions do not block traffic; instead, they continuously inspect network communications and generate alerts when traffic matches predefined detection logic.

This project demonstrates the fundamentals of working with **Snort**, one of the most widely used open-source Network Intrusion Detection Systems (NIDS). The exercises cover Snort architecture, operating modes, rule development, live traffic monitoring, and offline analysis of packet capture (PCAP) files.

The repository also includes practical detection engineering concepts, showing how custom rules can be developed to identify specific network activity and how historical packet captures can be analyzed during forensic investigations.

---

# Learning Objectives

After completing this project I can:

- Explain the purpose of an Intrusion Detection System (IDS)
- Differentiate IDS from traditional firewall technology
- Understand signature-based and anomaly-based detection
- Navigate the Snort directory structure
- Develop custom Snort detection rules
- Understand the components of Snort rule syntax
- Execute Snort against live network traffic
- Perform offline PCAP analysis using Snort
- Interpret generated IDS alerts

---

# Firewall vs IDS

| Firewall                         | IDS                                 |
| -------------------------------- | ----------------------------------- |
| Prevents unauthorized traffic    | Detects suspicious traffic          |
| Blocks connections               | Generates alerts                    |
| Operates at the network boundary | Monitors traffic within the network |
| Preventive security control      | Detective security control          |

A firewall decides whether traffic is permitted into a network.

An IDS assumes traffic may already be inside the network and continuously monitors it for signs of malicious activity.

---

# Understanding Snort

Snort is an open-source Network Intrusion Detection System (NIDS) capable of inspecting network traffic in real time or from previously captured packet captures.

Its primary capabilities include:

- Signature-based detection
- Anomaly detection
- Custom rule creation
- Network traffic monitoring
- Packet capture analysis
- Alert generation

---

# Snort Operating Modes

## Packet Sniffer Mode

Displays network packets without performing detection.

Typical use cases include:

- Network troubleshooting
- Traffic observation
- Protocol analysis

---

## Packet Logging Mode

Captures traffic into PCAP files for later analysis.

Useful during:

- Incident response
- Digital forensics
- Historical investigations

---

## Network Intrusion Detection System (NIDS) Mode

The primary operating mode.

Responsibilities include:

- Monitoring network traffic
- Comparing packets against rule sets
- Detecting malicious activity
- Generating security alerts

---

# Snort Directory Structure

Important files used throughout the project:

| File                           | Purpose                           |
| ------------------------------ | --------------------------------- |
| `/etc/snort/snort.lua`         | Main configuration                |
| `/etc/snort/rules/local.rules` | Custom detection rules            |
| `/etc/snort/rules/`            | Built-in rules                    |
| `/etc/snort/`                  | Main Snort installation directory |

---

# Understanding Snort Rules

A Snort rule consists of two major sections:

- Rule Header
- Rule Options

Example:

```snort
alert icmp any any -> 127.0.0.1 any (msg:"Loopback Ping Detected"; sid:10003; rev:1;)
```

### Rule Breakdown

| Component | Description          |
| --------- | -------------------- |
| alert     | Action               |
| icmp      | Protocol             |
| any       | Source IP            |
| any       | Source Port          |
| ->        | Traffic Direction    |
| 127.0.0.1 | Destination IP       |
| any       | Destination Port     |
| msg       | Alert Message        |
| sid       | Signature Identifier |
| rev       | Rule Revision        |

---

# Custom Rule Development

A custom detection rule was created to identify ICMP packets destined for the loopback interface.

Rule:

```snort
alert icmp any any -> 127.0.0.1 any (msg:"Loopback Ping Detected"; sid:10003; rev:1;)
```

Testing was performed by generating ICMP traffic using:

```bash
ping 127.0.0.1
```

Successful execution generated the expected Snort alert, confirming that the custom detection logic functioned correctly.

---

# Live Traffic Detection

Snort was executed in Network Intrusion Detection System mode using:

```bash
sudo snort -q -l /var/log/snort -i lo -A alert_fast -c /etc/snort/snort.lua
```

The IDS successfully inspected live traffic and generated alerts when packets matched the configured detection rules.

---

# Offline PCAP Analysis

A historical packet capture (`Intro_to_IDS.pcap`) was analyzed using Snort's offline analysis capabilities.

Command:

```bash
sudo snort -q -l /var/log/snort -r Intro_to_IDS.pcap -A alert_fast -c /etc/snort/snort.lua
```

This demonstrates how Snort can be incorporated into forensic investigations by applying detection logic to previously captured network traffic.

---

# Exercise Findings

Analysis of the supplied packet capture identified multiple IDS alerts.

## Detection Results

| Finding              | Value          |
| -------------------- | -------------- |
| SSH Source IP        | `10.11.90.211` |
| SSH Rule SID         | `1000002`      |
| Additional Detection | Ping Detected  |

The exercise demonstrates how multiple detection rules may trigger against the same packet capture, providing analysts with valuable indicators for further investigation.

---

# Detection Workflow

```
Network Traffic
       │
       ▼
Snort Engine
       │
       ▼
Rule Matching
       │
       ▼
Alert Generation
       │
       ▼
SOC Analyst Investigation
```

---

# Skills Demonstrated

- Intrusion Detection Systems (IDS)
- Network Security Monitoring
- Detection Engineering
- Snort Configuration
- Custom Rule Development
- Signature-Based Detection
- PCAP Analysis
- Linux Command Line
- Network Traffic Inspection
- Security Alert Interpretation

---

# Commands Reference

List Snort files:

```bash
ls /etc/snort
```

Edit custom rules:

```bash
sudo nano /etc/snort/rules/local.rules
```

Run Snort:

```bash
sudo snort -q -l /var/log/snort -i lo -A alert_fast -c /etc/snort/snort.lua
```

Analyze a PCAP:

```bash
sudo snort -q -l /var/log/snort -r Intro_to_IDS.pcap -A alert_fast -c /etc/snort/snort.lua
```

Generate ICMP traffic:

```bash
ping 127.0.0.1
```

---

# Lessons Learned

This project reinforced several important concepts in network security.

- Firewalls and IDS solutions complement one another rather than perform the same function.
- Detection engineering is heavily dependent on well-written rules.
- Signature IDs and rule revisions enable structured management of detection logic.
- Snort supports both proactive monitoring and retrospective forensic investigations.
- Offline packet analysis provides an effective method for validating detections during incident response.

---

# Analyst Reflection

One of the biggest lessons from this project is that detection tools are only as valuable as the rules they execute.

An IDS does not determine whether an event is malicious—it simply identifies activity that matches defined detection logic. The analyst remains responsible for validating alerts, correlating evidence across multiple data sources, and determining whether an incident requires escalation.

Developing effective detection rules is therefore just as important as investigating the alerts they produce.

## Tools Used

- Snort
- Linux
- ICMP
- PCAP

---
