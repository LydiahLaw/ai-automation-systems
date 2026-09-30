# AI Automation & Agent Portfolio

A collection of AI agent and automation projects built with n8n, demonstrating conversational AI, RAG (Retrieval-Augmented Generation), and voice agent design for customer support and business operations use cases.

Built as part of my transition into AI/DevOps-adjacent automation engineering, alongside my Cloud & DevOps work. See my [main portfolio](https://lydiahnganga.netlify.app) for the full picture of my work.

## Projects

| # | Project | Description | Key Skills |
|---|---------|-------------|------------|
| 1 | [Mali Travels Support Agent](./01-mali-travels-support-agent) | A conversational AI agent for a fictional Kenyan travel company — handles FAQs via RAG, collects booking requests, and escalates to a human when needed. | n8n, RAG, PGVector, AI Agents, Prompt Engineering |
| 2 | RAG Knowledge Agent | *(coming soon)* | n8n, Pinecone, RAG |
| 3 | WhatsApp Support Agent | *(coming soon)* | n8n, WhatsApp API, Conversational AI |
| 4 | Voice Support Agent | *(coming soon)* | n8n, ElevenLabs, Voice AI |

## Why these projects

These projects map directly to the core requirements of AI Solutions Engineer / Conversational AI roles: designing end-to-end AI solutions, working with LLMs, RAG, prompt engineering, and AI agents, and translating business requirements into working technical solutions for customer support and business operations.

## Tech stack across projects

- **n8n** (self-hosted via Docker) — workflow orchestration
- **OpenAI** — LLM (GPT-4.1-mini) + embeddings
- **PostgreSQL + pgvector** — vector storage for RAG, chat memory
- **Gmail API** — notification/escalation delivery
