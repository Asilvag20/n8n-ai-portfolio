# Setup Guide

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Docker Desktop | 4.x+ | [docker.com/products/docker-desktop](https://docker.com/products/docker-desktop) |
| Git | 2.x+ | [git-scm.com](https://git-scm.com) |
| Ollama | latest | [ollama.ai](https://ollama.ai) |

## 1. Clone the repository

```bash
git clone https://github.com/Asilvag20/n8n-ai-portfolio.git
cd n8n-ai-portfolio
```

## 2. Start n8n

```bash
docker-compose up -d
```

Access n8n at [http://localhost:5678](http://localhost:5678)
- **Username:** `admin`
- **Password:** `portfolio2024`

> **Security note:** Change these credentials before exposing to any network.

## 3. Install Ollama (local LLMs)

Download and install from [ollama.ai](https://ollama.ai/download/windows)

Then pull the required model:

```bash
ollama pull llama3.2
```

Verify Ollama is running:

```bash
curl http://localhost:11434/api/tags
```

## 4. Configure n8n credentials

In the n8n UI, go to **Settings → Credentials** and add:

- **OpenAI:** Add your API key (for cloud LLM flows)
- **Ollama:** Set URL to `http://host.docker.internal:11434` (from inside Docker)
- **Google Sheets / Gmail:** OAuth2 flow via Google Cloud Console
- **Airtable:** Personal Access Token from airtable.com/account
- **Slack:** Incoming Webhook URL from api.slack.com
- **Telegram:** Bot token from @BotFather

## 5. Import flows

1. Open n8n UI
2. Go to **Workflows → Import from file**
3. Select the `flow.json` from any flow folder

## Stopping n8n

```bash
docker-compose down
```

Your data persists in the `n8n_data` Docker volume.

## Reset everything

```bash
docker-compose down -v   # WARNING: deletes all data
docker-compose up -d
```
