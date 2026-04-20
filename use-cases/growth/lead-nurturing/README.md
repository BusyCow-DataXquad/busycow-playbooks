# Lead Nurturing — Use Case

An AI-powered CRM assistant for managing customer relationships through natural conversation.
Log interactions, track pipeline, send daily briefings, and run personalized outreach — all via chat.

**Category:** Growth
**Data Foundation Required:** [Growth](../../data-foundation/growth/README.md)

---

## What This Use Case Does

- Conversational CRM — log and query customer data through natural chat (no manual Notion editing)
- Automated daily morning briefing with personalized follow-up drafts saved to Content Library
- Bulk personalized email outreach by pipeline stage or industry
- Business card image recognition and auto-import to Contacts

---

## Required Data Tables

Install these tables from `data-foundation/growth/` before activating this use case:

| Table | Layer | Purpose |
|-------|-------|---------|
| Accounts | Fact | Companies and leads |
| Contacts | Fact | Individual people |
| Activities | Record | Every interaction logged here |
| Deals | Relationship | Pipeline and stages |
| Content Library | Standalone | Follow-up drafts and templates |

> **Not required:** Partnership (only needed if also running Partner Nurturing)

---

## Setup Steps

### Step 1 — Create Notion Databases

Follow `data-foundation/growth/README.md` to create the 5 required tables.
Share all databases with your Notion integration.

### Step 2 — Install the Skill

```bash
cp -r skills/growth/lead-nurturing ~/.hermes/skills/growth/
```

### Step 3 — Configure

```bash
cat config-template/env-template.txt >> ~/.hermes/.env
# Fill in all values
```

### Step 4 — Set Up Daily Report (Optional)

```bash
hermes cron create \
  --schedule "0 1 * * *" \
  --name "daily-crm-report" \
  --prompt "Generate the daily CRM follow-up report and send to Telegram"
```

### Step 5 — Set Up Telegram

1. Create a bot via @BotFather, copy the bot token
2. Add the bot to your team group
3. Note the chat ID and thread ID for the daily report

---

## How to Use

**Log an interaction:**
- 「剛跟 Morris 通話完，他說下週可以 demo」

**Query pipeline:**
- 「哪些客戶超過三天沒聯絡了？」
- 「AquaOptima 現在在哪個階段？」

**Import a business card:**
- Send a photo of the card — agent extracts info, confirms, and creates Contact

**Run outreach:**
- 「幫我寄開發信給所有 Qualified 階段的客戶」

---

## Folder Structure

```
lead-nurturing/
├── README.md
├── skills/
│   └── growth/
│       └── lead-nurturing/
│           └── SKILL.md          ← Install this
└── config-template/
    ├── config.yaml
    └── env-template.txt
```
