# Lead & Partners Nurturing — Use Case

An AI-powered CRM assistant for managing customer leads and partner relationships through natural conversation.
Log interactions, track pipeline stages, maintain partner relationships, generate quotations and contracts, send daily briefings, and run personalized outreach — all via chat.

**Category:** Growth
**Data Foundation Required:** [Growth](../../data-foundation/growth/README.md)

---

## What This Use Case Does

- **Conversational CRM** — log and query leads, contacts, and partner data through natural chat; not just lead tracking but also partner relationship maintenance and progress follow-up (no manual Notion editing required)
- **Automated daily morning briefing** with personalized follow-up drafts saved to Content Library, covering both pending leads and partner check-ins
- **Bulk personalized email outreach** by pipeline stage or industry
- **Quotation generation** — automatically produce structured quotations (line items, pricing, validity period) based on client requirements, with version control and multi-currency support
- **Contract generation** — apply contract templates to closing deals, auto-fill client data, service terms, payment schedule and duration, output a signable document, and log contract status back to CRM
- **Business card image recognition** and auto-import to Contacts

---

## Required Data Tables

Install these tables from `data-foundation/growth/` before activating this use case:

| Table | Layer | Purpose |
|-------|-------|---------|
| Accounts | Fact | Companies, leads, and partners |
| Contacts | Fact | Individual people |
| Activities | Record | Every interaction logged here (leads and partners) |
| Deals | Relationship | Pipeline and stages |
| Partnership | Relationship | Partner relationships and collaboration status |
| Content Library | Standalone | Follow-up drafts, quotation templates, contracts |

---

## Setup Steps

### Step 1 — Create Notion Databases

Follow `data-foundation/growth/README.md` to create the 6 required tables.
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

**Log a lead interaction:**
- "Just got off the phone with Alex — he said we can schedule the Use Case confirmation next week"

**Log a partner interaction:**
- "Just wrapped up a meeting with TechBridge — they said they'll refer two more clients next month"

**Query pipeline:**
- "Which clients haven't been contacted in over three days?"
- "What stage is GreenBridge at right now?"
- "Which partners had no interactions last month?"

**Generate a quotation:**
- "Create a quotation for NovaTech — standard deployment package, 50,000 HKD, valid for 30 days"

**Generate a contract:**
- "GreenBridge is ready to sign — generate a contract using the standard template and fill in their details"

**Import a business card:**
- Send a photo of the card — agent extracts info, confirms, and creates Contact

**Run outreach:**
- "Send a cold outreach email to all clients in the Qualified stage"

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
