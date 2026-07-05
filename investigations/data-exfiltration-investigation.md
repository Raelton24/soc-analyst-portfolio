# Data Exfiltration Investigation

> **Portfolio Case Study** — Enterprise Data Exfiltration Investigation

## Executive Summary

This investigation documents the analysis of multiple simulated data exfiltration techniques observed within an enterprise environment. Using **Wireshark** for packet analysis and **Splunk** for log correlation, four independent exfiltration methods were investigated:

- DNS Tunneling
- FTP Exfiltration
- HTTP POST Exfiltration
- ICMP Tunneling

The objective was to identify compromised hosts, determine the protocol used for exfiltration, correlate network and log evidence, preserve indicators of compromise (IOCs), map activity to MITRE ATT&CK, and recommend detection opportunities.

## Incident Overview

Internal monitoring identified abnormal outbound network activity from several workstations. The investigation focused on determining whether sensitive information had been exfiltrated, identifying the affected hosts, reconstructing attacker behaviour, and documenting detection opportunities.

## Investigation Objectives

- Detect suspicious outbound communications.
- Identify compromised systems.
- Determine the protocol used for exfiltration.
- Correlate packet captures with SIEM logs.
- Recover investigative evidence.
- Document attacker behaviour.
- Recommend improved detections.

## Environment

### Tools

- Wireshark
- Splunk
- PCAP Files
- DNS Logs
- HTTP Logs
- FTP Traffic

### Protocols Investigated

- DNS
- FTP
- HTTP
- ICMP

## Investigation Workflow

```
Alert
 ↓
Scope Investigation
 ↓
Identify Protocol
 ↓
Packet Analysis
 ↓
Log Correlation
 ↓
Evidence Collection
 ↓
IOC Extraction
 ↓
MITRE ATT&CK Mapping
 ↓
Detection Engineering
 ↓
Incident Summary
```

# Investigation 1 — DNS Tunneling

## Goal

Identify covert DNS-based data exfiltration.

## Methodology

Reviewed DNS traffic in Wireshark before correlating with Splunk DNS logs to identify abnormal query behaviour.

### Wireshark Filters

| Filter                                             | Purpose                   |
| -------------------------------------------------- | ------------------------- |
| `dns`                                              | Show all DNS traffic      |
| `dns.flags.response == 0`                          | DNS queries only          |
| `dns && frame.len > 70`                            | Large DNS packets         |
| `dns && dns.qry.name contains <suspicious_domain>` | Isolate suspicious domain |

### Splunk Queries

```spl
index=data_exfil sourcetype=DNS_logs
```

```spl
index=data_exfil sourcetype=DNS_logs | stats count by src_ip
```

```spl
index=data_exfil sourcetype=dns_logs | stats count by query | sort -count
```

```spl
index=data_exfil sourcetype=DNS_logs | where len(query)>30
```

### Findings

- Multiple hosts generated abnormal DNS requests.
- Long DNS query names indicated encoded data.
- Large query volume targeted a single external domain.
- Behaviour was consistent with DNS tunneling.

### Detection Logic

- Alert on unusually long DNS query names.
- Alert on high query volume to a single external domain.
- Detect repeated NXDOMAIN responses.
- Detect high-entropy subdomains.

---

# Investigation 2 — FTP Exfiltration

## Goal

Detect unauthorized file uploads using FTP.

### Wireshark Filters

| Filter                       | Purpose           |
| ---------------------------- | ----------------- | ---------------------------- | -------------------- |
| `ftp                         |                   | ftp-data`                    | FTP control and data |
| `ftp.request.command=="USER" |                   | ftp.request.command=="PASS"` | Credentials          |
| `ftp contains "STOR"`        | Upload operations |
| `ftp contains "csv"`         | CSV transfers     |
| `ftp && frame.len > 90`      | Large transfers   |

### Evidence

- Guest account established **5** FTP connections.
- Sensitive file **customer_data.xlsx** uploaded.
- Largest payload originated from **192.168.1.105**.

### Detection Logic

- Alert on guest FTP authentication.
- Alert on STOR operations involving business documents.
- Alert on large outbound FTP transfers.

---

# Investigation 3 — HTTP POST Exfiltration

## Goal

Detect large outbound HTTP uploads.

### Splunk Queries

```spl
index="data_exfil" sourcetype="http_logs"
```

```spl
index="data_exfil" sourcetype="http_logs" method=POST
```

```spl
index="data_exfil" sourcetype="http_logs" method=POST | stats count avg(bytes_sent) max(bytes_sent) min(bytes_sent) by domain | sort -count
```

```spl
index="data_exfil" sourcetype="http_logs" method=POST bytes_sent>600 | table _time src_ip uri domain dst_ip bytes_sent | sort -bytes_sent
```

### Wireshark Filters

| Filter                                          | Purpose                   |
| ----------------------------------------------- | ------------------------- |
| `http`                                          | HTTP traffic              |
| `http.request.method=="POST"`                   | POST requests             |
| `http.request.method=="POST" and frame.len>500` | Large uploads             |
| `http.request.method=="POST" and frame.len>750` | Isolate suspicious upload |

### Findings

- Large outbound POST request identified.
- Correlated with SIEM evidence.
- HTTP stream confirmed sensitive document exfiltration.

### Detection Logic

- Detect unusually large POST requests.
- Detect uploads to uncommon domains.
- Detect repeated uploads outside baseline.

---

# Investigation 4 — ICMP Exfiltration

## Goal

Identify covert exfiltration using ICMP.

### Wireshark Filters

| Filter                           | Purpose        |
| -------------------------------- | -------------- |
| `icmp`                           | All ICMP       |
| `icmp.type==8`                   | Echo Requests  |
| `icmp.type==8 and frame.len>100` | Large payloads |

### Findings

- Large ICMP Echo Requests carried abnormal payloads.
- Behaviour consistent with ICMP tunneling.

### Detection Logic

- Alert on oversized ICMP payloads.
- Detect periodic ICMP communications.
- Detect sustained outbound ICMP to external hosts.

# Indicators of Compromise

| IOC  | Observation                 |
| ---- | --------------------------- |
| DNS  | Suspicious long DNS queries |
| FTP  | customer_data.xlsx uploaded |
| FTP  | Guest account activity      |
| FTP  | 192.168.1.105               |
| HTTP | Large outbound POST upload  |
| ICMP | Oversized Echo Requests     |

# MITRE ATT&CK Mapping

| Tactic / Technique                     | Reason     |
| -------------------------------------- | ---------- |
| Exfiltration Over Alternative Protocol | DNS & ICMP |
| Exfiltration Over Web Service          | HTTP POST  |
| Exfiltration Over Unencrypted Protocol | FTP        |

# Detection Engineering Opportunities

- Create SIEM analytics for abnormal DNS query length.
- Detect high-volume DNS requests to uncommon domains.
- Alert on FTP STOR commands involving sensitive file extensions.
- Monitor outbound HTTP POST requests exceeding baseline.
- Monitor ICMP payload sizes significantly above normal.
- Correlate endpoint archive creation with outbound network transfers.

# Skills Demonstrated

- Packet Analysis
- Wireshark
- Splunk
- Network Forensics
- DNS Analysis
- FTP Investigation
- HTTP Investigation
- ICMP Investigation
- Log Correlation
- Threat Hunting
- IOC Identification
- MITRE ATT&CK Mapping
- Detection Engineering
- Incident Documentation

# Lessons Learned

- Legitimate protocols can be abused for data exfiltration.
- Effective detection requires correlating endpoint and network telemetry.
- Packet captures provide evidence that complements SIEM logs.
- Behaviour-based detection is more resilient than relying solely on signatures.

# Analyst Reflection

This investigation reinforced that successful data exfiltration detection depends on understanding normal protocol behaviour before identifying abuse. Although HTTP and FTP are common exfiltration channels, DNS and ICMP demonstrate how adversaries can leverage trusted protocols to establish covert channels. Across all four investigations, the strongest conclusions came from correlating packet captures, SIEM telemetry, protocol analysis, and contextual evidence rather than relying on a single alert source.
