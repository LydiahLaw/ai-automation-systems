# Notion setup for Founder Inbox

Create a database called "Founder Inbox" with these properties (exact names and options matter, because the workflow maps to them):

| Property | Type | Options |
|---|---|---|
| Subject | Title | |
| From | Text | |
| Category | Select | Lead, Client, Invoice, Scheduling, Other |
| Priority | Select | High, Medium, Low |
| Summary | Text | |
| Status | Select | New, Draft ready, Done |
| Received | Date | |
| Email link | URL | |

Then create a Notion internal integration, copy its token into an n8n Notion credential, and connect the integration to the database (page menu, then Connections).

After importing `workflow.json`, reselect your own credentials (Gmail OAuth2, OpenAI, Notion) and point the Notion nodes at your database. The placeholders in the file must be replaced.