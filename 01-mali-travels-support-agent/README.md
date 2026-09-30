# Mali Travels — AI Support & Booking Agent

A conversational AI agent for **Mali Travels**, a fictional Kenyan safari and travel company. The agent answers customer questions using a RAG-powered knowledge base, collects booking requests, and escalates to a human team member when needed.

## The problem

Small travel/tour operators often can't staff round-the-clock customer support, yet customers expect instant answers about packages, pricing, and booking logistics — especially outside business hours. This agent demonstrates how a small business could deploy an always-on first line of support without hiring additional staff.

## Architecture

```
Customer message
      |
      v
 Chat Trigger (n8n)
      |
      v
  AI Agent (GPT-4.1-mini)
      |-- Postgres Chat Memory (conversation history)
      |-- PGVector Knowledge Base (RAG - FAQ retrieval)
      |-- Booking Tool (Gmail - sends booking requests)
      |-- Human Escalation Tool (Gmail - routes to a person)
      |
      v
  Response to customer
```

A separate workflow handles knowledge base updates: a form upload feeds into OpenAI Embeddings, which writes into the Postgres PGVector Store — so the knowledge base can be updated any time without touching the agent workflow itself.

## Tools used

- **n8n** (self-hosted, Docker) — workflow orchestration
- **OpenAI GPT-4.1-mini** — the agent's reasoning model
- **OpenAI Embeddings** — turns knowledge base text into vectors for retrieval
- **PostgreSQL + pgvector** (Docker) — vector store for RAG, plus chat memory persistence
- **Gmail API (OAuth2)** — sends booking confirmations and human-escalation alerts

## Key design decisions

- **RAG over a static FAQ list**: lets the agent answer naturally-phrased questions rather than requiring exact keyword matches, and lets the knowledge base grow without changing the agent's logic.
- **Separate KB-update workflow**: decouples "teaching" the agent from the agent's runtime logic — a non-technical team member could update the knowledge base by just uploading a new document through a form.
- **Explicit escalation path**: the agent is instructed to hand off to a human when a request falls outside its knowledge base or the customer explicitly asks for a person, rather than guessing or hallucinating an answer.

## Demo

*(Add your recorded demo link here once complete — a short recording showing a few sample conversations: an FAQ question, a booking request, and a human-escalation request.)*

## Files in this project

- `knowledge-base/mali-travels-kb.md` — the source Q&A content used to build the RAG knowledge base
- `workflows/update-mali-travels-kb.json` — exported n8n workflow for updating the knowledge base
- `workflows/mali-travels-support-booking.json` — exported n8n workflow for the live support/booking agent

## Background

Built while working through an n8n/AI automation course, then adapted into an original travel-industry use case with custom knowledge base content, branding, and booking flow.
