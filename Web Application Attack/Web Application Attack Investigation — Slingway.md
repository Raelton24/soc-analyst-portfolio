# Web Application Attack Investigation — Slingway

## Investigation Overview

This investigation analyzed web server and ModSecurity audit logs to reconstruct an attacker’s activity against the Slingway web application.

The investigation focused on identifying:

- Attacker activity and source IP

- Web enumeration and reconnaissance

- Authentication attempts

- Successful administrative access

- Local File Inclusion (LFI)

- Database credential retrieval

- phpMyAdmin access

- Database export activity

- Malicious database modification

- The flag inserted into the database

The investigation was performed using Elasticsearch/Kibana log data collected through Filebeat from the Apache ModSecurity audit log.

---

## Environment

| Component         | Details                             |

| ----------------- | ----------------------------------- |

| Web Server        | Apache                              |

| Log Source        | ModSecurity Audit Log               |

| Log Path          | `/var/log/apache2/modsec_audit.log` |

| Log Collection    | Filebeat 8.8.2                      |

| Analysis Platform | Elasticsearch / Kibana              |

| Target Host       | `slingway.thm`                      |

| Target IP         | `10.0.2.4`                          |

| Attacker IP       | `10.0.2.15`                         |

---

# 1. Investigation Objective

The objective was to analyze HTTP activity associated with the attacker and reconstruct the attack chain from initial reconnaissance through compromise and database manipulation.

The investigation relied primarily on:

- `transaction.remote_address`

- `http.url`

- `http.method`

- `request.request_line`

- `request.headers`

- `request.body`

- `response.status`

- `message`

A key investigative principle throughout the investigation was to progressively narrow the dataset using known indicators and attacker behavior.

---

# 2. Initial Reconnaissance

The attacker first performed web directory enumeration against the target.

An example event showed:

```text

GET /backups/?flag=a76637b62ea99acda12f5859313f539a HTTP/1.1

User-Agent: Mozilla/5.0 (Gobuster)



```

Source:

10.0.2.15

The Gobuster User-Agent strongly indicated automated directory/content discovery.

The server returned:

HTTP/1.1 200

This demonstrated that the discovered /backups/ resource existed and was accessible.

Investigation Query

transaction.remote_address: "10.0.2.15"

The investigation was then narrowed using HTTP paths and response codes to distinguish discovered resources from unsuccessful requests.

3. Administrative Page Discovery

Further investigation identified an administrative login endpoint:

GET /admin-login.php HTTP/1.1

User-Agent: Mozilla/4.0 (Hydra)

Response: 401

The Hydra User-Agent indicated automated authentication attempts.

The 401 Unauthorized response was significant because it showed that the endpoint existed but authentication had not yet succeeded.

Finding

/admin-login.php

This established the administrative login page as a target of the attacker.

4. Successful Authentication

The investigation subsequently identified a successful authentication event.

The credentials used were:

admin:thx1138

The investigation required correlating authentication-related requests with successful HTTP responses and examining relevant request data.

This represented a transition from reconnaissance and attempted authentication to authenticated access.

5. Local File Inclusion

After gaining administrative access, the attacker targeted the application's settings functionality.

The following request was identified:

GET /admin/settings.php?page=../../../../../../../../etc/phpmyadmin/config-db.php HTTP/1.1

The request contained repeated directory traversal sequences:

../../../../../../../../

The attacker attempted to access:

/etc/phpmyadmin/config-db.php

The server returned:

HTTP 200

Analysis

The page parameter was being manipulated to traverse outside the intended application directory and retrieve a local file.

This is characteristic of a Local File Inclusion (LFI) attack.

The targeted configuration file was associated with phpMyAdmin and contained database-related credentials.

Attack Technique

Local File Inclusion (LFI)

MITRE ATT&CK

T1005 — Data from Local System

The attacker accessed a locally stored configuration file containing sensitive information.

6. phpMyAdmin Access

After obtaining database-related credentials, the attacker interacted with the phpMyAdmin interface.

The investigation identified extensive activity under:

/phpmyadmin/

The attacker interacted with database administration functionality including import and export operations.

At this stage, the investigation shifted from general web application activity to database-related activity.

7. Database Export

The attacker accessed phpMyAdmin's table export functionality.

A relevant request was:

GET /phpmyadmin/tbl_export.php?db=customer_credit_cards&table=credit_cards&single_table=true HTTP/1.1

The request explicitly identified the database:

customer_credit_cards

and the table:

credit_cards

Finding

The attacker exported data from:

customer_credit_cards

This represented successful access to sensitive database information.

8. Database Modification

The investigation then identified an important POST request to:

/phpmyadmin/import.php

The request body contained an SQL INSERT operation.

Relevant SQL:

INSERT INTO `credit_cards`

(`card_number`, `cardholder_name`, `expiration_date`, `cvv`)

VALUES ('000', 'c6aa3215a7d519eeb40a660f3b76e64c', '000', '000');

The request returned:

HTTP 200

and the request body contained:

Your SQL query has been executed successfully.

Finding

The attacker successfully inserted a record into the credit_cards table.

The value inserted as the cardholder_name was:

c6aa3215a7d519eeb40a660f3b76e64c

This was the flag inserted into the database.

9. Attack Timeline

Time (UTC)  Activity  Evidence

14:27:44  Directory enumeration GET /backups/

14:27:44  Automated reconnaissance  User-Agent: Gobuster

Later Administrative endpoint discovered  /admin-login.php

Later Automated authentication attempts User-Agent: Hydra

Later Successful administrative authentication  admin:thx1138

14:31:38  LFI attempt /admin/settings.php?page=../../../../../../../../etc/phpmyadmin/config-db.php

Later phpMyAdmin access /phpmyadmin/

Later Database export /phpmyadmin/tbl_export.php?db=customer_credit_cards...

14:34:45  Database modification POST /phpmyadmin/import.php

14:34:45  SQL INSERT executed INSERT INTO credit_cards...

10. Attack Chain

The investigation reconstructed the following sequence:

Web Reconnaissance

       ↓

Directory Enumeration

       ↓

Administrative Page Discovery

       ↓

Brute-Force Authentication

       ↓

Successful Admin Access

       ↓

Local File Inclusion

       ↓

Database Credential Retrieval

       ↓

phpMyAdmin Access

       ↓

Database Export

       ↓

Database Modification

       ↓

Flag Inserted

11. Key Indicators of Compromise

Attacker

10.0.2.15

Target

slingway.thm

10.0.2.4

Reconnaissance

User-Agent: Mozilla/5.0 (Gobuster)

Authentication Attack

User-Agent: Mozilla/4.0 (Hydra)

Administrative Endpoint

/admin-login.php

LFI Target

/etc/phpmyadmin/config-db.php

Database Interface

/phpmyadmin/

Database

customer_credit_cards

Export Endpoint

/phpmyadmin/tbl_export.php

Import Endpoint

/phpmyadmin/import.php

Database Modification

INSERT INTO credit_cards

12. Investigation Methodology

The investigation demonstrated the importance of progressively narrowing a large log dataset.

Rather than manually reviewing every event, the investigation used known context to construct increasingly targeted searches.

For example:

Identify attacker activity

transaction.remote_address: "10.0.2.15"

Investigate phpMyAdmin activity

transaction.remote_address: "10.0.2.15"

AND http.url: _phpmyadmin_

Identify database modifications

transaction.remote_address: "10.0.2.15"

AND http.url: _phpmyadmin_

AND request.body: _INSERT_

The final query was particularly useful because the question explicitly described an insertion into the database.

This allowed the investigation to move directly toward the request body containing the SQL statement.

13. Lessons From the Investigation

Several investigation techniques were particularly useful.

1. HTTP status codes provide context

A 401 response to /admin-login.php indicated that the endpoint existed but authentication had failed.

A later successful response provided evidence of successful access.

2. User-Agent values can reveal attacker tooling

The following values provided useful behavioral indicators:

Gobuster

Hydra

These helped distinguish automated reconnaissance and authentication activity from normal browser traffic.

3. Request parameters can expose attack techniques

The following parameter immediately revealed directory traversal:

page=../../../../../../../../etc/phpmyadmin/config-db.php

4. Request bodies can contain the most important evidence

The final database modification was not obvious from the URL alone.

The critical evidence was inside:

request.body

Specifically:

INSERT INTO `credit_cards`

This is an important lesson when investigating HTTP-based attacks: do not limit analysis to URLs and request lines. Inspect request bodies when available.

14. Security Recommendations

Based on the observed attack chain, the following defensive controls should be considered.

Web Application

Validate and sanitize file inclusion parameters.

Prevent directory traversal.

Implement allowlists for files that can be loaded through application parameters.

Remove sensitive configuration files from web-accessible environments.

Apply least-privilege principles to web application accounts.

Authentication

Enforce strong administrative credentials.

Implement account lockout or rate limiting.

Monitor repeated authentication failures.

Restrict administrative interfaces to trusted networks where possible.

Monitor authentication activity associated with automated tools.

phpMyAdmin

Do not expose phpMyAdmin unnecessarily to the public internet.

Restrict access by network or VPN.

Enforce strong authentication.

Monitor database export/import operations.

Monitor administrative database activity.

Logging & Detection

Monitor for:

../

../../

/etc/passwd

/etc/phpmyadmin/

phpmyadmin

import.php

tbl_export.php

INSERT INTO

SELECT

User-Agent: Gobuster

User-Agent: Hydra

Correlating these indicators with a single source IP can help identify an attack progression rather than treating each event independently.

15. Final Assessment

The investigation identified a multi-stage web application compromise originating from:

10.0.2.15

The attacker performed automated web enumeration, discovered an administrative login page, conducted authentication attempts, and subsequently obtained administrative access.

The attacker then exploited a Local File Inclusion vulnerability to retrieve:

/etc/phpmyadmin/config-db.php

The retrieved information enabled access to phpMyAdmin, where the attacker accessed the:

customer_credit_cards

database, exported sensitive data, and subsequently modified the credit_cards table through an SQL INSERT operation.

The investigation demonstrates how seemingly separate HTTP events can be correlated into a coherent attack chain through:

Source IP correlation

HTTP status analysis

URL and parameter analysis

User-Agent analysis

Request body inspection

SQL operation identification

Temporal correlation

The primary investigative lesson is that effective SIEM investigation is not simply searching logs for keywords; it is using the available evidence and the known attacker behavior to progressively reduce the search space until the relevant event is isolated.
