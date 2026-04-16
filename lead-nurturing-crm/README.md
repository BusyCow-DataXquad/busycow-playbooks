# Lead Nurturing CRM Playbook

A complete CRM setup for SMEs using Notion as the database and Telegram as the interface.

## What's Included

- AI agent that logs activities via conversation (no manual Notion editing needed)
- Daily follow-up report delivered to Telegram every morning
- Accounts, Contacts, and Activities tracking
- Next Action prompts after every follow-up

## Stack

- Hermes Agent (AI backbone)
- Notion (database)
- Telegram (interface)

---

## Setup Steps

### Step 1 — Notion

1. Create a new Notion workspace (or use an existing one)
2. Generate a Notion integration token at https://www.notion.so/my-integrations
3. Import the database schema (see `notion-schema/README.md`)

### Step 2 — Telegram

1. Create a new bot via @BotFather
2. Add the bot to your team group
3. Note the bot token and your chat/thread ID

### Step 3 — Install Skills

```bash
cp -r skills/productivity/* ~/.hermes/skills/productivity/
```

### Step 4 — Configure

Copy `config-template/config.yaml` to `~/.hermes/config.yaml` and fill in your values.

---

## Folder Structure

```
lead-nurturing-crm/
├── README.md               # This file
├── skills/                 # Hermes skill files
│   └── productivity/
│       ├── notion/
│       ├── notion-api-v2025/
│       └── notion-crm-lead-nurturing/
├── notion-schema/          # Database structure guide
│   └── README.md
└── config-template/        # Settings template
    └── config.yaml
```
