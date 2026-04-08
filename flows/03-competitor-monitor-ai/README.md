# Flow 03 — Competitor Monitor AI

## Problem Statement

Keeping track of competitor moves — pricing changes, new features, blog posts, job listings — requires daily manual checking across dozens of sources. Most companies miss critical competitive signals entirely.

## Solution Overview

An automated intelligence workflow that monitors competitor websites, RSS feeds, and social channels daily. It uses AI to summarize changes, filter noise, and deliver only the relevant insights to your team via Telegram or Slack.

## Architecture Diagram

```
[Cron: Daily 9AM]
      |
      v
[HTTP Request x N] --> Scrape competitor URLs
      |
      v
[RSS Feed Nodes] --> Pull competitor blogs / news
      |
      v
[Code Node] --> Diff against previous content (hash comparison)
      |
      v
[IF: Changes found?]
      |
      v
[Ollama llama3.2] --> Summarize & classify changes
      |                (pricing / feature / hiring / content)
      v
[Telegram / Slack] --> Send daily intelligence digest
      |
      v
[Airtable] --> Log all detected changes with timestamps
```

## Nodes Used

| Node | Purpose |
|------|---------|
| Schedule Trigger | Daily morning run |
| HTTP Request | Scrape target pages |
| RSS Feed | Monitor competitor blogs |
| Code (JS) | Hash-based diff detection |
| IF | Skip when no changes |
| Ollama Chat | Classify and summarize |
| Telegram | Send digest to team |
| Airtable | Persistent change log |

## How to Run Locally

1. Start n8n and Ollama: `docker-compose up -d && ollama serve`
2. Import `flow.json`
3. Edit the competitor URLs array in the Code node
4. Set your Telegram bot token and chat ID
5. Configure Airtable for change logging
6. Activate — first run builds the baseline, subsequent runs detect diffs

## Business Impact

- **Coverage:** Monitor 20+ competitor sources simultaneously
- **Signal-to-noise:** AI filters noise, only alerts on meaningful changes
- **Response time:** React to competitor moves same day
- **Cost:** Eliminates need for expensive competitive intelligence tools ($500+/mo)
