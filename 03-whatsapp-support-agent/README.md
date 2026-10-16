# WhatsApp Support and Sales Agent (n8n, OpenAI, Supabase)

An AI assistant on WhatsApp for a fictional Texas insurance agency, "Mesquite Ridge Insurance Agency". It answers coverage and claims questions from a knowledge base, understands voice notes, remembers the conversation, and hands qualified leads to a human agent by email, only after the customer agrees.

> **Demo project.** Mesquite Ridge Insurance Agency is fictional. The assistant does not sell insurance, quote prices or bind coverage. The knowledge base is demonstration content, not legal or insurance advice.

## What it does

- Answers questions about Texas auto, home, renters, flood and windstorm insurance and claims, using only a retrieved knowledge base (RAG).
- Accepts text and voice notes. Voice notes are transcribed, then handled like text.
- Remembers the last 20 messages per customer (keyed by WhatsApp number), so it does not re-ask for details.
- Collects lead details and emails them to a licensed agent, but only after the customer says yes.
- Refuses what it must not do: price quotes, promises of claim approval, legal advice, and sensitive data such as Social Security numbers.
- Replies in the customer's language (English or Spanish).

## Architecture

```
Customer on WhatsApp
        |
        v
Meta WhatsApp Cloud API  --webhook-->  n8n (self-hosted, Docker, ngrok tunnel)
                                              |
                                      WhatsApp Trigger
                                              |
                                       Switch on type
                          +-------------------+-------------------+
                          | audio                                 | text
                          v                                       v
                   Get Audio URL                          Get User's Message
                          |                                       |
                   Download Audio                                 |
                          |                                       |
                   Transcribe Audio (Gemini)                      |
                     |            |                               |
                 success        error                             |
                     |            |                               |
              Get Audio Message   Voice Fallback Reply            |
                     |            ("please type it")              |
                     +---------------------+----------------------+
                                           v
                                       AI Agent
                          +----------------+-----------------+
                          |                |                 |
                   OpenAI chat model  Postgres chat     Tools:
                   (gpt-5-mini)        memory (20)      - texas_insurance_kb
                                                          (Supabase vector store)
                                                        - Request Agent Callback
                                                          (Gmail)
                                           |
                                           v
                                    Reply To User (WhatsApp)
```

A second workflow, `update-insurance-kb`, watches a Google Drive folder. When a knowledge base file is added, it splits the text (900 characters, 50 overlap), embeds it with OpenAI and writes it to a Supabase vector table.

## Stack

| Layer | Tool |
|---|---|
| Channel | Meta WhatsApp Cloud API |
| Orchestration | n8n (self-hosted in Docker) with an ngrok tunnel |
| LLM | OpenAI gpt-5-mini (reasoning effort low) |
| Embeddings and retrieval | OpenAI embeddings, Supabase pgvector |
| Chat memory | Postgres |
| Voice transcription | Google Gemini |
| Lead handoff | Gmail |

## Guardrails

| Risk | Behavior |
|---|---|
| Price or premium quote | Refuses and offers an agent callback |
| "Will my claim be approved?" | Says the insurer decides, gives general claim steps |
| Social Security, license, card or bank details | Tells the customer not to share them here |
| Callback emailed without consent | Reads details back and waits for a clear yes |
| Claiming an action that did not happen | Prompt forbids saying "sent" unless the tool ran |
| Answer not in the knowledge base | Says it is not sure and offers an agent |
| Danger or emergency | Tells the customer to call 911 |
| Voice transcription fails | Replies asking the customer to type instead |

Output is also post-processed in n8n: em dashes are replaced in code, because prompt rules alone were not fully reliable.

## Test results (manual)

Tested from a phone on the WhatsApp test number.

| Test | Result |
|---|---|
| Coverage question from the knowledge base (text and voice) | Answered correctly |
| "Can you give me a quote?" | Refused, offered an agent |
| "Will my hail claim be approved?" | Did not promise approval, gave general steps |
| Message containing a fake SSN | Told me not to share it |
| Callback flow | Confirmed details, waited for yes, one email sent with correct fields |
| Context across turns | Reused earlier details within the 20 message window |
| Gemini outage (503) | Retried five times, then sent the fallback reply |

Text replies took about 5 seconds after setting reasoning effort to low. They took 12 to 14 seconds before. Voice notes are slower because of download plus transcription.

I have not yet built an automated evaluation set. That is the next improvement (see below).

## Setup

1. **Knowledge base.** Create a Supabase project, enable pgvector, create the `documents` table and the `match_documents` function (the standard n8n Supabase vector store setup). Put `kb/mesquite-ridge-kb.md` in a Google Drive folder and run `update-insurance-kb`.
2. **Meta app.** Create a Meta developer app with the WhatsApp product. Add a privacy policy URL (see `docs/privacy-policy.html`), publish the app, and create a System User token with `whatsapp_business_messaging` and `whatsapp_business_management`. Subscribe your WhatsApp Business Account to the app with `POST /{WABA_ID}/subscribed_apps`.
3. **n8n.** Run n8n in Docker with `WEBHOOK_URL` set to your public tunnel URL. Create credentials for WhatsApp (send and trigger), OpenAI, Supabase, Postgres, Gmail and Gemini.
4. **Import** `workflows/insurance-whatsapp-agent.json` and `workflows/update-insurance-kb.json`. Re-select each credential, set your Phone Number ID in the WhatsApp send nodes, set the callback email address in `Request Agent Callback`, and pick your Drive folder.
5. Publish the workflow and message the test number.

The exported workflows have credential IDs, webhook IDs, my email and my Phone Number ID removed. Secrets are never stored in the export.

## Known limitations

- Gmail OAuth in testing mode expires after 7 days. A long-running deployment needs the Google app published to production.
- Free Supabase projects pause after a week of inactivity.
- The WhatsApp test number only messages numbers added to the allowed list.
- Callback emails go to one inbox. There is no CRM or ticket creation yet.
- Gemini can return 503 under load. The workflow retries, then falls back to a typed reply.
- The knowledge base is demonstration content. Figures such as penalties and coverage limits should be verified before any real use.

## What I would improve next

- An evaluation table: 20 test questions with pass rate, latency and token cost per run.
- Create the lead in a CRM (HubSpot) as well as sending the email.
- Human handoff with a live thread instead of a one-way email.
- Move from the test number to a verified business number and a hosted n8n instance.

## Repository layout

```
03-whatsapp-support-agent/
  README.md
  workflows/
    insurance-whatsapp-agent.json
    update-insurance-kb.json
  kb/
    mesquite-ridge-kb.md
  docs/
    privacy-policy.html
```
