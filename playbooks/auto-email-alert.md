# Playbook — Automated Email Alert on High Severity Incident

## Overview
An Azure Logic App playbook that triggers automatically when Microsoft Sentinel
creates a High or Critical severity incident. It sends an email notification
with full incident details for immediate analyst awareness.

---

## Why This Matters
This playbook automates Tier 1 alert notification — one of the most repetitive
SOC tasks. Instead of an analyst manually checking the Sentinel dashboard,
the playbook pushes incident details directly to their inbox within seconds
of detection.

**SOAR concept demonstrated:** Automated alert triage notification (MITRE ATT&CK: detection → response loop)

---

## Prerequisites
- Microsoft Sentinel workspace active
- Azure Logic Apps enabled in your subscription
- An email account to receive notifications (Outlook or Gmail)

---

## Step-by-Step Build Guide

### Step 1 — Open Automation in Sentinel
1. Go to **Microsoft Sentinel → Automation**
2. Click **"+ Create"** → **"Playbook with incident trigger"**
3. Fill in:
   - **Subscription:** Azure subscription 1
   - **Resource group:** soc-homelab-rg
   - **Playbook name:** `SOC-Email-Alert-HighSeverity`
   - **Region:** East US
4. Click **"Review + create"** → **"Create and continue to designer"**

---

### Step 2 — Configure the Trigger
The Logic App designer opens with the trigger pre-set:
- **Trigger:** `Microsoft Sentinel incident (V3)`
- This fires every time Sentinel creates or updates an incident

---

### Step 3 — Add a Severity Filter Condition
1. Click **"+ New step"** → search **"Condition"**
2. Configure:
   - **Value:** `@triggerBody()?['object']?['properties']?['severity']`
   - **Condition:** `is equal to`
   - **Value:** `High`
3. Click **"+ Add"** → **"Add row"** → add a second condition:
   - **Value:** same expression
   - **Condition:** `is equal to`
   - **Value:** `Incident`  *(change to `Critical`)*
4. Set the combinator to **"Or"**

This ensures only High and Critical incidents trigger the email.

---

### Step 4 — Add Send Email Action (True Branch)

Inside the **"True"** branch of the condition:

1. Click **"+ Add an action"**
2. Search **"Send an email"** → select **Office 365 Outlook** or **Gmail**
3. Sign in to your email account when prompted
4. Configure the email using the values below.

    To:
    your-email@gmail.com

    Subject:
    🚨 Sentinel Alert: @{triggerBody()?['object']?['properties']?['severity']} Incident — @{triggerBody()?['object']?['properties']?['title']}

    Body:
    ⚠️ NEW SENTINEL INCIDENT DETECTED

    Incident Title:   @{triggerBody()?['object']?['properties']?['title']}
    Severity:         @{triggerBody()?['object']?['properties']?['severity']}
    Status:           @{triggerBody()?['object']?['properties']?['status']}
    Incident Number:  @{triggerBody()?['object']?['properties']?['incidentNumber']}
    Created Time:     @{triggerBody()?['object']?['properties']?['createdTimeUtc']}

    Description:
    @{triggerBody()?['object']?['properties']?['description']}

    View in Sentinel:
    @{triggerBody()?['object']?['properties']?['incidentUrl']}

    ---
    Automated alert from SOC Home Lab — Microsoft Sentinel

---

### Step 5 — Save and Enable the Playbook
1. Click **"Save"** in the Logic App designer
2. Go back to **Sentinel → Automation**
3. Click **"+ Create"** → **"Automation rule"**
4. Configure:
   - **Name:** `Trigger Email on High Severity`
   - **Trigger:** When incident is created
   - **Conditions:** Severity = High OR Critical
   - **Actions:** Run playbook → `SOC-Email-Alert-HighSeverity`
5. Click **"Apply"**

---

### Step 6 — Test the Playbook
1. Go to **Sentinel → Incidents**
2. Click any existing incident → **"Actions"** → **"Run playbook"**
3. Select `SOC-Email-Alert-HighSeverity` → **"Run"**
4. Check your inbox — the alert email should arrive within 30 seconds
5. ✅ **Screenshot:** Email received with incident details

---

## What This Proves

| Skill | Evidence |
|---|---|
| SOAR configuration | Built and deployed a Logic App playbook |
| Incident response automation | Playbook auto-notifies on High/Critical severity |
| Azure Logic Apps | Configured multi-step workflow with conditions |
| Sentinel integration | Playbook tied to Sentinel automation rules |
| Tier 1 triage automation | Eliminates manual dashboard polling |

---

## Screenshots
See `screenshots/playbook-trigger.png` for the Logic App designer view.
See `screenshots/playbook-email.png` for the received alert email.
