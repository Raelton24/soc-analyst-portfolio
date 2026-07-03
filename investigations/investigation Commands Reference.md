# | Investigation Goal | Linux Command | Why It Matters |

| Investigation Goal      | Linux Command                | Purpose                          |
| ----------------------- | ---------------------------- | -------------------------------- |
| Identify reconnaissance | `grep "BLOCK" firewall.log`  | Find blocked inbound connections |
| Count attacker IPs      | `cut ... \| sort \| uniq -c` | Identify highest-volume scanners |
| Review VPN failures     | `grep FAIL vpn_auth.log`     | Detect brute-force activity      |
| Find successful logins  | `grep SUCCESS vpn_auth.log`  | Identify initial access          |
| Search SMB alerts       | `grep SMB ids_alerts.log`    | Investigate lateral movement     |
| Hunt for C2             | `grep C2 ids_alerts.log`     | Detect beaconing                 |
| Hunt exfiltration       | `grep "Large Upload"`        | Identify outbound data transfers |
