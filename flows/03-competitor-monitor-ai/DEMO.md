# DEMO Guide — Flow 03: Competitor Monitor AI

Demo guide for recording. Target time: **90–120 seconds**.

---

## Script (timestamps)

### [0:00 – 0:10] Problem context
Show the Google Sheet with the `competitors` tab open — 3 active rows with URLs.
> *"Manually tracking competitor websites takes hours per week. This flow does it automatically, every 6 hours, with AI."*

### [0:10 – 0:25] The flow in n8n
Open n8n (`http://localhost:5678`) and show the canvas.
> *"Two triggers: every 6 hours for monitoring, every Monday 8 AM for the weekly report."*

Walk through the monitoring path:
Trigger → Read Competitors → Filter Active → Fetch → Clean & Compare → Change Detected? → AI → Route → Sheets / Telegram

### [0:25 – 0:55] Live execution
Click **"Test workflow"** on the `Every 6 Hours` trigger.

Show each node output:
- **Read Competitors** → 3 items from the sheet
- **Fetch Competitor Site** → 3 HTTP responses
- **Clean & Compare** → hashes compared, `changeDetected: true/false`
- **Change Detected? TRUE** → items going to AI Analysis
- **AI Change Analysis** → Ollama output: `criticality`, `summary`, `recommended_action`
- **Route by Criticality** → CRITICAL / MODERATE / LOW routing
- **Log Change** → appended row in `change_log` tab
- **Update Competitor** → `last_content` and `last_checked` updated

### [0:55 – 1:15] Weekly report
Click **"Test workflow"** on `Every Monday 8AM`.

Show:
- **Read Change Log** → all logged changes
- **Build Weekly Context** → aggregated KPIs: total, critical, moderate, low
- **Weekly Summary** → Ollama strategic narrative
- **Send Weekly Report** → `messageId` confirming send

Switch to inbox and show the HTML email:
- Dark gradient header with week number and date range
- 4 KPI cards: Total · Critical · Moderate · Low
- AI strategic analysis section

### [1:15 – 1:30] Close
Back to canvas.
> *"24/7 competitive intelligence. Instant Telegram alerts for critical changes. Weekly strategic report every Monday. AI cost: zero — Ollama runs locally."*

---

## Test data

Make sure the `competitors` sheet has at least 3 rows:

```
competitor_name | url                        | section_to_monitor | last_content | last_checked | status
Competitor A    | https://example.com        | pricing            |              |              | active
Competitor B    | https://example.org        | homepage           |              |              | active
Competitor C    | https://example.net        | services           |              |              | active
```

Leave `last_content` empty on first run — the flow will establish the baseline automatically.

On second run, manually edit any cell in a competitor's page content via devtools or use a real competitor URL to see a genuine change detected.

---

## Recording tips

**Free tools (Windows):**
- **Xbox Game Bar** (`Win + G`) — instant, no install
- **OBS Studio** — full control, ideal for zooming into specific nodes

**Recommended settings:**
- Resolution: 1920x1080
- n8n zoom: 75–80% to see the full monitoring path
- Split screen: n8n canvas on left, Google Sheet on right

**Before recording:**
1. Clear `last_content` in the sheet so first run triggers baseline logging
2. Verify Ollama is running: `docker ps` → check ollama container
3. Have inbox open in another tab
4. Do a dry run to confirm everything works end-to-end

**Editing:**
- Speed up Ollama wait (5–15s) to 2x
- Add subtitle overlay showing criticality classification
- Optional: show Telegram alert on phone while n8n runs

---

## Key points to highlight

1. **No SplitInBatches needed** — n8n processes all competitors natively in one pass
2. **Hash-based detection** — only calls Ollama when content actually changed (zero wasted LLM calls)
3. **First-run baseline** — no false positives on initial setup
4. **Two independent triggers** — monitoring loop and weekly report run separately, clean paths
5. **AI cost: $0** — Ollama runs locally, no API fees regardless of how many competitors
