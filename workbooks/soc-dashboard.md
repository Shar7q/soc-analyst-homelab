# SOC Dashboard Workbook — Microsoft Sentinel

## Overview
Custom Sentinel Workbook built to provide a real-time SOC operations view. Monitors authentication activity, threat intelligence matches, and alert severity distribution.

---

## How to Access
1. Go to **Microsoft Sentinel → Workbooks**
2. Click **"My workbooks"** tab
3. Open **"SOC Overview Dashboard"**

---

## Dashboard Sections

### 1. Failed Login Attempts Over Time
**Visualization:** Line chart
SigninLogs

| where ResultType != "0" |
| --- |
| summarize FailedLogins = count() by bin(TimeGenerated, 1h) |
| order by TimeGenerated asc |

**What it shows:** Spikes in failed logins indicate brute force or credential stuffing attempts.

---

### 2. Top 10 Suspicious Source IPs
**Visualization:** Bar chart
SigninLogs

| where ResultType != "0" |
| --- |
| summarize AttemptCount = count() by IPAddress |
| order by AttemptCount desc |
| take 10 |

**What it shows:** Identifies the most aggressive source IPs targeting the environment.

---

### 3. Active Alerts by Severity
**Visualization:** Pie chart
SecurityAlert

| summarize count() by AlertSeverity |
| --- |
| order by count_ desc |

**What it shows:** Distributes active alerts across Critical, High, Medium, and Low severity levels for triage prioritization.

---

### 4. Geographic Sign-in Map
**Visualization:** Map tile
SigninLogs

| where ResultType == "0" |
| --- |
| project TimeGenerated, UserPrincipalName, IPAddress, |
Latitude = toreal(LocationDetails.geoCoordinates.latitude),
Longitude = toreal(LocationDetails.geoCoordinates.longitude),
City = tostring(LocationDetails.city),
Country = tostring(LocationDetails.countryOrRegion)
| where isnotempty(Latitude) |
| --- |

**What it shows:** Geographic origin of successful sign-ins — anomalies like logins from unexpected countries stand out immediately.

---

### 5. Threat Intel IOC Feed Summary
**Visualization:** Table
ThreatIntelIndicators

| where IsActive == true |
| --- |
| summarize count() by Type, ObservableKey |
| order by count_ desc |

**What it shows:** Summary of active threat intel indicators by type (IP, domain, URL, file hash).

---

### 6. Audit Log Failures
**Visualization:** Table
AuditLogs

| where Result == "failure" |
| --- |
| project TimeGenerated, OperationName, Result, InitiatedBy |
| order by TimeGenerated desc |
| take 20 |

**What it shows:** Recent failed operations in Azure AD — useful for detecting unauthorized access attempts or misconfigured automation.

---

## How to Build This Workbook in Sentinel
1. Go to **Sentinel → Workbooks → + New**
2. Click **"</> Advanced Editor"**
3. For each section above:
   - Click **"+ Add"** → **"Add query"**
   - Paste the KQL query
   - Select the visualization type from the dropdown
   - Set time range to **Last 24 hours**
4. Click **"Done Editing"** → **Save**
5. Name it: `SOC Overview Dashboard`

---

## Screenshots
See `screenshots/soc-dashboard.png` for the live dashboard view.
