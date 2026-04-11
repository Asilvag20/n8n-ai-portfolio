# Flow 03 — Competitor Monitor AI

![n8n](https://img.shields.io/badge/n8n-workflow-orange) ![Ollama](https://img.shields.io/badge/LLM-Ollama_llama3.2-blue) ![Google Sheets](https://img.shields.io/badge/Storage-Google_Sheets-green) ![Telegram](https://img.shields.io/badge/Alert-Telegram-2CA5E0) ![Gmail](https://img.shields.io/badge/Report-Gmail_HTML-red)

Automated competitive intelligence agent that monitors competitor websites every 6 hours, detects changes using AI, sends instant Telegram alerts for critical changes, and delivers a weekly strategic report every Monday at 8 AM.

---

## Problem Statement

Manually tracking competitor websites is time-consuming and inconsistent. Price changes, new service launches, and messaging shifts often go unnoticed until they impact your business. This flow automates continuous monitoring with AI-powered classification, so you act on competitive intelligence before it's too late.

---

## Architecture

```
MONITORING LOOP (every 6h)
Every 6 Hours
       │
       ▼
Read Competitors ──── Google Sheets "competitors" tab
       │               Reads active URLs to monitor
       ▼
SplitInBatches ──── Processes 1 competitor at a time
       │
       ▼
Fetch Competitor Site ──── HTTP GET with realistic User-Agent
       │                    continueOnFail: true (timeout 10s)
       ▼
Clean & Extract ──── Code Node
       │              Strips scripts/styles/nav/footer
       │              Extracts prices, H1-H3 titles
       │              Calculates MD5 hash of cleaned text
       ▼
Get Stored Data ──── Code Node
       │              Detects first run vs subsequent runs
       ▼
Compare Content ──── Code Node
       │              Hash comparison → generates diff for AI
       ▼
Change Detected? ──── IF Node
       │
   ┌───┴──────────────────────┐
  YES                         NO
   │                           │
   ▼                           ▼
AI Change Analysis         Update No Change
(Basic LLM Chain)          (Sheets: last_checked)
   ▲
   └── Ollama llama3.2
       temperature: 0.2
   │
   ▼
Parse AI Response ──── Code Node (JSON parse + fallback)
   │
   ▼
Route by Criticality ──── Switch Node
   │
   ├── CRITICAL ──► Telegram Alert + Log + Update Competitor
   ├── MODERATE ──► Log + Update Competitor
   └── LOW      ──► Log + Update Competitor

WEEKLY REPORT (Monday 8AM)
Every Monday 8AM
       │
       ▼
Read Change Log ──── Google Sheets "change_log" tab
       │
       ▼
Build Weekly Context ──── Code Node (last 7 days aggregation)
       │
       ▼
Weekly Summary ──── Basic LLM Chain + Ollama llama3.2
       │             temperature: 0.3
       ▼
Build Weekly HTML ──── Code Node (responsive email)
       │
       ▼
Send Weekly Report ──── Gmail OAuth2
```

---

## Nodes

| Node | Type | Function |
|------|------|----------|
| **Every 6 Hours** | Schedule Trigger | Monitoring loop trigger |
| **Every Monday 8AM** | Schedule Trigger | Weekly report trigger |
| **Read Competitors** | Google Sheets | Reads active competitor URLs from `competitors` tab |
| **SplitInBatches** | Split In Batches | Processes 1 competitor per iteration |
| **Fetch Competitor Site** | HTTP Request | GET with Chrome User-Agent, 10s timeout, continueOnFail |
| **Clean & Extract** | Code | Strips HTML noise, extracts prices + titles, MD5 hash |
| **Get Stored Data** | Code | Validates stored content, detects first-run baseline |
| **Compare Content** | Code | Hash diff → builds change context for AI |
| **Change Detected?** | IF | Routes changed vs unchanged content |
| **AI Change Analysis** | Basic LLM Chain | Classifies change: criticality, type, impact |
| **Ollama Analysis** | LM Ollama | llama3.2, temp 0.2 for consistent JSON output |
| **Parse AI Response** | Code | Extracts JSON from Ollama, normalizes with fallback |
| **Route by Criticality** | Switch | CRITICAL / MODERATE / LOW routing |
| **Telegram Critical Alert** | Telegram | Instant Markdown message for CRITICAL changes only |
| **Log Change** | Google Sheets | Appends to `change_log` tab |
| **Update Competitor** | Google Sheets | Updates `last_content`, `last_checked` |
| **Update No Change** | Google Sheets | Updates `last_checked` on no-change iterations |
| **Read Change Log** | Google Sheets | Reads full `change_log` for weekly aggregation |
| **Build Weekly Context** | Code | Filters last 7 days, groups by competitor |
| **Weekly Summary** | Basic LLM Chain | Generates strategic executive report |
| **Ollama Weekly** | LM Ollama | llama3.2, temp 0.3 |
| **Build Weekly HTML** | Code | Responsive HTML email with KPI cards |
| **Send Weekly Report** | Gmail | Sends weekly report email |

---

## Engineering Decisions

### Why Google Sheets as storage instead of a database?
- **Free**: No database hosting cost
- **Auditable**: Anyone on the team can open the sheet and see the full change history
- **Editable**: Add or disable competitors by changing a cell — no code changes
- **Portable**: The flow works without setting up any external infrastructure beyond Google OAuth

### Why two separate Schedule Triggers?
The monitoring loop (every 6h) and the weekly report (Monday 8AM) serve different purposes:
- The 6h loop detects and alerts — it must run frequently and only alert when something changes
- The Monday report summarizes — it runs once, reads the full week's log, and delivers strategic context

Combining them in one flow would require complex branching logic. Separate triggers keeps each path clean and independently configurable.

### Why `continueOnFail: true` on HTTP Request?
Some competitor sites may be temporarily down, return 403, or timeout. Without `continueOnFail`, a single failing request would stop the entire batch. With it, the flow logs the error and moves to the next competitor.

### Hash-based change detection
Instead of storing the full HTML diff (expensive), the flow:
1. Cleans the HTML to remove ephemeral noise (dates, session tokens, ads)
2. Computes MD5 of the cleaned text
3. Compares hashes — only calls Ollama when content actually changed

This keeps the Ollama call rate low (only on real changes) and avoids LLM token waste.

### First-run baseline
On the first execution for a new competitor, `last_content` is empty. The flow detects this (`isFirstRun: true`) and stores the current content as baseline without triggering an alert — avoiding false positives on first setup.

---

## Setup

### Google Sheets structure

**Sheet: `competitors`**

| competitor_name | url | section_to_monitor | last_content | last_checked | status |
|---|---|---|---|---|---|
| Competitor A | https://... | pricing page | *(auto-filled)* | *(auto-filled)* | active |

**Sheet: `change_log`** (auto-filled by the flow)

| date | competitor | change_type | criticality | summary | notified |
|---|---|---|---|---|---|

### How to add a competitor
Just add a new row in the `competitors` sheet:
1. Fill `competitor_name`, `url`, `section_to_monitor`
2. Leave `last_content`, `last_checked` empty
3. Set `status` to `active`

The flow will establish a baseline on the next 6h cycle and start monitoring automatically.

### How to disable a competitor
Change `status` to `inactive` in the `competitors` sheet. No flow edits needed.

### Credentials required

| Credential | Type | Scopes |
|---|---|---|
| Google Sheets account | OAuth2 | `spreadsheets` |
| Gmail account | OAuth2 | `gmail.send` |
| Ollama account | API | `http://host.docker.internal:11434` |
| Telegram Bot | API Token | Bot token from BotFather |

### Telegram Bot setup (5 minutes, free)
See the **Telegram Setup** section below.

---

## Telegram Setup

### 1. Create the bot
1. Open Telegram and search for **@BotFather**
2. Send `/newbot`
3. Choose a name: e.g. `Competitor Monitor`
4. Choose a username: e.g. `competitor_monitor_yourname_bot`
5. BotFather gives you a **token**: `123456789:ABCdef...` — copy it

### 2. Create the credential in n8n
1. Go to **Settings → Credentials → New**
2. Type: **Telegram API**
3. Paste the token
4. Save as `Telegram Bot`

### 3. Get your Chat ID
1. Search for **@userinfobot** in Telegram
2. Send `/start`
3. It replies with your Chat ID, e.g. `123456789`

Alternatively:
1. Send any message to your bot
2. Open: `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
3. Find `"chat":{"id":XXXXXX}` in the response

### 4. Update the flow
In the **Telegram Critical Alert** node, replace `YOUR_CHAT_ID` with your actual Chat ID.

---

## How to run locally

```bash
# 1. Start n8n
docker compose up -d

# 2. Import the flow
docker cp flows/03-competitor-monitor-ai/flow.json n8n-portfolio:/tmp/flow03.json
docker exec n8n-portfolio n8n import:workflow --input=/tmp/flow03.json

# 3. In n8n UI (http://localhost:5678)
#    - Connect Google Sheets credential on: Read Competitors, Log Change,
#      Update Competitor, Update No Change, Read Change Log
#    - Connect Telegram credential on: Telegram Critical Alert
#    - Update YOUR_CHAT_ID in the Telegram node
#    - Activate the flow

# 4. Manual test (monitoring loop)
#    Click "Test workflow" on the "Every 6 Hours" trigger

# 5. Manual test (weekly report)
#    Click "Test workflow" on the "Every Monday 8AM" trigger
```

---

## Scaling

| Scenario | Action |
|---|---|
| Add a competitor | Add row to `competitors` sheet — no code changes |
| Monitor more sections | Add rows with same URL but different `section_to_monitor` value |
| Increase frequency | Change Schedule Trigger from 6h to 1h or 3h |
| Add Slack alerts | Duplicate the Telegram node, connect to CRITICAL output |
| Add more recipients | Duplicate the Gmail node or add CC in the options |

---

## Business Impact

- **Coverage**: 24/7 monitoring without manual effort
- **Speed**: Critical alerts in minutes, not days
- **AI cost**: $0 — Ollama runs locally
- **Storage cost**: $0 — Google Sheets free tier
- **Scalability**: Add unlimited competitors by adding rows to a spreadsheet
