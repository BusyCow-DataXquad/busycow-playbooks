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
- 「剛跟 Alex 通話，他說下週可以安排 Use Case 確認」

**Log a partner interaction:**
- 「剛跟 TechBridge 開完會，他們說下個月會再轉介兩個客戶」

**Query pipeline:**
- 「哪些客戶超過三天沒聯絡了？」
- 「GreenBridge 現在在哪個階段？」
- 「有哪些 Partner 上個月沒有任何互動？」

**Generate a quotation:**
- 「幫我幫 NovaTech 做一份報價單，方案是標準部署，50,000 HKD，有效期 30 天」

**Generate a contract:**
- 「GreenBridge 要簽約了，幫我用標準合約範本產生合約，填入他們的資料」

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
