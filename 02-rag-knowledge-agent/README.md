# Nyumba Prime Properties — RAG Knowledge Agent

A conversational property assistant for **Nyumba Prime Properties**, a fictional Nairobi-based real estate agency. The agent answers questions about listings, pricing, legal process, and diaspora buyer support using a RAG-powered knowledge base backed by Pinecone, and routes viewing requests and human escalations by email.

## Architecture

```
Customer message
      |
      v
 Chat Trigger (n8n)
      |
      v
  AI Agent (OpenAI)
      |-- Simple Memory (conversation history)
      |-- Pinecone Knowledge Base (RAG retrieval for properties and FAQs)
      |-- Schedule Viewing Tool (Gmail, sends viewing requests)
      |-- Human Agent Tool (Gmail, routes to a person)
      |
      v
  Response to customer
```

A separate workflow handles knowledge base updates. Documents uploaded to a Google Drive folder are automatically chunked, embedded, and stored in Pinecone, so the knowledge base can grow without touching the agent workflow itself.

## Tools used

- **n8n** (self-hosted, Docker): workflow orchestration
- **OpenAI**: the agent's reasoning model and embeddings
- **Pinecone** (serverless): vector store for RAG
- **Google Drive**: source for knowledge base documents, auto-ingested on upload
- **Gmail API** (OAuth2): sends viewing requests and human-escalation alerts

## Key design decisions

- **Pinecone over a self-hosted vector DB**: demonstrates working with a managed vector database, a common requirement for RAG roles, alongside the Postgres/pgvector approach used in the Mali Travels project.
- **Drive-triggered ingestion**: knowledge base updates require no technical steps. Dropping a file into a folder is enough, making the system usable by a non-technical team.
- **Shared Gmail credential pattern**: reuses the same OAuth2 credential and tool structure as the Mali Travels project, showing a consistent, reusable approach across agents rather than one-off builds.

## Demo

[Watch the demo](https://www.loom.com/share/5243ba30cc7c42e3bcd91211e37f9783)

## Files in this project

- `knowledge-base/nyumba-prime-kb.md`: the source content used to build the RAG knowledge base
- `workflows/update-real-estate-kb.json`: exported n8n workflow for ingesting knowledge base documents from Google Drive into Pinecone
- `workflows/real-estate-rag-agent.json`: exported n8n workflow for the live property assistant