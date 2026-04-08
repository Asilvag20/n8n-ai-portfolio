# Flow 02 — Sales Report Automation

## Problem Statement

Sales managers spend Friday afternoons manually pulling data from spreadsheets and CRMs to compile weekly reports. The process is error-prone, time-consuming, and the reports often lack actionable narrative.

## Solution Overview

A scheduled workflow that pulls sales data every Monday morning, processes it through an LLM for AI-generated insights and narrative, and delivers a polished report to stakeholders via email — automatically.

## Architecture Diagram

```
[Cron: Every Monday 8AM]
      |
      v
[Google Sheets] --> Fetch weekly sales data
      |
      v
[Code Node] --> Calculate KPIs (total, growth %, top deals)
      |
      v
[Ollama llama3.2] --> Generate executive narrative summary
      |
      v
[HTML Template] --> Build formatted email report
      |
      v
[Gmail / SMTP] --> Send to sales team + management
      |
      v
[Google Sheets] --> Log report history
```

## Nodes Used

| Node | Purpose |
|------|---------|
| Schedule Trigger | Run every Monday at 8 AM |
| Google Sheets | Pull raw sales data |
| Code (JS) | Calculate KPIs and metrics |
| Ollama Chat | Generate narrative insights |
| HTML node | Format the report |
| Gmail | Deliver to stakeholders |

## How to Run Locally

1. Start n8n and Ollama: `docker-compose up -d && ollama serve`
2. Import `flow.json` in the n8n UI
3. Connect Google Sheets credentials (OAuth2)
4. Configure Gmail SMTP or OAuth2
5. Set your spreadsheet ID and sheet name
6. Test manually: trigger the workflow from n8n UI
7. Activate for automatic Monday delivery

## Business Impact

- **Time saved:** 3–5 hours/week for sales managers
- **Consistency:** Same quality report every week, no human error
- **Speed:** Report ready in < 2 minutes after week close
- **Insight depth:** AI identifies trends humans might miss
