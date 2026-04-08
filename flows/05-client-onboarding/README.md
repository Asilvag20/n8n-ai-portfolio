# Flow 05 — Client Onboarding Automation

## Problem Statement

New client onboarding involves 15–20 repetitive tasks: sending welcome emails, creating project spaces, setting up accounts, scheduling kickoff calls, and collecting documents. Done manually, it takes days and mistakes cause poor first impressions.

## Solution Overview

A comprehensive onboarding orchestrator triggered when a deal closes in the CRM. It automates every step of the onboarding sequence — from welcome email to kickoff scheduling — delivering a consistent, professional experience in minutes.

## Architecture Diagram

```
[HubSpot Trigger] --> Deal marked "Closed Won"
      |
      v
[HubSpot] --> Fetch full client + deal data
      |
      v
[OpenAI] --> Generate personalized welcome message
      |
      v
[Parallel branches]
      |--- [Gmail] Send welcome email with onboarding guide
      |--- [Notion] Create client workspace from template
      |--- [Calendly API] Generate kickoff scheduling link
      |--- [HubSpot] Create onboarding tasks + timeline
      |--- [Slack] Notify internal team #new-clients
      |
      v
[Wait Node: 24h]
      |
      v
[IF: Documents received?]
      |--- Yes --> [Gmail] Confirm receipt + next steps
      |--- No  --> [Gmail] Gentle reminder
      |
      v
[HubSpot] --> Update deal stage to "Onboarding Active"
```

## Nodes Used

| Node | Purpose |
|------|---------|
| HubSpot Trigger | Fire on deal close |
| HubSpot | Read/write CRM data |
| OpenAI | Personalize messages |
| Gmail | Send emails |
| Notion | Create project workspace |
| Calendly (HTTP) | Generate scheduling links |
| Slack | Internal team notification |
| Wait | Time-delay between steps |
| IF | Conditional follow-up logic |

## How to Run Locally

1. Start n8n: `docker-compose up -d`
2. Import `flow.json`
3. Configure HubSpot OAuth2 credentials
4. Set OpenAI API key
5. Connect Gmail OAuth2
6. Configure Notion integration token and template page ID
7. Set Slack incoming webhook
8. Test with a dummy HubSpot deal or use the manual trigger

## Business Impact

- **Onboarding time:** Reduced from 2–3 days to < 5 minutes
- **Consistency:** Every client gets the same quality experience
- **Team hours saved:** 4–6 hours per new client
- **Client satisfaction:** Professional, immediate response after signing
- **Scalability:** Works identically for 1 or 100 new clients per week
