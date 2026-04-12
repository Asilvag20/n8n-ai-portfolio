# Flow 04 — AI Support Bot with RAG — Restaurant Use Case

![n8n](https://img.shields.io/badge/n8n-workflow-orange) ![Ollama](https://img.shields.io/badge/LLM-Ollama_llama3.2-blue) ![Pinecone](https://img.shields.io/badge/Vector_DB-Pinecone-green) ![Telegram](https://img.shields.io/badge/Channel-Telegram-2CA5E0) ![Gmail](https://img.shields.io/badge/Alerts-Gmail-red) ![OCR](https://img.shields.io/badge/OCR-Google_Drive-yellow)

AI-powered customer support bot for "La Fogata", a Colombian restaurant. Handles menu queries via RAG, routes orders to the kitchen Telegram group, escalates complaints to the owner, and lets the owner update the menu by simply sending a PDF to Telegram.

---

## Business Problem

Restaurant owners spend hours answering the same questions on WhatsApp/Telegram:
- "¿Cuánto vale el ajiaco?"
- "¿A qué hora abren los domingos?"
- "Mesa para 4 el sábado"
- "La comida llegó fría"

This flow automates 80% of those interactions, routes the rest to the right person, and delivers a weekly sentiment report — all with zero AI API cost.

---

## Two-Flow Architecture

### Flow A — Menu Indexer (`indexer-flow.json`)
Owner sends a scanned PDF via Telegram → OCR via Google Drive → chunked → embedded with `nomic-embed-text` → stored in Pinecone. Zero manual steps to update the menu.

### Flow B — Support Bot + Weekly Report (`bot-flow.json`)
Receives client messages, classifies intent, routes to the right handler. RAG retrieves relevant menu context from Pinecone before every AI response.

---

## PDF Ingestion Pipeline (OCR via Google Drive)

```
Owner sends PDF via Telegram
       │
       ▼
Telegram getFile → Download binary
       │
       ▼
Upload to Google Drive (PDF)
       │
       ▼
Copy to Google Doc format ← triggers built-in OCR on scanned images
       │
       ▼
Export as plain text
       │
       ▼
Clean & Structure (detect category headers, prices, descriptions)
       │
       ▼
Validate (≥5 chunks extracted?)
       │
       ▼
Get Embeddings (Ollama nomic-embed-text, 768 dims, per chunk)
       │
       ▼
Clear old Pinecone index → Upsert new vectors
       │
       ▼
Log to Google Sheets menu_index + Delete temp Doc
       │
       ▼
Confirm to owner via Telegram ✅
```

**Why Google Drive OCR?** Free, handles scanned images natively, no third-party service needed. Copying a PDF to `application/vnd.google-apps.document` format automatically applies OCR.

---

## Intent Routing Table

| Intent | Detection keywords | Action |
|--------|-------------------|--------|
| `GREETING` | hola, hey, buenas… | Welcome message with capabilities |
| `MENU_QUERY` | precio, menú, plato, vegano… | RAG → Pinecone → Ollama |
| `HOURS_INFO` | horario, dirección, wifi, pago… | RAG → Pinecone → Ollama |
| `ORDER` | pedir, domicilio, delivery… | Telegram kitchen group + client confirm + Sheets log |
| `RESERVATION` | reserva, mesa, personas… | Gmail owner + client confirm + Sheets log |
| `COMPLAINT` | queja, mal, frío, tarde… | Telegram empathy + Gmail 🚨 urgent + Sheets log |
| `FEEDBACK` | gracias, delicioso, excelente… | Thank-you reply + Sheets log |
| `UNKNOWN` | everything else | RAG attempt → escalate if score < 0.55 |

---

## Why Pinecone over Keyword Search

Keyword search fails for restaurant queries:
- "¿Tienen algo sin carne?" must match vegano, vegetariano, ensalada, sin proteína
- "¿El plato más económico?" must understand price ordering + categories

Pinecone semantic search finds relevant chunks regardless of exact wording. A query about "opciones ligeras" correctly returns salads and soups even without those exact words in the query.

---

## How the Owner Updates the Menu

1. Export the menu as PDF from any source
2. Send the PDF directly to the Telegram bot
3. Bot replies: "⏳ Recibí el PDF, aplicando OCR..."
4. ~1-2 minutes later: "✅ Menú actualizado — N chunks indexados"

No n8n UI access needed. No code changes. The menu is live immediately.

---

## Environment Variables (docker-compose.yml)

| Variable | Description |
|----------|-------------|
| `TELEGRAM_BOT_TOKEN` | Bot token from BotFather |
| `TELEGRAM_OWNER_CHAT_ID` | Owner's personal chat_id |
| `TELEGRAM_COCINA_GROUP_ID` | Kitchen group chat_id (negative number) |
| `PINECONE_API_KEY` | Pinecone API key |
| `PINECONE_HOST` | Pinecone index host URL |
| `GMAIL_DUENO` | Owner's email for alerts |
| `GOOGLE_SHEETS_ID` | Google Sheets file ID |
| `GOOGLE_DRIVE_FOLDER_ID` | Drive folder ID for menu PDFs |

---

## Google Sheets Structure

| Sheet | Purpose |
|-------|---------|
| `menu_index` | Indexed menu chunks audit log |
| `info_restaurante` | Manual restaurant info (hours, address, payments) |
| `pedidos` | Order log with status |
| `reservas` | Reservation log with confirmation status |
| `escalations` | Complaint and escalation records |
| `feedback` | Client sentiment log |
| `conversation_memory` | Per-session chat history |

---

## Adapting to Other Businesses

| Business | Replace | Keep |
|----------|---------|------|
| Hotel | menu → room catalog | reservation + complaint routing |
| Law firm | menu → service catalog | escalation + intake |
| Clinic | menu → treatment catalog | appointment booking |
| E-commerce | menu → product catalog | order + complaint + feedback |

Change intent keywords in `Detect Intent` and handler messages. RAG pipeline, Pinecone, and routing architecture stay identical.

---

## How to Run Locally

```bash
# 1. Complete setup: Pinecone index, Ollama pull nomic-embed-text, Telegram groups
# 2. Fill docker-compose.yml variables and restart
docker compose down && docker compose up -d

# 3. Import flows
docker cp flows/04-support-bot-rag/indexer-flow.json n8n-portfolio:/tmp/flow04-indexer.json
docker cp flows/04-support-bot-rag/bot-flow.json n8n-portfolio:/tmp/flow04-bot.json
MSYS_NO_PATHCONV=1 docker exec n8n-portfolio n8n import:workflow --input=/tmp/flow04-indexer.json
MSYS_NO_PATHCONV=1 docker exec n8n-portfolio n8n import:workflow --input=/tmp/flow04-bot.json

# 4. In n8n UI — connect Telegram credential on all Telegram nodes
# 5. Test indexer: send "actualizar menú" to your bot
# 6. Test bot: send "¿Tienen ajiaco?" to your bot
```

---

## Business Impact

| Metric | Manual | Automated |
|--------|--------|-----------|
| Menu Q&A response time | Hours | Seconds |
| Order routing to kitchen | Manual forward | Instant |
| Complaint escalation | Often missed | Always immediate |
| Menu update | Reprogram bot | Send PDF to Telegram |
| AI cost | $0.01–0.10/query | $0 — Ollama local |
| Weekly sentiment review | Manual | Sunday 8PM auto-email |
