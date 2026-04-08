# AI Automation Engineer Portfolio

**Cristian Andrés Silva Grisales** — AI Automation Engineer

[![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docker.com)
[![Ollama](https://img.shields.io/badge/Ollama-000000?style=for-the-badge&logo=ollama&logoColor=white)](https://ollama.ai)
[![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Asilvag20)

---

## Overview

A curated collection of production-ready **AI-powered automation workflows** built with n8n. Each flow solves a real business problem using modern AI integrations — from local LLMs via Ollama to cloud APIs — demonstrating end-to-end automation design, RAG pipelines, and intelligent data processing.

> Live demo & documentation: **[your-domain.com](#)** *(coming soon)*

---

## Workflow Catalog

| # | Flow | Description | Tech Stack |
|---|------|-------------|------------|
| 01 | [Lead Qualification AI](./flows/01-lead-qualification-ai/) | Scores and qualifies inbound leads automatically using LLM analysis | n8n · OpenAI · Airtable · Slack |
| 02 | [Sales Report Automation](./flows/02-sales-report-automation/) | Generates weekly AI-narrated sales reports from CRM data | n8n · Ollama · Google Sheets · Gmail |
| 03 | [Competitor Monitor AI](./flows/03-competitor-monitor-ai/) | Monitors competitor websites and summarizes changes with AI | n8n · Ollama · RSS · Telegram |
| 04 | [Support Bot RAG](./flows/04-support-bot-rag/) | Customer support chatbot with Retrieval-Augmented Generation | n8n · Ollama · Pinecone · WhatsApp |
| 05 | [Client Onboarding](./flows/05-client-onboarding/) | Automates the full client onboarding sequence end-to-end | n8n · OpenAI · Notion · HubSpot |

---

## Tech Stack

### Automation & Orchestration
[![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)](https://n8n.io)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docker.com)

### AI / LLM
[![OpenAI](https://img.shields.io/badge/OpenAI_GPT--4-412991?style=flat-square&logo=openai&logoColor=white)](https://openai.com)
[![Ollama](https://img.shields.io/badge/Ollama_Llama3-000000?style=flat-square)](https://ollama.ai)

### Data & Storage
[![Airtable](https://img.shields.io/badge/Airtable-18BFFF?style=flat-square&logo=airtable&logoColor=white)](https://airtable.com)
[![Notion](https://img.shields.io/badge/Notion-000000?style=flat-square&logo=notion&logoColor=white)](https://notion.so)
[![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=flat-square&logo=googlesheets&logoColor=white)](https://sheets.google.com)

### Integrations
[![Slack](https://img.shields.io/badge/Slack-4A154B?style=flat-square&logo=slack&logoColor=white)](https://slack.com)
[![Telegram](https://img.shields.io/badge/Telegram-26A5E4?style=flat-square&logo=telegram&logoColor=white)](https://telegram.org)
[![HubSpot](https://img.shields.io/badge/HubSpot-FF7A59?style=flat-square&logo=hubspot&logoColor=white)](https://hubspot.com)

---

## About Me

I'm an AI Automation Engineer specializing in building intelligent workflows that connect APIs, AI models, and business tools. I design systems that eliminate repetitive work, reduce response times, and create data-driven insights — all without writing boilerplate infrastructure code.

- **Focus areas:** AI agents, RAG pipelines, CRM automation, report generation
- **Tools I work with:** n8n, Make, Zapier, LangChain, OpenAI, Ollama
- **Available for:** Freelance projects, consulting, full-time remote roles

**Contact:** [GitHub @Asilvag20](https://github.com/Asilvag20) · [your-email@domain.com](#)

---

## Getting Started

### Prerequisites
- Docker & Docker Compose
- Git

### Run locally

```bash
git clone https://github.com/Asilvag20/n8n-ai-portfolio.git
cd n8n-ai-portfolio
docker-compose up -d
```

Open [http://localhost:5678](http://localhost:5678)
- **User:** `admin`
- **Password:** `portfolio2024`

### Run with local LLMs (Ollama)

```bash
# Install Ollama from https://ollama.ai
ollama pull llama3.2
ollama serve
```

---

## Repository Structure

```
n8n-portfolio/
├── flows/                    # Workflow definitions (JSON + docs)
│   ├── 01-lead-qualification-ai/
│   ├── 02-sales-report-automation/
│   ├── 03-competitor-monitor-ai/
│   ├── 04-support-bot-rag/
│   └── 05-client-onboarding/
├── docs/                     # Setup and deployment guides
├── docker-compose.yml        # n8n local environment
└── README.md
```

---

*Built with n8n · Powered by AI · Made in Colombia*
