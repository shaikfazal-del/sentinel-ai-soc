<div align="center">

<img src="https://img.shields.io/badge/Microsoft%20Sentinel-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white" />
<img src="https://img.shields.io/badge/Azure%20Logic%20Apps-0066FF?style=for-the-badge&logo=microsoftazure&logoColor=white" />
<img src="https://img.shields.io/badge/OpenAI%20GPT-412991?style=for-the-badge&logo=openai&logoColor=white" />
<img src="https://img.shields.io/badge/SIEM%20%2F%20SOAR-red?style=for-the-badge&logo=shield&logoColor=white" />
<img src="https://img.shields.io/badge/Free%20Tier-10GB%2FDay-00C853?style=for-the-badge" />

<br /><br />

# 🛡️ Microsoft Sentinel — AI-Powered SIEM / SOAR

### A cloud-native Security Operations Centre with GPT-driven automated incident response

*Built on Microsoft Azure · Behaviour-based detection · Zero-cost free-tier deployment*

<br />

</div>

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Step-by-Step Setup](#-step-by-step-setup)
  - [1. Deploy Sentinel All-In-One](#1-deploy-sentinel-all-in-one)
  - [2. Fix Diagnostic Settings](#2-fix-diagnostic-settings)
  - [3. Open Microsoft Sentinel](#3-open-microsoft-sentinel)
  - [4. Enable UEBA](#4-enable-ueba-behaviour-based-detection)
  - [5. Playbook Permissions](#5-configure-playbook-permissions)
  - [6. Create the Watchlist](#6-create-the-watchlist--tor-exit-nodes)
  - [7. Create Analytics Rules](#7-create-analytics-rules)
  - [8. Build the GPT Playbook](#8-build-the-ai-playbook-logic-app)
  - [9. Grant IAM Permissions](#9-grant-sentinel-responder-role-to-the-logic-app)
  - [10. Create the Automation Rule](#10-create-the-automation-rule)
  - [11. Test the Pipeline](#11-test-the-full-pipeline)
- [Cost Management](#-cost-management--free-tier-tips)
- [Troubleshooting](#-troubleshooting)
- [Glossary](#-glossary)

---

## 🔍 Overview

This project implements a **fully automated, AI-augmented Security Operations Centre (SOC)** using Microsoft Sentinel on Azure. When a security incident is detected, an Azure Logic App playbook fires automatically, queries the OpenAI GPT API for an AI-generated analysis, and posts the recommendation as a comment directly on the incident — giving analysts an immediate head-start on investigation.

The entire stack runs on the **Azure free tier**, which provides **10 GB of free log ingestion per day** for the first 31 days.

```
Log Sources → Log Analytics Workspace → Microsoft Sentinel
                                              │
                              ┌───────────────▼────────────────┐
                              │   Analytics Rules (KQL)        │
                              │   UEBA Behavioural Baseline     │
                              │   Watchlist (Tor Exit Nodes)    │
                              └───────────────┬────────────────┘
                                              │  Incident Created
                              ┌───────────────▼────────────────┐
                              │   Automation Rule              │
                              │   └─► Logic App Playbook       │
                              │         └─► OpenAI GPT API     │
                              │               └─► AI Comment   │
                              │                   on Incident  │
                              └────────────────────────────────┘
```

---

## 🏗️ Architecture

| Layer | Service | Role |
|-------|---------|------|
| **Log Storage** | Azure Log Analytics Workspace | Ingests and stores all security logs |
| **SIEM** | Microsoft Sentinel | Detection, incident management, analytics |
| **Threat Intel** | Watchlist (Tor Exit Nodes CSV) | Enriches rules with known malicious IPs |
| **Behavioural Analytics** | UEBA | Detects anomalies against user/entity baselines |
| **SOAR** | Azure Logic Apps | Serverless automation playbooks |
| **AI Engine** | OpenAI GPT-3.5-Turbo | Generates natural-language incident analysis |
| **Identity & Access** | Azure IAM (Managed Identity) | Securely connects Logic App to Sentinel |

---

## ✨ Features

- ✅ **One-click deployment** via the Sentinel All-In-One ARM template
- ✅ **10 GB/day free log ingestion** within Azure free-tier limits
- ✅ **UEBA** — machine-learning behavioural baselines for users, devices, and hosts
- ✅ **Tor exit-node watchlist** — detects sign-ins from anonymisation infrastructure
- ✅ **Custom KQL analytics rules** with MITRE ATT&CK tactic mapping
- ✅ **GPT-powered automated incident analysis** posted as incident comments
- ✅ **Automation rules** that trigger playbooks the moment an incident is created
- ✅ **Role-based access** using Azure Managed Identity (no stored credentials)
- ✅ **Cost guardrails** — daily cap, budget alerts, and connector hygiene guidance

---

## 🧰 Prerequisites

Before you begin, make sure you have:

- [ ] A **Microsoft Azure account** — [Create a free account](https://azure.microsoft.com/free/)
- [ ] **Contributor** or **Owner** role on the Azure subscription
- [ ] An **OpenAI account** with an API key — [Get one here](https://platform.openai.com/api-keys)
- [ ] A modern web browser

> **Note:** The Azure free account provides $200 credit for 30 days. Microsoft Sentinel's free trial grants 10 GB/day of log ingestion at no cost for the first 31 days.

---

## 🚀 Step-by-Step Setup

### 1. Deploy Sentinel All-In-One

1. Open the official repository:
   ```
   https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Sentinel-All-In-One
   ```

2. Click the **Deploy to Azure** button — you will be redirected to the Azure Portal custom deployment blade.

3. Configure the deployment parameters:

   | Parameter | Recommended Value | Notes |
   |-----------|-------------------|-------|
   | Subscription | Your free-tier subscription | — |
   | Resource Group | `rg-sentinel-lab` (create new) | **Note this name — used throughout** |
   | Region / Location | Closest to your users | Lower latency and egress cost |
   | Workspace Name | `law-sentinel-lab` | **Note this name — used throughout** |
   | Daily Ingestion Cap | 10 GB | Free-tier maximum |

4. **Settings tab** → Enable `Enable Sentinel health diagnostics` ✅

5. **Content Hub Solutions tab** → Click **Select All** for all three solution categories.

6. **Data Connectors tab** → Click **Select All**.
   > ⚠️ Some connectors require paid licences. A partial deployment failure from these connectors is **expected and safe to ignore**.

7. **Analytics tab** → Tick the top checkbox to **Select All** rule templates.

8. Click **Review + Create** → **Create**.
   - Deployment takes approximately **10–15 minutes**.
   - A `Failed` status is expected due to unlicensed connectors. The Sentinel workspace will still be healthy.

---

### 2. Fix Diagnostic Settings

After deployment (even if partially failed):

1. In the Azure Portal left sidebar → **Resource Groups** → click `rg-sentinel-lab`.
2. Click the **Log Analytics Workspace** resource (`law-sentinel-lab`).
3. In the workspace menu → **Monitoring** → **Diagnostic settings** → **+ Add diagnostic setting**.
4. Configure:
   - **Name:** `sentinel-diagnostics`
   - **Logs:** Select all categories
   - **Destination:** Send to Log Analytics Workspace → select `law-sentinel-lab`
5. Click **Save**.

---

### 3. Open Microsoft Sentinel

1. In the Azure Portal search bar, search for **Microsoft Sentinel**.
2. Click on your workspace (`law-sentinel-lab`).

> All remaining steps happen inside the Microsoft Sentinel portal unless stated otherwise.

---

### 4. Enable UEBA (Behaviour-Based Detection)

User and Entity Behaviour Analytics (UEBA) builds ML baselines and detects deviations.

1. In Sentinel → **Settings** → find and click **Set UEBA**.
2. Enable the following entity types:
   - [x] Users
   - [x] Groups
   - [x] Hosts / Devices
   - [x] Azure Resources *(if applicable)*
3. Click **Apply**.

> **Note:** UEBA requires 24–48 hours to accumulate baseline data before generating meaningful anomaly scores.

---

### 5. Configure Playbook Permissions

Sentinel needs permission to trigger Logic App playbooks in your resource group.

1. In Sentinel → **Settings** → scroll to **Playbook permissions**.
2. Click **Configure permissions**.
3. Select your resource group (`rg-sentinel-lab`).
4. Click **Apply**.

---

### 6. Create the Watchlist — Tor Exit Nodes

A Watchlist imports threat intelligence (as CSV) and makes it queryable via KQL.

1. Download the Tor exit-nodes CSV from:
   ```
   https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Sentinel-All-In-One
   ```
   *(or generate a fresh list from https://check.torproject.org/exit-addresses — format as CSV with columns `IpAddress, Description`)*

2. In Sentinel → **Watchlist** → click **+ New**.
3. Fill in:
   - **Name:** `TorExitNodes`
   - **Alias:** `TorExitNodes`
   - **Description:** Known Tor exit-node IP addresses
4. Upload the CSV file. Set **SearchKey** to `IpAddress`.
5. Click **Review + Create** → **Create**.

**Using the watchlist in KQL:**

```kql
let TorExitNodes = (_GetWatchlist('TorExitNodes') | project IpAddress);
SigninLogs
| where IPAddress in (TorExitNodes)
| project TimeGenerated, UserPrincipalName, IPAddress, Location, ResultType
```

---

### 7. Create Analytics Rules

Analytics Rules are the detection engine — they run KQL queries on a schedule and generate incidents when matches are found.

**Creating a Scheduled Rule (example: Tor Sign-In Detection):**

1. In Sentinel → **Analytics** → **+ Create** → **Scheduled query rule**.
2. **General tab:**
   - Name: `Tor Exit Node Sign-In Detected`
   - Severity: `High`
   - Tactics: `Initial Access` (MITRE ATT&CK)
3. **Set rule logic tab** — paste the KQL query:
   ```kql
   let TorExitNodes = (_GetWatchlist('TorExitNodes') | project IpAddress);
   SigninLogs
   | where IPAddress in (TorExitNodes)
   | project TimeGenerated, UserPrincipalName, IPAddress, Location, ResultType
   ```
   - Rule runs every: `5 Minutes`
   - Lookup data from last: `5 Minutes`
4. **Incident settings tab** → Enable `Create incidents from alerts`.
5. **Automated response tab** → Attach the GPT playbook *(after completing Step 8)*.
6. Click **Review + Create** → **Save**.

**Available Rule Types:**

| Type | Description |
|------|-------------|
| **Scheduled** | KQL query on a timer — best for known attack patterns |
| **Microsoft Security** | Auto-creates incidents from Defender alerts |
| **Fusion** | ML correlation of low-fidelity signals into multi-stage attacks |
| **ML Behavioural Analytics** | Built-in Microsoft ML anomaly models |
| **Anomaly** | UEBA-powered deviation signals |
| **Threat Intelligence** | Matches IOCs from TI connectors against logs |

---

### 8. Build the AI Playbook (Logic App)

This Logic App calls OpenAI GPT when an incident fires and posts the AI analysis as a comment on the incident.

#### 8a. Create the Playbook

1. In Sentinel → **Automation** → **+ Create** → **Playbook**.
2. Configure:
   - **Playbook name:** `GPT-Incident-Responder`
   - **Region:** Same as workspace
   - **Resource Group:** `rg-sentinel-lab`
3. Click **Review + Create** → **Create** → **Go to resource** (opens Logic App Designer).

#### 8b. Build the Logic App Flow

The designer opens with a **Microsoft Sentinel Incident trigger** already present.

1. Click **+ New Step** → search for **HTTP** → select the HTTP action.
2. Configure the HTTP action:

   | Field | Value |
   |-------|-------|
   | Method | `POST` |
   | URI | `https://api.openai.com/v1/chat/completions` |
   | Content-Type | `application/json` |
   | Authorization | `Bearer <YOUR_OPENAI_API_KEY>` |

3. **Body** — paste this JSON (the `@{...}` expressions pull values from the Sentinel incident):

   ```json
   {
     "model": "gpt-3.5-turbo",
     "messages": [
       {
         "role": "system",
         "content": "You are a cybersecurity analyst. Analyse the incident and provide: a brief summary, the likely attack vector, recommended containment actions, and suggested next steps."
       },
       {
         "role": "user",
         "content": "Incident Title: @{triggerBody()?['object']?['properties']?['title']}\nSeverity: @{triggerBody()?['object']?['properties']?['severity']}\nDescription: @{triggerBody()?['object']?['properties']?['description']}"
       }
     ],
     "max_tokens": 500
   }
   ```

4. Click **+ New Step** → search for **Add comment to incident** (Microsoft Sentinel action).
5. Configure:
   - **Incident ARM ID:** Use dynamic content → `See more` → select **Incident ARM ID**
   - **Comment:** Use dynamic content → `See more` → select **Body** (the HTTP response text)
6. Click **Save** (top-left corner).

#### 8c. Get Your OpenAI API Key

1. Go to [https://platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Click **+ Create new secret key** → name it `sentinel-playbook` → click **Create**.
3. **Copy the key immediately** — it won't be shown again.
4. Store it securely (ideally in **Azure Key Vault**).
5. Paste into the `Authorization` header as: `Bearer sk-...`

> 💡 **Customising the prompt:** You can tailor the GPT system prompt to your needs:
> - *Compliance:* Add `"Identify if personal data regulated under GDPR or DPDPA may be involved."`
> - *Cloud security:* Add `"List any affected Azure resources and their risk classification."`
> - *Threat hunting:* Add `"Suggest a KQL query to hunt for related activity in the last 7 days."`

---

### 9. Grant Sentinel Responder Role to the Logic App

The Logic App needs the **Microsoft Sentinel Responder** role to post comments on incidents.

1. Azure Portal → **Resource Groups** → `rg-sentinel-lab` → **Access control (IAM)**.
2. Click **+ Add** → **Add role assignment**.
3. **Role tab:** Search for and select `Microsoft Sentinel Responder` → **Next**.
4. **Members tab:**
   - Assign access to: **Managed Identity**
   - Click **+ Select members**
   - Managed identity type: **Logic App**
   - Select: `GPT-Incident-Responder`
   - Click **Select**
5. Click **Review + Assign** → **Assign**.

> ⚠️ Skipping this step causes a `403 Forbidden` error in the Logic App run history.

---

### 10. Create the Automation Rule

An Automation Rule wires everything together — it fires the GPT playbook whenever a new incident is created.

1. In Sentinel → **Automation** → **+ Create** → **Automation rule**.
2. Configure:
   - **Name:** `Trigger GPT Analysis on New Incident`
   - **Trigger:** `When incident is created`
   - **Conditions (optional):** `Severity is High or Medium` *(to limit GPT API calls)*
3. Under **Actions** → **+ Add action** → **Run playbook** → select `GPT-Incident-Responder`.
4. Set **Order:** `1`.
5. Click **Apply**.

---

### 11. Test the Full Pipeline

1. In Sentinel → **Incidents** → **+ Create incident**.
2. Fill in:
   - **Title:** `Test — Tor Login Detected`
   - **Severity:** `High`
3. Click **Save**.
4. After a few seconds, click the incident → scroll to **Comments**.
5. Verify that a GPT-generated analysis comment has been posted. ✅

**If no comment appears:** Open the Logic App → **Overview** → **Runs history** → inspect the failed run for error details.

---

## 💰 Cost Management & Free Tier Tips

| Service | Free Allowance | What to Do if Exceeded |
|---------|---------------|------------------------|
| Microsoft Sentinel | 10 GB/day (first 31 days) | Set a daily ingestion cap on the workspace |
| Log Analytics Workspace | 5 GB/month (after free trial) | Disable unused data connectors |
| Azure Logic Apps | 4,000 actions/month (Consumption) | Add severity conditions to limit playbook triggers |
| OpenAI GPT-3.5-Turbo | $5 credit (new accounts) | Set a hard monthly limit in the OpenAI dashboard |

### Key Actions to Avoid Charges

- **Set a daily ingestion cap:** Workspace → **Usage and estimated costs** → **Daily cap** → `10 GB`
- **Set a budget alert:** Azure Portal → **Cost Management + Billing** → **Budgets** → **+ Add** → set $0 threshold with email alerts at 80% and 100%
- **Disable unused connectors:** Sentinel → **Data connectors** → disconnect any source not used in a rule
- **Set an OpenAI spending limit:** [https://platform.openai.com/account/limits](https://platform.openai.com/account/limits) → hard limit of $5/month
- **Delete resources when done:** Resource Groups → `rg-sentinel-lab` → **Delete resource group** *(removes all billable resources)*

---

## 🔧 Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| Deployment shows `Failed` | Unlicensed data connector | Ignore — workspace is healthy. Proceed with setup. |
| No data in Logs | Connectors not configured | Sentinel → Data Connectors → follow each connector's configuration guide |
| Logic App fails with `403` | Missing Sentinel Responder role | Repeat Step 9 — verify the correct Logic App was selected |
| No GPT comment on incident | API key wrong or Logic App not triggered | Check Logic App run history; confirm `Bearer` prefix on API key |
| GPT response is empty | `max_tokens` too low | Increase to `800` in the HTTP body |
| UEBA data not showing | Baselines still building | Wait 24–48 hours after enabling UEBA |
| Unexpected Azure charges | Daily cap or budget alert not set | Set ingestion cap + budget alert (see Cost Management above) |

---

## 📖 Glossary

| Term | Definition |
|------|------------|
| **Analytics Rule** | A KQL-based detection query that generates Sentinel alerts on a schedule |
| **ARM Template** | Azure Resource Manager template — infrastructure-as-code for Azure deployments |
| **Automation Rule** | Sentinel feature that auto-triggers actions when an incident is created or updated |
| **Data Connector** | Integration that streams logs from an external source into Log Analytics |
| **UEBA** | User and Entity Behaviour Analytics — ML anomaly detection against baselines |
| **KQL** | Kusto Query Language — the query language for Azure Log Analytics |
| **Logic App** | Azure serverless workflow engine used to build automated playbooks |
| **Managed Identity** | An Azure AD identity managed by Azure — no stored passwords or secrets |
| **Playbook** | A Logic App workflow that automates a security response action |
| **SIEM** | Security Information and Event Management |
| **SOAR** | Security Orchestration, Automation, and Response |
| **Tor Exit Node** | Final relay IP in the Tor network — often used to anonymise attacker origin |
| **Watchlist** | A CSV-based lookup table queryable inside Sentinel KQL rules |

---

## 📁 Project Structure

```
📦 sentinel-ai-soc/
├── 📄 README.md                  ← You are here
├── 📂 watchlists/
│   └── tor-exit-nodes.csv        ← Tor exit node IP block list
├── 📂 analytics-rules/
│   └── tor-signin-detection.json ← Exported KQL analytics rule
├── 📂 playbooks/
│   └── gpt-incident-responder/   ← Logic App ARM template (optional export)
└── 📂 docs/
    └── Microsoft_Sentinel_SIEM_Documentation.docx
```

---

## 🤝 Credits & References

- [Azure Sentinel All-In-One](https://github.com/Azure/Azure-Sentinel/tree/master/Tools/Sentinel-All-In-One) — Microsoft
- [Microsoft Sentinel Documentation](https://learn.microsoft.com/en-us/azure/sentinel/)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [Tor Project — Exit Node List](https://check.torproject.org/exit-addresses)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

<div align="center">

**Built with ❤️ using Microsoft Azure · Microsoft Sentinel · OpenAI GPT**

*If this project helped you, please ⭐ star the repo!*

</div>
