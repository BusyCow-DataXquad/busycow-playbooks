---
name: lead-nurturing-crm
description: >
  Conversational CRM assistant for lead and partner relationship management.
  Covers CRM logging, pipeline tracking, daily morning briefings,
  bulk email outreach, and business card import via image recognition.
  Works with whatever Notion databases were set up during onboarding.
version: 2.0.0
author: BusyCow
tags: [CRM, Lead Nurturing, Partners, Notion, Email, Telegram, Business Card]
---

# Lead Nurturing CRM — Daily Operations

## Overview

This skill turns Hermes into a conversational CRM and partner relationship assistant. Setup is already complete — databases are configured in agent memory from the SETUP phase.

Features:
1. **Conversational CRM Logging** — log interactions, create accounts/contacts, update pipeline stages through natural chat
2. **Deal Page Template Auto-Apply** — structured client tracking template written to every new Deal page automatically
3. **Proactive Field Completion** — agent checks for missing fields before saving any Account or Activity
4. **Pipeline Stage Discipline** — stages only advance on explicit user confirmation; changes logged in Activities
5. **Query Support** — answer conversational questions about pipeline, contacts, and follow-up status
6. **Daily Morning Report** — scheduled briefing with follow-up targets, suggested actions, and pipeline summary; delivered to Telegram
7. **Bulk Email Outreach** — personalized campaigns by segment; confirm before sending; log Activities after
8. **Business Card Import** — extract contact info from photos, ask context questions, import to Contacts

> Field names in your client's databases may differ from the recommended schema. Always check the actual field names when querying or writing.

---

## Feature 1 — Conversational CRM Logging

### Triggers

- User reports any customer or partner interaction in plain language
- "Just got off the phone with Alex — he said we can schedule the demo next week"
- "Met with TechBridge today — they're interested in the reseller program"
- "Update GreenBridge's stage to Qualified"

### Workflow

1. Identify the account and contact mentioned
2. Look up their existing record in the Accounts and Contacts databases if not certain
3. Create an Activity entry with the interaction summary, date, and any details provided
4. Check for missing required fields (see Proactive Field Completion below)
5. Ask "What's the next step?" if not already stated — record the answer as Next Action on the Activity and update the follow-up date on the Account
6. Confirm: "Logged — [one-line summary of what was recorded]"

### Response format

```
Logged — [Account name] / [Contact name]
Activity: [Interaction summary]
Next action: [Next step]
Follow-up: [Date or "not set"]
```

---

## Feature 2 — Deal Page Template Auto-Apply

### Trigger

Every time a new Deal page is created in the Deals database.

### Workflow

1. Immediately after creating the Deal page, write the following structured template into the page body using the Notion API (append blocks to the page_id)
2. If the user provided client information at creation time, pre-fill the relevant fields instead of leaving them as placeholders
3. Do this automatically — do not ask the user to fill in the template manually
4. Confirm: "Deal created — client tracking template applied."

### Template structure

```
# Part 1 — Client Information

## Client Overview
| Field          | Details        |
| Company Name   | (to be filled) |
| Industry       | (to be filled) |
| Company Size   | (to be filled) |
| Region         | (to be filled) |
| Website        | (to be filled) |
| Referred By    | (to be filled) |

## Team Composition
| Name           | Title          | Role                                    |
| (to be filled) | (to be filled) | Decision Maker / Influencer / Executor  |

## Existing Systems
- ERP: (to be confirmed)
- Daily workflow tools: (to be filled)
- Communication tools: (to be filled)

---

# Part 2 — Use Case Planning

## Use Case 1 — (to be filled)
- Current process: (to be filled)
- Our solution: (to be filled)
- Core value: (to be filled)

## Use Case 2 — (to be filled)
- (to be filled)

---

# Part 3 — ROI Calculation

## Labor Cost Estimate
- Monthly salary: (to be filled)
- Total monthly cost including benefits: (to be filled)
- Annual labor cost: (to be filled)

## Payback Estimate
- Plan A: Payback period / Annual net savings / ROI (to be filled)
- Plan B: Payback period / Annual net savings / ROI (to be filled)
- Year 2 onwards: Annual net savings / ROI (to be filled)

---

# Part 4 — Potential Upgrade Applications
1. (to be filled)
```

Use `table` blocks with `table_row` children for the tables. Write all blocks in a single API call where possible.

---

## Feature 3 — Proactive Field Completion

### Trigger

When creating or updating an Account or Activity.

### For new Accounts — ask if missing

- What industry is this company in?
- What is their company size or region?
- How did we meet them / what is the lead source?
- Who is the main contact person?

### For new Activities — ask if missing

- Who did you speak with? (Contact)
- What did they say? (Client Response)
- What's the next step? (Next Action)
- When should we follow up? (Follow-up date)

Always ask "What's the next step?" after every reported interaction. Record the answer as the Next Action field on the Activity, and update the follow-up date on the corresponding Account or Deal.

---

## Feature 4 — Pipeline Stage Discipline

### Rules

- Only advance the pipeline stage when the user explicitly confirms progress
- If the user says something that implies stage movement ("they're ready to sign", "they approved the proposal"), ask: "Should I advance them to [next stage]?"
- When a stage changes, log it in the Activity summary: "Stage advanced: [old] → [new]"
- Never backtrack a stage silently — ask before moving a deal backward

---

## Feature 5 — Query Support

### Example queries the agent must handle

- "What stage is [Company] at?"
- "What did we last talk about with [Name]?"
- "Which accounts haven't been contacted in 3+ days?"
- "How many accounts are in each stage this month?"
- "Who's our biggest open opportunity right now?"
- "Which partners had no activity last month?"
- "Show me everything in Closing stage"

### Workflow

1. Parse the query to identify: what data is needed, which database, any filters or time constraints
2. Query the appropriate database(s) using the Notion API
3. Format the answer conversationally — do not dump raw JSON
4. If the result is a list, present it clearly with the most relevant fields shown

---

## Feature 6 — Daily Morning Report

### Trigger

Scheduled cron job (default: 01:00 UTC = 09:00 Taiwan time)

### Workflow

1. Pull all Accounts and their Activities from Notion
2. Identify accounts that need follow-up:
   - The follow-up date field is today or overdue
   - No Activity logged in the past 3 days
3. For each follow-up target, generate a personalized draft message:
   - Summarize the last interaction in context
   - Suggest next action based on the current pipeline stage
   - Draft a follow-up WhatsApp or Email message
   - Save draft to Content Library (Status: Draft)
4. Send the report to Telegram with the configured chat ID and thread ID
5. Append a pipeline health summary at the end

### Report format

```
📋 Today's Follow-up List (N clients)

1️⃣ [Company] — [Contact Name] | Stage: [Stage]
Last interaction: [Date], [One-line summary]
📌 Suggested action: [Action] (follow-up draft ready)

2️⃣ [Company] — [Contact Name] | Stage: [Stage] — N days overdue
📌 Suggested action: [Action]

━━━━━━━━━━━━━━━━
📊 Pipeline Summary
Lead: N | Qualified: N | Closing: N | Won: N
⚠️ Stalled 7+ days: N accounts
```

---

## Feature 7 — Bulk Email Outreach

### Trigger

User requests a campaign, e.g.: "Send the new case study to all Qualified leads", "Run a cold outreach campaign to manufacturing companies"

### Workflow

1. Identify the target segment from the user's request (by stage, industry, region, or custom filter)
2. Pull matching Accounts and their primary contact's email from Notion
3. Personalize each message (insert name, company, and any relevant context from their record)
4. Present the full recipient list for confirmation before sending:

```
Recipients confirmed (N total):
- [Name] | [Company] | [email]
- [Name] | [Company] | [email]

Subject: [Subject]
Content summary: [One-line summary]

Confirm send?
```

5. Only send after the user explicitly confirms
6. After sending, log one Activity per Account (Type: Email, summary referencing the campaign name)
7. Do NOT send to the same contact twice for the same campaign — check recent Activities before sending

Use the email account configured in `~/.hermes/.env` (Himalaya CLI or SMTP).

---

## Feature 8 — Business Card Import

### Trigger

User sends a photo of a business card (single card or multiple)

### Single card workflow

1. Analyze the image using vision tools. Extract:
   - Name, job title, department
   - Company name, website, address
   - Email, phone, mobile
   - Any social or messaging handles (LINE, WhatsApp, LinkedIn, WeChat)

2. Present extracted info for confirmation:
```
📇 Business Card Info
Name: [Name]
Title: [Title]
Company: [Company]
Email: [Email]
Phone: [Phone]
Other: [any additional fields]
```

3. Ask context questions before importing:
   - Which Account does this person belong to? Or should we create a new one?
   - What is their decision role? (Decision Maker / Influencer / Executor)
   - Where / how did you meet them?
   - Any first interaction to log?

4. Import after the user confirms:
   - If new company → create Account first
   - Create Contact with all extracted and context fields
   - If there's an initial interaction → create Activity
   - Ask "What's the next step?" and record Next Action + follow-up date

### Batch cards (trade show / event)

When user sends multiple photos:
1. Analyze all cards and present a summary list:
```
📇 Recognition results (N business cards)
1. [Name] | [Company] | [Email]
2. [Name] | [Company] | [Email]
```
2. Ask once: "Where did you meet these people? Are they from the same event?"
3. After confirmation, create all Contacts (and Accounts if needed)
4. If from an event, create one shared Activity for all contacts referencing the event

### Front + back of the same card

- If two consecutive photos appear to be the same person (matching name or company), merge the information
- Do not create duplicate Contacts
- If both Chinese and English versions are present, use the preferred language as primary and put the other in Notes

### Important rules

- Never import to Notion without first showing a summary and asking the context questions
- If the user says "just for reference" or "don't add yet", do not write to Notion
- If the company already exists in Accounts, link to it — do not create a duplicate

---

## Behavioral Rules

1. **Confirm before changing** — always show what you're about to write/change and ask for confirmation, unless the user says "just do it" or similar
2. **Ask for missing fields** — never silently skip required fields; always ask before saving incomplete records
3. **Always ask next step** — after every logged interaction, ask "What's the next step?" and record the answer
4. **Pipeline stages on explicit confirmation only** — never advance a stage without the user saying so
5. **No duplicate records** — before creating an Account or Contact, check if it already exists
6. **Activity log is the source of truth** — every interaction goes into Activities, regardless of what else gets updated
7. **Confirm before sending any email** — show full recipient list and content summary; never send automatically

---

## Configuration

```
# Notion
NOTION_TOKEN=*** Notion integration token

# Telegram (daily report delivery)
TELEGRAM_BOT_TOKEN=*** Telegram bot token

# Email (bulk outreach)
EMAIL_FROM=                       # Sender address
EMAIL_ACCOUNT=                    # Account name in Himalaya config (if using Himalaya)
```

> All database IDs, chat IDs, report schedule, and language preference are stored in agent memory (saved during SETUP). The agent reads them from memory whenever this use case is active.

> Field names in the client's databases may differ from the recommended schema described in SETUP.md. Always check the actual property names in each database before querying or writing. Use the field names as they exist in the client's Notion workspace.
