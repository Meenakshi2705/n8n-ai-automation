
# AI Agent + Google Sheets CRM Automation (n8n)

A lightweight **AI-assisted CRM updater** built in **n8n**. It listens to **Telegram** messages, uses a **Google Gemini** LLM as an **Agent** with **tool access**, reads your **Google Sheet**, decides what to append/update, writes back, and replies with a short confirmation.

> Tech: n8n, Google Sheets, Google Gemini (LLM), Telegram Bot

---

## What this workflow does

- **Trigger:** Telegram message (`/start`, plain text, polls).
- **Agent:** A LangChain-style **AI Agent** with a strict system prompt—*always check the sheet first, only update what’s new, confirm briefly*.
- **Memory:** Short-term conversation window (last 15 turns).
- **Tools:** 
  - **Read rows** from a Google Sheet.
  - **Append or Update** a row (match by `Client`).
- **Output:** Telegram confirmation like “Updated client revenue in the sheet.”

---

## Architecture (high level)

```
[Telegram Trigger]
        |
        v
[AI Agent (Gemini Chat Model) + Memory]
        |
        |-- uses tool: Read Google Sheet
        |
        |-- uses tool: AppendOrUpdate Google Sheet (matching column: Client)
        |
        v
[Short confirmation back to user]
```

### n8n Nodes (as configured)
- **Telegram Trigger** – listens for user messages.
- **Google Gemini Chat Model** – the LLM backing the Agent.
- **AI Agent** – orchestrates tool calls using the system rules.
- **Simple Memory** – windowed memory (15).
- **Google Sheets: Get row(s)** – read tool.
- **Google Sheets: Append or update row** – write tool (matching on `Client`).

---

## Google Sheet structure (expected)

Sheet: `Sheet1` (gid=0)

Columns (case-sensitive):
- `Client` *(used as unique key for matching)*
- `Amount Paid`
- `Industry`
- `Gmail`

> Tip: If `Amount Paid` represents **Revenue**, keep it as a number; you can format as `₹` in Sheets UI or handle currency formatting in downstream dashboards.

---

## Setup

1. **n8n**: Self-host or Cloud. Import the provided `My workflow.json` via **Workflows → Import from File**.
2. **Credentials** (set in n8n UI):
   - **Telegram API** – Bot token from BotFather.
   - **Google Sheets OAuth2** – connect the Google account that owns the sheet.
   - **Google Gemini (PaLM) API** – set the API key for the LLM.
3. **Sheet Access**: Put your Google Sheet **Document ID** and **Sheet name** into the node params.
4. **Security**: Do **not** commit tokens or credential IDs. Use n8n Credentials, environment variables, or the Secrets store.

---

## Usage (example Telegram prompts)

- `Add client Acme Corp with Amount Paid 250000, Industry SaaS, Gmail ops@acme.com`
- `Update SlashTech revenue to 175000`
- `Change Gmail for Bright Labs to hello@brightlabs.io`

The agent will:
1) Read the sheet to verify current values
2) Decide what changed
3) Append/update only the relevant row (matching `Client`)
4) Reply with a short confirmation (as per system prompt)

---



