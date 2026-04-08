# Flow 01 — Lead Qualification AI

## Problem Statement

Sales teams waste hours manually reviewing inbound leads, guessing which ones are worth pursuing. Without a scoring system, high-value prospects get missed and reps chase cold leads.

## Solution Overview

An automated n8n workflow that receives inbound leads (via webhook, form, or CRM), analyzes them with an LLM, assigns a qualification score (0–100), and routes them to the right sales channel — all in under 30 seconds.

## Architecture Diagram

```
[Webhook / Form]
      |
      v
[HTTP Request] --> Enrich lead data (Clearbit/Hunter)
      |
      v
[OpenAI / Ollama] --> Score lead (0-100) + generate summary
      |
      v
[IF Node] ---> Score >= 70 --> [Airtable] + [Slack #hot-leads]
          \--> Score 40-69 --> [Airtable] + [Email nurture]
           \-> Score < 40  --> [Airtable] + [Archive]
```

## Nodes Used

| Node | Purpose |
|------|---------|
| Webhook | Receive inbound lead data |
| HTTP Request | Enrich lead with external data |
| OpenAI / Ollama Chat | Score and analyze the lead |
| IF | Route by score tier |
| Airtable | Log all leads with score |
| Slack | Alert sales on hot leads |
| Send Email | Trigger nurture sequences |

## How to Run Locally

1. Start n8n: `docker-compose up -d`
2. Import `flow.json` in the n8n UI (Settings → Import)
3. Configure credentials: OpenAI API key or Ollama endpoint
4. Set your Airtable base ID and table name
5. Set your Slack webhook URL
6. Activate the workflow
7. Test with: `curl -X POST http://localhost:5678/webhook/lead -d '{"name":"John","email":"john@company.com","company":"Acme Inc"}'`

## Business Impact

- **Time saved:** ~4 hours/week per sales rep
- **Lead response time:** Reduced from 24h to < 1 minute
- **Conversion rate improvement:** 15–30% by focusing reps on qualified leads
- **Scalable:** Handles unlimited inbound volume without additional headcount
