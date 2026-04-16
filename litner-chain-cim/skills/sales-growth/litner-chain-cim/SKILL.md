---
name: litner-chain-cim
description: >
  Generic Lead Nurturing CRM skill for Hermes Agent.
  Covers conversational CRM (CRUD), automated daily reports,
  bulk email outreach, and business card import via image recognition.
  Designed to be adapted to any client's database schema and stack.
version: 1.0.0
author: BusyCow
tags: [CRM, Lead Nurturing, Notion, Email, Telegram, Business Card]
---

# Litner Chain CIM — Lead Nurturing CRM

## Overview

This skill turns Hermes into a conversational CRM assistant. The agent:
- Logs and manages customer data through natural conversation (no manual database editing)
- Tracks follow-up rhythm and pipeline stages
- Sends a daily morning briefing with personalized follow-up drafts
- Sends bulk personalized emails by segment
- Imports contacts from business card photos

---

## Client Setup Checklist

Before activating this skill, the client must provide:

| Item | Description |
|------|-------------|
| `NOTION_TOKEN` | Notion integration API key |
| Accounts DB ID | The database tracking companies/leads |
| Contacts DB ID | The database tracking individual contacts |
| Activities DB ID | The database tracking every interaction |
| Content Library DB ID | The database storing email/message templates |
| `TELEGRAM_BOT_TOKEN` | Bot token for the daily report |
| Telegram Chat ID + Thread ID | Where to deliver the daily report |
| Email account | Gmail or SMTP credentials for sending |

Store all secrets in `~/.hermes/.env`. Reference them by name in this skill.

---

## Database Schema

> Each client's exact fields may differ. The schema below defines the **minimum recommended structure**.
> During onboarding, ask the client which fields they want to track and adjust accordingly.

### Accounts (Companies / Leads)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Company name |
| Stage | Status / Select | e.g. Lead → Qualified → Closing → Customer → Lost |
| Industry | Select | Client-defined |
| Country / Region | Select | Client-defined |
| Est. Value | Number | Expected contract value |
| Next Follow-up | Date | When to contact next |
| Owner | People | Assigned salesperson |
| Website | URL | |
| Notes | Rich Text | |

Recommended rollups from Activities (set up after relations are configured):
- Last Activity Date (function: latest_date)
- Activity Count (function: count)

### Contacts (People)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Person's name |
| Job Title | Rich Text | |
| Account | Relation → Accounts | Which company |
| Email | Email | |
| Phone | Phone Number | |
| Decision Role | Select | Decision Maker / Influencer / Executor |
| Preferred Channel | Select | Email / WhatsApp / LINE / Phone |
| Notes | Rich Text | How we met, referral info, etc. |

### Activities (Interaction Log)

| Field | Type | Notes |
|-------|------|-------|
| Summary | Title | One-line description of the interaction |
| Date | Date | When it happened |
| Type | Select | Call / Meeting / Demo / Email / Proposal / Contract / Other |
| Account | Relation → Accounts | |
| Contact | Relation → Contacts | |
| Client Response | Rich Text | What the client said |
| Next Action | Rich Text | What to do next |
| Stage Advanced? | Checkbox | Did this move the deal forward? |
| Owner | People | Who handled this |

### Content Library (Templates & Drafts)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Template or draft title |
| Type | Select | Email Template / WhatsApp Message / Follow-up Draft / Other |
| Stage Target | Multi-select | Which pipeline stages this applies to |
| Industry Target | Multi-select | Which industries this applies to |
| Channel | Multi-select | Email / WhatsApp / LINE / Phone Script |
| Status | Select | Draft / Ready / Archived |
| Body | Page content (blocks) | Full message text — stored in page body, NOT in table fields |

---

## Behavioral Rules

These rules define how the agent must behave in all CRM interactions.

### 1. Conversational Logging

- When the user reports any customer interaction, immediately write it to the Activities database
- Do NOT ask the user to open Notion or enter data manually
- If the user says "just talked to [Name]", create an Activity and ask for details

### 2. Proactive Field Completion

When adding a new Account or Activity, always check for missing required fields and ask for them before saving:

**For new Accounts, ask if missing:**
- Industry
- Company size or region
- How we met / lead source
- Who is the contact person?

**For new Activities, ask if missing:**
- Who did you speak with? (Contact)
- What did they say? (Client Response)
- What's the next step? (Next Action)
- When should we follow up? (Next Follow-up date)

> Always ask "下一步是什麼？" (What's the next step?) after every reported interaction.
> Record Next Action on the Activity AND update Next Follow-up on the Account.

### 3. Pipeline Stage Discipline

- Only advance the Stage when the user confirms progress
- If Stage Advanced? is checked, ask "Which stage are they moving to?"
- Log stage changes in the Activity summary

### 4. Query Support

The agent should be able to answer conversational queries such as:
- "What stage is [Company] at?"
- "What did we talk about last time with [Name]?"
- "Which accounts haven't been contacted in 3+ days?"
- "How many accounts are in each stage this month?"
- "Who's our biggest open opportunity right now?"

---

## Daily Morning Report

**Trigger**: Scheduled cron job (recommend 8:00–9:00 AM local time)
**Delivery**: Telegram (configured chat + thread)

### Report Generation Steps

1. Pull all Accounts and their Activities from Notion
2. Identify accounts that need follow-up:
   - Next Follow-up date is today or overdue
   - No Activity logged in the past 3 days
3. For each follow-up target, generate a personalized draft message:
   - Summarize last interaction context
   - Suggest next action based on stage
   - Draft a follow-up WhatsApp/Email message
   - Save draft to Content Library (Status: Draft)
4. Send the report to Telegram with:
   - Numbered list of today's follow-up targets
   - Stage, last interaction summary, suggested action for each
   - Note that drafts are ready (user can request them by name)
5. Append a pipeline health summary:
   - Count per stage
   - Any accounts stalled for 7+ days

### Example Report Format

```
📋 今日跟進清單（N 位客戶）

1️⃣ [Company] — [Contact Name] | Stage: [Stage]
上次互動：[Date]，[One-line summary]
📌 建議：[Suggested action]（跟進草稿已備好）

2️⃣ [Company] — [Contact Name] | Stage: [Stage]，逾期 N 天未互動
📌 建議：[Suggested action]

━━━━━━━━━━━━━━━━
📊 Pipeline 概況
Lead: N | Qualified: N | Closing: N | Customer: N
⚠️ 停滯 7 天以上：N 個客戶
```

---

## Bulk Email Outreach

**Trigger**: User requests a campaign (e.g. "Send the new case study to all Qualified leads")

### Steps

1. Identify the target segment from the user's request (by stage, industry, region, or custom filter)
2. Pull matching Accounts + their primary Contact's email from Notion
3. For each contact, personalize the message (insert name, company, relevant context)
4. Present the full recipient list for confirmation:

```
收件人確認（N 位）：
- [Name] | [Company] | [email]
- [Name] | [Company] | [email]

主旨：[Subject]
內容摘要：[Summary]

確認發送嗎？
```

5. Only send after the user explicitly confirms
6. After sending, log one Activity per Account (Type: Email, summary referencing the campaign)
7. Do NOT send to the same contact twice for the same campaign — check recent Activities before sending

### Email Sending Tool

Use the configured email account (Himalaya CLI or SMTP, as set up per client).
Reference the client's email config in `~/.hermes/.env`.

---

## Business Card Import

**Trigger**: User sends a photo (single card or batch)

### Single Card Flow

1. **Analyze the image** using vision tools. Extract:
   - Name, job title, department
   - Company name, website, address
   - Email, phone, mobile
   - Any social/messaging handles (LINE, WhatsApp, LinkedIn, WeChat)

2. **Present extracted info** for confirmation:
```
📇 名片資訊
姓名：[Name]
職稱：[Title]
公司：[Company]
Email：[Email]
電話：[Phone]
```

3. **Ask context questions** (always ask before importing):
   - Which Account does this person belong to? Or should we create a new one?
   - What is their decision role? (Decision Maker / Influencer / Executor)
   - Where / how did you meet them?
   - Any first interaction to log? (Activity)

4. **Import after confirmation**:
   - If new company → create Account first (apply standard page template if configured)
   - Create Contact with all extracted fields + context
   - If there's an initial interaction → create Activity
   - Ask "What's the next step?" and record Next Action + Next Follow-up

### Batch Cards (Trade Show / Event)

When the user sends multiple card photos:
1. Analyze all cards and present a summary list:
```
📇 辨識結果（N 張名片）
1. [Name] | [Company] | [Email]
2. [Name] | [Company] | [Email]
...
```
2. Ask once: "Where did you meet these people? Are they from the same event?"
3. After confirmation, create all Contacts (and Accounts if needed)
4. If from an event, create one shared Activity for all contacts referencing the event

### Front + Back of Same Card

If the user sends two consecutive photos of the same card:
- Detect it's the same person (matching name/company)
- Merge the information — do not create duplicate Contacts
- Chinese and English versions: use Chinese as primary, English in Notes or Job Title field

### Important Rules

- Never import to CRM without first showing a summary and asking context questions
- If the user says "just for reference" or "don't add yet", do not write to Notion
- If the company already exists in Accounts, link to the existing record — do not create a duplicate

---

## Customization Guide

When deploying for a new client, ask the following questions and update the skill accordingly:

| Question | Why it matters |
|----------|----------------|
| What are your pipeline stages? | Update Stage options in Accounts |
| Do you have resellers / partners separate from customers? | May need a Partner table |
| Which industries do you serve? | Update Industry select options |
| Which channels do you use most? (WhatsApp, Email, LINE?) | Affects Preferred Channel and Content Library |
| Do you want daily reports? What time? | Set up cron job |
| Who should receive the daily report? | Set Telegram chat/thread ID |
| Do you use Gmail or another mail provider? | Configure email tool accordingly |
| What language should the agent respond in? | Set language preference |

After gathering answers, update:
1. Notion database field options (Stage, Industry, Type, Channel)
2. `~/.hermes/.env` with the client's tokens and database IDs
3. Cron job schedule and delivery target
4. This skill file's configuration section with the client's actual IDs

---

## Configuration Block

Fill in this section after client onboarding:

```
# Client: [CLIENT NAME]
# Configured: [DATE]

NOTION_TOKEN=             # from ~/.hermes/.env
ACCOUNTS_DB=              # Notion database ID
CONTACTS_DB=              # Notion database ID
ACTIVITIES_DB=            # Notion database ID
CONTENT_LIBRARY_DB=       # Notion database ID

TELEGRAM_CHAT_ID=         # e.g. -100XXXXXXXXX
TELEGRAM_THREAD_ID=       # Thread/topic ID for daily report

EMAIL_FROM=               # Sender address
EMAIL_ACCOUNT=            # Account name in Himalaya config

DAILY_REPORT_TIME=        # e.g. "0 1 * * *" (01:00 UTC = 09:00 TWN)
REPORT_LANGUAGE=          # zh-TW / en / etc.
```
