# Lead Nurturing CRM

A complete AI-powered CRM system for SMEs using Notion as the database and Telegram as the daily interface.

## What's Included

- Conversational CRM — log and query customer data through natural chat
- Automated daily morning briefing with personalized follow-up drafts
- Bulk personalized email outreach by pipeline stage or industry
- Business card image recognition and auto-import to Contacts

## Stack

- Hermes Agent (AI backbone)
- Notion (database — Accounts, Contacts, Activities, Content Library)
- Telegram (daily report delivery)
- Email / SMTP (outreach)

---

## Setup Steps

### Step 1 — Notion

1. Create a Notion integration at https://www.notion.so/my-integrations
2. Copy the integration token
3. Create the 4 required databases (see `notion-schema/README.md`)
4. Share each database with your integration

### Step 2 — Telegram

1. Create a bot via @BotFather, copy the bot token
2. Add the bot to your team group
3. Enable Topics if using threads
4. Note the chat ID and thread ID for the daily report

### Step 3 — Email

1. Set up the email account for outreach
2. Configure Himalaya CLI (see `config-template/email-setup.md`)

### Step 4 — Install the Skill

```bash
cp -r skills/productivity/litner-chain-cim ~/.hermes/skills/productivity/
```

### Step 5 — Configure

Fill in `config-template/config.yaml` with the client's values and apply to `~/.hermes/config.yaml` and `~/.hermes/.env`.

### Step 6 — Set Up Daily Report Cron

```bash
hermes cron create \
  --schedule "0 1 * * *" \
  --name "daily-crm-report" \
  --prompt "Generate the daily CRM follow-up report and send to Telegram"
```

---

## Folder Structure

```
litner-chain-cim/
├── README.md
├── skills/
│   └── productivity/
│       └── litner-chain-cim/
│           └── SKILL.md          ← Core skill (install this)
├── notion-schema/
│   └── README.md                 ← Database structure guide
└── config-template/
    ├── config.yaml               ← Hermes config template
    └── env-template.txt          ← .env variables template
```
