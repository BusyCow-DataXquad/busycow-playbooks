# Lead & Partner Nurturing — Use Case

Small and medium businesses lose deals not because they lack leads, but because follow-up falls through the cracks. When sales conversations, next steps, and relationship history live in spreadsheets, WhatsApp threads, and people's heads, nothing gets acted on consistently. The Lead & Partner Nurturing use case gives your team a conversational CRM assistant: just tell it what happened, and it handles the logging, tracking, and follow-up planning — no manual Notion editing required.

This use case covers both customer leads and partner relationships in a single unified system. Log a call, check a pipeline stage, run a morning briefing, send bulk outreach, or import a stack of business cards from a trade show — all through natural chat with Hermes.

---

## Features

- **Conversational CRM Logging** — Report interactions in plain language; agent logs to Notion automatically and asks for any missing fields
- **Deal Page Template Auto-Apply** — Every new Deal page gets a structured client tracking template written in automatically
- **Proactive Field Completion** — Agent checks for missing fields when creating Accounts or Activities and asks before saving
- **Pipeline Stage Discipline** — Stage advances only on explicit user confirmation; all changes are logged in Activities
- **Query Support** — Ask anything: who needs follow-up, what stage a company is at, which partners have gone quiet
- **Daily Morning Report** — Scheduled briefing covering today's follow-up targets, suggested actions, and pipeline health summary; delivered to Telegram
- **Bulk Email Outreach** — Send personalized campaigns by pipeline stage, industry, or custom filter; confirmation required before sending
- **Business Card Import** — Send a card photo; agent extracts info, asks context questions, and imports to Contacts (and Accounts if new)

---

## What's in This Package

| File | Purpose |
|------|---------|
| README.md | This file — human overview, features, and install steps |
| SETUP.md | Agent-facing setup instructions — run once during onboarding |
| SKILL.md | Agent-facing daily operating instructions — loaded on every use |
| config-template/env-template.txt | Environment variable template — copy and fill in |

---

## Install Steps

### Step 1 — Copy config template

```bash
cp use-cases/growth/lead-nurturing/config-template/env-template.txt ~/.hermes/.env
```

Open `~/.hermes/.env` and fill in all values. Leave anything unknown blank for now — SETUP.md will guide the agent through finding or creating databases.

### Step 2 — Copy SKILL.md to your Hermes skills folder

```bash
mkdir -p ~/.hermes/skills/growth/lead-nurturing
cp use-cases/growth/lead-nurturing/SKILL.md ~/.hermes/skills/growth/lead-nurturing/SKILL.md
```

### Step 3 — Run the setup

Tell Hermes:

> "Run the Lead Nurturing setup — use SETUP.md from the lead-nurturing use case."

The agent will search for existing Notion databases, help create any that are missing, adapt schemas to add recommended fields, and save all IDs to `~/.hermes/.env`.

### Step 4 — Set up daily report (optional)

```bash
hermes cron create \
  --schedule "0 1 * * *" \
  --name "daily-crm-report" \
  --prompt "Generate the daily CRM follow-up report and send to Telegram"
```

Adjust the schedule to match your timezone (01:00 UTC = 09:00 Taiwan time).

### Step 5 — Set up Telegram

1. Create a bot via @BotFather and copy the bot token
2. Add the bot to your team group or channel
3. Note the Chat ID and Thread ID (if using topics)
4. Add `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`, and `TELEGRAM_THREAD_ID` to `~/.hermes/.env`
