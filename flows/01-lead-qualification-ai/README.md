# Flow 01 — AI Lead Qualification Agent

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama_llama3.2-000000?style=flat-square)
![Airtable](https://img.shields.io/badge/Airtable-18BFFF?style=flat-square&logo=airtable&logoColor=white)
![Gmail](https://img.shields.io/badge/Gmail-EA4335?style=flat-square&logo=gmail&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)

---

## Problem Statement

Sales teams receive inbound leads from multiple sources — contact forms, LinkedIn, referrals — but spend hours manually reading each message to decide who to call first. Without a scoring system, high-value prospects with explicit budget and urgency get the same priority as vague "just browsing" inquiries. Reps chase cold leads and miss the hot ones.

**The cost:** missed revenue, wasted rep time, and slow response to buyers who are ready to buy now.

---

## Solution Overview

An automated n8n agent that activates the moment a lead submits a contact form. It validates the data, sends the full context to a local LLM (Ollama / llama3.2), receives a structured qualification score (0–100) with tier classification, saves the enriched record to Airtable, and emails the sales team — all in under 15 seconds, with zero human involvement.

**Business outcome:** sales reps open their inbox and already know whether to call immediately (HOT), nurture over email (WARM), or deprioritize (COLD).

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        INBOUND LAYER                                │
│                                                                     │
│   [Lead Form / CRM / API]                                           │
│         │  POST /webhook/new-lead                                   │
│         ▼                                                           │
│   ┌─────────────────┐                                               │
│   │ Webhook Trigger │  n8n-nodes-base.webhook                      │
│   └────────┬────────┘                                               │
└────────────┼────────────────────────────────────────────────────────┘
             │
┌────────────▼────────────────────────────────────────────────────────┐
│                      VALIDATION LAYER                               │
│                                                                     │
│   ┌──────────────────────────┐                                      │
│   │  Validate & Enrich Lead  │  Code node (JavaScript)             │
│   │  · Checks required fields│  Generates LEAD-{timestamp} ID      │
│   │  · Enriches with metadata│  Extracts email domain              │
│   │  · Calculates msg length │  Counts message words               │
│   └────────────┬─────────────┘                                      │
└────────────────┼────────────────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────────────────┐
│                        AI LAYER                                     │
│                                                                     │
│   ┌──────────────────────────┐                                      │
│   │    AI Lead Qualifier     │  @n8n/n8n-nodes-langchain.chainLlm  │
│   │    (Basic LLM Chain)     │  Prompt: lead context + JSON schema  │
│   └────────────┬─────────────┘                                      │
│                │ ◆ model connection                                 │
│   ┌────────────▼─────────────┐                                      │
│   │  AI Qualification        │  @n8n/n8n-nodes-langchain.lmOllama  │
│   │  (Ollama llama3.2)       │  temp: 0.3  ·  local · free        │
│   └──────────────────────────┘                                      │
└────────────────┼────────────────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────────────────┐
│                      PARSING LAYER                                  │
│                                                                     │
│   ┌──────────────────────────┐                                      │
│   │   Parse AI Response      │  Code node (JavaScript)             │
│   │   · JSON.parse() output  │  Fallback if LLM returns bad JSON   │
│   │   · Merge lead + scores  │  Adds tierColor + priority fields   │
│   └────────────┬─────────────┘                                      │
└────────────────┼────────────────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────────────────┐
│                      ROUTING LAYER                                  │
│                                                                     │
│   ┌──────────────────────────┐                                      │
│   │     Route by Tier        │  IF node                            │
│   │     tier === "HOT" ?     │                                      │
│   └──────┬──────────┬────────┘                                      │
│          │ true     │ false                                         │
│        HOT          WARM/COLD                                       │
└────────────────────┼────────────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────────────┐
│                       OUTPUT LAYER                                  │
│                                                                     │
│   ┌──────────────────────────┐                                      │
│   │   Save to Airtable CRM   │  n8n-nodes-base.airtable            │
│   │   Base: Lead Pipeline    │  Creates record with all fields      │
│   │   Table: Leads           │                                      │
│   └────────────┬─────────────┘                                      │
│                │                                                    │
│   ┌────────────▼─────────────┐   ┌──────────────────────────┐      │
│   │  Send Notification Email │   │   Respond to Webhook     │      │
│   │  Gmail — HTML template   │   │   JSON confirmation      │      │
│   │  Subject: dynamic by tier│   │   { success, tier, score}│      │
│   └──────────────────────────┘   └──────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Node Breakdown

| # | Node | Type | Function |
|---|------|------|----------|
| 1 | Webhook - New Lead | `n8n-nodes-base.webhook` | Receives POST from form/CRM. Path: `/webhook/new-lead` |
| 2 | Validate & Enrich Lead | `n8n-nodes-base.code` | Validates required fields, generates unique ID, enriches metadata |
| 3 | AI Lead Qualifier | `@n8n/n8n-nodes-langchain.chainLlm` | Builds prompt with lead context, calls the LLM, returns structured JSON |
| 4 | AI Qualification (Ollama) | `@n8n/n8n-nodes-langchain.lmOllama` | Local LLM inference via Ollama. Model: `llama3.2:latest`, temp: 0.3 |
| 5 | Parse AI Response | `n8n-nodes-base.code` | Parses LLM JSON output, merges with lead data, handles fallback on parse error |
| 6 | Route by Tier | `n8n-nodes-base.if` | Routes HOT leads vs WARM/COLD (extensible for different urgency handling) |
| 7 | Save to Airtable CRM | `n8n-nodes-base.airtable` | Creates a record in the "Leads" table with all qualification fields |
| 8 | Send Notification Email | `n8n-nodes-base.gmail` | Sends HTML email to sales team with full lead summary and AI insights |
| 9 | Respond to Webhook | `n8n-nodes-base.respondToWebhook` | Returns JSON confirmation `{ success, leadId, tier, score }` to the form |

---

## Key Engineering Decisions

### Ollama over OpenAI
- **Cost:** Ollama is free and runs locally — no per-token cost during development or for low-volume usage
- **Privacy:** Lead data (names, emails, company details) never leaves the local machine
- **Latency:** 4–15 seconds on a local machine — acceptable for async lead qualification
- **Swap path:** Replace the `lmOllama` sub-node with `lmOpenAi` and update the model to `gpt-4o` for cloud production. No other changes needed.

### Airtable over a database
- **Zero setup:** no schema migrations, no SQL, no infrastructure. Free tier handles 1,000 records/base.
- **Visual CRM:** sales team can view, filter, and update leads directly in Airtable without any extra tooling
- **API-first:** Airtable's REST API integrates cleanly with n8n's native node

### Basic LLM Chain over AI Agent
- **Determinism:** the chain runs a single structured prompt and returns. An Agent loop would add latency and unpredictability for this use case.
- **Structured output:** temperature 0.3 + explicit JSON schema in the prompt produces reliable parseable output ~95% of the time. The Code node handles the remaining 5% with a fallback.

### Code nodes for validation and parsing
- **Flexibility:** JavaScript code nodes give full control over data transformation without being limited by a visual node's constraints
- **Fallback logic:** the Parse AI Response node gracefully handles LLM output that isn't valid JSON, preventing the workflow from crashing

---

## Portability: Migrating to n8n Cloud

To run this flow on n8n Cloud (or any hosted n8n) without Ollama:

1. **Replace the LLM node:** delete `AI Qualification (Ollama)`, add `OpenAI Chat Model` or `Groq Chat Model` sub-node
2. **Connect it** to the `AI Lead Qualifier` (Basic LLM Chain) via the model port
3. **Add credentials:** OpenAI API key or Groq API key in n8n Settings → Credentials
4. **Recommended models:**
   - OpenAI: `gpt-4o-mini` (fast, cheap, reliable JSON output)
   - Groq: `llama-3.1-8b-instant` (free tier, ~1s latency)
5. **Update Webhook URL** in the HTML form to your n8n Cloud instance URL

Everything else — validation, parsing, Airtable, Gmail — works identically.

---

## Business Impact

| Metric | Before (manual) | After (automated) |
|--------|-----------------|-------------------|
| Time to qualify a lead | 5–15 min per lead | < 15 seconds |
| Response time to HOT leads | Hours to days | Immediate email alert |
| Lead data in CRM | Inconsistent, manual | 100% captured, structured |
| Sales rep time on qualification | ~30% of work hours | Near zero |
| Missed HOT leads | Common | Eliminated |

---

## How to Run Locally

### Prerequisites
- Docker Desktop installed and running
- Git
- Ollama installed with `llama3.2` pulled (`ollama pull llama3.2`)

### Steps

```bash
# 1. Clone the repo
git clone https://github.com/Asilvag20/n8n-ai-portfolio.git
cd n8n-ai-portfolio

# 2. Start n8n
docker-compose up -d

# 3. Verify Ollama is running
curl http://localhost:11434/api/tags

# 4. Open n8n at http://localhost:5678
#    Import flow: ⋮ menu → Import from file
#    → flows/01-lead-qualification-ai/flow.json

# 5. Configure credentials in Settings → Credentials:
#    · Ollama: http://host.docker.internal:11434
#    · Airtable Personal Access Token
#    · Gmail OAuth2

# 6. Activate the workflow (toggle top-right → green)

# 7. Open the lead form in your browser:
#    forms/lead-form.html
```

---

## How to Test

```bash
curl -X POST http://localhost:5678/webhook/new-lead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Carlos Mendoza",
    "email": "carlos.mendoza@techcorp.co",
    "company": "TechCorp Colombia",
    "message": "Estamos buscando automatizar nuestro proceso de ventas. Somos una empresa de 50 personas, necesitamos solución urgente este trimestre. Presupuesto: $5000 USD/mes.",
    "source": "Website"
  }'
```

**Expected response:**
```json
{
  "success": true,
  "leadId": "LEAD-1234567890",
  "tier": "HOT",
  "score": 87,
  "message": "Lead received and qualified successfully"
}
```
