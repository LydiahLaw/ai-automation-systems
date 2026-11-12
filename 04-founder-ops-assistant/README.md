# Founder Operations Assistant

An n8n workflow that triages a founder's inbox: it classifies new email, logs each one to Notion, and drafts a reply for review. Nothing is sent automatically.

Demo: https://www.loom.com/share/8ba0ae4788034b47942a7fb48f2239ff

## What it does

1. **Gmail Trigger** polls every 15 minutes for unread Inbox mail.
2. **Edit Fields** cleans the subject and sender, and passes the email body on.
3. **Notion lookup** checks the Founder Inbox database for the email's Gmail ID. If it exists, the workflow stops, so no duplicate rows are created.
4. **OpenAI (gpt-5-mini)** returns JSON with category (Lead, Client, Invoice, Scheduling, Other), priority, a one-sentence summary, and a draft reply.
5. **Notion** creates a row with the subject, sender, category, priority, summary, status, received time, and a link to the original email.
6. **If** checks whether a draft is needed. If so, **Gmail** saves it to Drafts.
7. **Gmail Mark as read** clears the email from the queue so it isn't processed again.
<img width="1366" height="689" alt="the workflow" src="https://github.com/user-attachments/assets/052b503d-7a61-4c80-90e3-e4ca0bb939c5" />


## Tested so far

- Manual runs and one automatic scheduled run.
- Test emails for Lead, Client, Invoice, Scheduling, and Other categories.
- Duplicate check: a re-run on an already-logged email created no new row.
- Drafts contain no em dashes and use short paragraphs.

## Limits

- Tested with my own test emails. It has not yet processed live client mail.
- Gmail sends mail to Spam without triggering the workflow, so spam-filtered client emails will be missed.
- Google apps in Testing mode can expire OAuth tokens (about every 7 days).
- The Scheduling draft contains a `[CALENDLY_LINK]` placeholder, which must be replaced before sending.
- Drafts are never sent automatically. The owner reviews every reply.

## Setup

- n8n (self-hosted in Docker), with Gmail, Notion, and OpenAI credentials.
- Notion database "Founder Inbox" with properties: Subject, From, Category, Priority, Summary, Status, Received, Email link.
- Import `workflow.json` (sanitised, no credentials) and reconnect your own accounts.

## Next steps

- Daily brief workflow (calendar, open Notion items, high-priority email summary).
- Optional rebuild of one step in Make.
