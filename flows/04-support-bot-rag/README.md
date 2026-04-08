# Flow 04 — Support Bot RAG

## Problem Statement

Support teams answer the same questions repeatedly. Knowledge bases exist but agents don't consult them consistently. Response quality varies by agent experience, and scaling requires hiring more people.

## Solution Overview

A RAG (Retrieval-Augmented Generation) chatbot that connects to your knowledge base, retrieves the most relevant context, and generates accurate, grounded answers. Deployed on WhatsApp or web chat — available 24/7.

## Architecture Diagram

```
[WhatsApp / Webhook] --> Receive user message
      |
      v
[Embeddings Node] --> Vectorize the question
      |
      v
[Pinecone / Qdrant] --> Retrieve top-K relevant docs
      |
      v
[Code Node] --> Build prompt with context + question
      |
      v
[Ollama llama3.2] --> Generate grounded answer
      |
      v
[IF: Confidence check]
      |--- High confidence --> [WhatsApp] Send answer
      |--- Low confidence  --> [WhatsApp] Escalate to human agent
      |
      v
[Airtable] --> Log conversation for QA and fine-tuning
```

## Nodes Used

| Node | Purpose |
|------|---------|
| Webhook / WhatsApp | Receive customer messages |
| Embeddings | Vectorize query |
| Vector Store (Pinecone) | Semantic search over knowledge base |
| Code (JS) | Prompt engineering with context |
| Ollama Chat | Generate answer |
| IF | Confidence-based routing |
| WhatsApp | Send response |
| Airtable | Conversation logging |

## How to Run Locally

1. Start n8n and Ollama: `docker-compose up -d && ollama serve`
2. Import `flow.json`
3. Set up Pinecone index and API key
4. Run the "Ingest" sub-workflow to populate the vector store with your docs
5. Configure WhatsApp Business API credentials (or test via webhook)
6. Activate and test with: `curl -X POST http://localhost:5678/webhook/support -d '{"message":"How do I reset my password?"}'`

## Business Impact

- **Ticket deflection rate:** 60–70% of common queries resolved without human
- **Response time:** Instant vs. average 4h human response
- **Availability:** 24/7/365
- **Scalability:** Handles 1000s of concurrent conversations
- **Quality:** Answers grounded in your actual documentation (no hallucinations)
