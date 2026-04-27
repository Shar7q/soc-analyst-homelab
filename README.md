soc-analyst-homelab — [README.md](http://README.md)
# SOC Analyst Home Lab — Microsoft Sentinel

<aside>
  
  **A cloud-based SIEM home lab built on Microsoft Sentinel (Azure) to simulate SOC operations, detect threats, and automate incident response.**

</aside>

---

## Project Overview

This home lab replicates a real-world Security Operations Center (SOC) environment using Microsoft Sentinel on Azure. It demonstrates practical skills in SIEM configuration, log ingestion, threat detection, KQL-based threat hunting, and automated response using SOAR playbooks.

Built as part of my transition from QA Engineering into a SOC Analyst (L1) role.
Architecture
[Azure Activity Logs]
[Microsoft Entra ID Logs]  ──▶  [Log Analytics Workspace]  ──▶  [Microsoft Sentinel]
[Defender for Cloud]                                                      │
[Threat Intelligence Feed]                                     ┌──────────┴──────────┐
                                                               ▼                     ▼
                                                        Analytics Rules         Workbooks
                                                        (Auto-Detections)       (Dashboards)
                                                               │
                                                               ▼
                                                          Playbooks
                                                       (SOAR Automation)

## Tools & Technologies

- **Microsoft Sentinel** — Cloud-native SIEM/SOAR
- **Azure Log Analytics** — Log storage and querying
- **KQL (Kusto Query Language)** — Threat hunting queries
- **Microsoft Entra ID** — Identity & sign-in log source
- **Microsoft Defender for Cloud** — Security posture & alerts
- **Azure Logic Apps** — Playbook automation
- **TAXII Threat Intelligence Feed** — Live IOC ingestion

Data Connectors Configured
| Connector | Log Types | Purpose |
| --- | --- | --- |
| Azure Activity | Resource operations | Detect rare/suspicious Azure actions |
| Microsoft Entra ID | Sign-in & audit logs | Brute force, failed logins, MFA events |
| Defender for Cloud | Security alerts | Misconfigurations & threat detections |
| Threat Intelligence (TAXII) | IOCs (IPs, domains, hashes) | Match logs against live threat feeds |

## Detections & KQL Queries

### 1. Failed Login Detection
SigninLogs
| where ResultType != "0"
| summarize FailedAttempts = count() by UserPrincipalName, IPAddress
| where FailedAttempts > 3
| order by FailedAttempts desc
**MITRE ATT&CK:** T1110 — Brute Force

---

### 2. Sign-ins from Suspicious IPs
SigninLogs
| where RiskLevelDuringSignIn in ("medium", "high")
| project TimeGenerated, UserPrincipalName, IPAddress, Location, RiskLevelDuringSignIn
| order by TimeGenerated desc
**MITRE ATT&CK:** T1078 — Valid Accounts

---

### 3. Rare Azure Subscription Operations
AzureActivity
| where ActivityStatusValue == "Success"
| summarize count() by OperationNameValue, Caller
| where count_ < 3
| order by count_ asc
**MITRE ATT&CK:** T1098 — Account Manipulation

---

### 4. Multiple Account Lockouts
SigninLogs
| where ResultType == "50053"
| summarize LockoutCount = count() by UserPrincipalName, bin(TimeGenerated, 1h)
| where LockoutCount > 2
**MITRE ATT&CK:** T1110.001 — Password Guessing

---

### 5. Login Outside Business Hours
SigninLogs
| where ResultType == "0"
| extend Hour = datetime_part("Hour", TimeGenerated)
| where Hour < 6 or Hour > 22
| project TimeGenerated, UserPrincipalName, IPAddress, Location
**MITRE ATT&CK:** T1078 — Valid Accounts (Off-Hours Access)

---

MITRE ATT&CK Coverage
| Detection Rule | Tactic | Technique |
| --- | --- | --- |
| Brute Force Login | Credential Access | T1110 |
| Suspicious IP Sign-in | Initial Access | T1078 |
| Rare Subscription Operations | Persistence | T1098 |
| Account Lockout Spike | Credential Access | T1110.001 |
| Off-Hours Login | Defense Evasion | T1078 |

## Playbook — Automated Alert Response

Built a Logic App playbook that triggers on **High severity incidents** and automatically:

1. Captures incident details (title, severity, affected entities)
2. Sends an email notification with a full incident summary
3. Posts an alert to a designated Teams/Slack channel *(optional)*

**Why this matters:** Demonstrates understanding of SOAR — automating Tier 1 alert triage tasks that a SOC analyst would otherwise handle manually.
## Workbook Dashboard

Custom Sentinel Workbook built to monitor:

- Failed login attempts over time (line chart)
- Top 10 suspicious source IPs (bar chart)
- Active alerts by severity (pie chart)
- Geographic map of sign-in locations

Repository Structure
soc-analyst-homelab/
├── README.md
├── setup/
│   ├── sentinel-setup.md          # Step-by-step Azure + Sentinel setup
│   └── data-connectors.md         # Connector configuration notes
├── detections/
│   ├── failed-logins.kql
│   ├── suspicious-ip-signin.kql
│   ├── rare-operations.kql
│   ├── account-lockout.kql
│   └── off-hours-login.kql
├── playbooks/
│   └── auto-email-alert.md        # Playbook setup + screenshots
├── workbooks/
│   └── soc-dashboard.md           # Dashboard walkthrough + screenshots
└── screenshots/
    ├── sentinel-overview.png
    ├── analytics-rules.png
    ├── playbook-trigger.png
    └── soc-dashboard.png

## What I Learned

- How to ingest and normalize logs from multiple Azure data sources into a centralized SIEM
- Writing KQL queries to detect credential-based attacks, anomalous behavior, and privilege abuse
- Mapping detections to MITRE ATT&CK tactics and techniques for structured threat analysis
- How SOAR playbooks reduce mean time to respond (MTTR) by automating Tier 1 alert actions
- The difference between alert tuning (reducing false positives) and detection coverage gaps
## Connect

- LinkedIn: [linkedin.com/in/shariq-mesia-6a3304309](http://linkedin.com/in/shariq-mesia-6a3304309)
- GitHub: [github.com/Shar7q](http://github.com/Shar7q)
