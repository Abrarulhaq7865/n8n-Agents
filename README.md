# n8n-Agents

## What this is
A collection of n8n workflow templates and example automations that use LLM integrations (Google Gemini, Groq/LLaMA, and other models) and common cloud services (Google Sheets/Drive, Gmail, Telegram) to automate summarization, reporting, auditing, and resume screening.

### Stack
- **Language(s):** JSON (n8n workflow exports) and small JavaScript snippets inside n8n Code nodes
- **Framework / runtime:** n8n (workflow automation platform)
- **Notable integrations/services:** Google Gemini (PaLM) / Google Palm API, Groq (LLaMA endpoints), Google Sheets/Drive, Gmail, Telegram, SMTP

## How it's organized
```text
README.md                          - this overview
tech-news-summarizer.json          - n8n workflow: fetch tech RSS & summarize via LLM, email results
india-news-summarizer.json         - n8n workflow: fetch India news RSS & summarize via LLM, email results
ats-resume-scanner.json            - n8n workflow: form trigger → extract resume → LLM score & keywords → append to Google Sheet
attendance-auditor.json            - n8n workflow: download attendance CSV → audit via LLM agents → send email / WhatsApp draft
multi_agent_research_report_groq.json - n8n workflow: Telegram-triggered multi-agent research pipeline using Groq + LLaMA models and Google Sheets logging
empty-template-workflow.json       - empty template workflow
```

How it fits together: each .json file is an exported n8n workflow. Workflows use n8n triggers (manual, form, Telegram) to start flows, Code nodes for small transformations, model nodes (LangChain-style or HTTP requests) to call LLMs, and output integrations (email, Google Sheets, Telegram) to deliver or persist results. Several workflows rely on external credentials (Google APIs, Groq API, SMTP/Gmail, Telegram bot token).

## How to run it
The files are n8n workflow exports. Import the JSON into an n8n instance to run them.

Quick start (recommended: Docker):

1. Start n8n with Docker (example):

```bash
# run n8n as a container (data ephemeral). Bind port 5678 or change as needed.
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -e N8N_BASIC_AUTH_ACTIVE=false \
  n8nio/n8n:latest
```

2. Open http://localhost:5678, go to "Workflows" → "Import from file" and upload a .json workflow (one at a time).

3. Configure credentials for each workflow before activating:
   - Google APIs: Google Sheets, Google Drive (OAuth2 credentials)
   - Google Palm / Gemini: API key / credential used by the Google Gemini node
   - Groq: API key / HTTP header credential for Groq endpoints
   - Gmail / SMTP: OAuth2 or SMTP credentials for sending email
   - Telegram: Bot token for Telegram Trigger/Send nodes

4. Test manually with the included manual triggers or sample inputs. Review Code nodes for expected JSON shapes (some Code nodes parse model outputs — if you change models, update parsing logic).

Alternative (npm):

```bash
npm install -g n8n
n8n start
```

Notes:
- The workflows include nodes that expect specific fields (for example, the ATS scanner expects a Job Description and a Resume file). When importing, check the form trigger fields and test with matching data.
- Several workflows include LLM-specific prompts and model choices; update those nodes if you prefer different models or providers.

## Workflows (brief descriptions)
- tech-news-summarizer.json — Reads the TechCrunch RSS feed, collates items via a Code node, uses a Google Gemini LLM node (chainLlm) to produce a 3-part summary, and emails the summary.
- india-news-summarizer.json — Same pattern as tech-news but targets an India news RSS feed (abplive). Uses Gemini and emails results.
- ats-resume-scanner.json — A form-triggered workflow that accepts a Job Description and Resume (file), extracts text from the resume, merges inputs, prepares a prompt for a model (Gemini), extracts structured scoring/keywords/feedback via a Code node, and appends results to a Google Sheet.
- attendance-auditor.json — Downloads an attendance CSV from Google Drive, loops over rows, uses Groq/LLaMA model agents to audit attendance and draft outreach messages, then sends emails (and drafts WhatsApp text via an agent). Includes an emailSend node configured for SMTP.
- multi_agent_research_report_groq.json — A Telegram-triggered multi-agent research pipeline: multiple Groq (LLaMA) agents perform web research, statistics, and expert insights; their outputs are merged and compiled into a professional report, then sent back to Telegram and logged to Google Sheets.
- empty-template-workflow.json — Empty workflow template to use as a starting point.

## Security & credentials
These workflows reference external API credentials (OAuth2/Groq/API keys). Do NOT commit real secrets to this repo. When importing, create credentials in your n8n instance and bind them to the workflow nodes.

## Try asking
- How can I change the ATS Resume Scanner to use OpenAI instead of Google Gemini? Which nodes and prompt parsing need edits? (Hint: check nodes named "Message a model" and the Code nodes that parse JSON output.)
- In attendance-auditor.json the workflow downloads a CSV and loops rows; how can I adapt it to write audit results to a Google Sheet and also send Slack notifications? Which nodes should I add/replace?
- The multi-agent Groq workflow compiles reports — where should I change the chunk size or Telegram formatting if messages are being split incorrectly? Look at "Split Report into Chunks" and the Send Report via Telegram node.

---

(Updated README to summarize repository contents, usage steps, and per-workflow notes.)
