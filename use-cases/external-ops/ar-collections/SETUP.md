# AR Collections — Setup Guide

This guide walks through onboarding a new client for the AR Collections use case. Follow the phases in order.

---

## Phase 1: Discovery

Before creating anything, check if the client already has relevant data in Notion.

Search the client's Notion workspace for pages or databases with these keywords:

- English: `invoice`, `receivable`, `AR`, `debt`, `collection`, `overdue`
- Chinese: `欠款`, `應收`, `催收`, `逾期`, `發票`

For each match found:
- Note the database/page ID
- Review the field structure
- Ask the client: "Is this your current AR tracking? Do you want to migrate this or start fresh?"

If existing data is found, map client fields to the recommended schema before proceeding. If none found, proceed directly to Phase 2.

---

## Phase 2: Onboard — Create Data Tables

Create tables in this order (dependencies must exist before relations can be set).

### Step 1: Debtor List (no dependencies)

**Recommended schema:**

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name / Company | Title | Debtor's full name or company name |
| Debtor Type | Select | Corporate, Individual |
| Debt Nature | Select | Invoice, Loan, Salary Overpayment, Other |
| Case Status | Select | Active Reminder, Overdue Escalation, Negotiating Installment, Settled, Legal Action |
| Collection Difficulty | Select | Easy, Normal, Hard, High Risk |
| Suggested Tone | Select | Gentle, Firm |
| Contact Person | Rich Text | Name of contact at the debtor entity |
| Email | Email | Primary contact email for collection messages |
| Phone | Phone | Primary contact phone |
| Notes | Rich Text | Free-form notes on the case |

### Step 2: AR Records (relates to Debtor List)

**Recommended schema:**

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Invoice / Claim Name | Title | Descriptive name of the debt item |
| Debtor | Relation | → Debtor List |
| Amount Due | Number | Total amount originally owed |
| Amount Received | Number | Total received to date |
| Outstanding Amount | Formula | Amount Due - Amount Received |
| Due Date | Date | Original payment due date |
| Issue Date | Date | Date the invoice or loan was issued |
| Days Overdue | Number | Updated daily by the agent |
| Collection Stage | Select | Pre-due Reminder, Friendly Follow-up, Formal Notice, Final Warning, Escalation, Settled |
| Next Follow-up Date | Date | Set automatically by the agent after each action |
| Status | Select | Outstanding, Partially Paid, Settled, Written Off |
| Notes | Rich Text | Case notes |

### Step 3: Collection Log (relates to Debtor List)

**Recommended schema:**

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Log Title | Title | Auto-generated: [Debtor] - [Date] - [Stage] |
| Debtor | Relation | → Debtor List |
| Date | Date | Date of the collection action |
| Channel | Select | Email, Phone, SMS |
| Collection Stage | Select | Matches stage at time of action |
| Message Content | Rich Text | Full text of message sent or call summary |
| Debtor Response | Rich Text | What the debtor said or did |
| Promise Date | Date | If debtor committed to a payment date |

### Step 4: Collection Message Templates (standalone page)

Check if a Notion page already exists matching: `template`, `collection message`, `催款`

If found: review the structure and confirm with the client whether to use it or replace it.

If not found: offer to create a new page with the following structure:

```
Collection Message Templates
├── Corporate Debtors
│   ├── Pre-due Reminder (ZH + EN)
│   ├── Formal Notice (ZH + EN)
│   └── Final Warning (ZH + EN)
└── Individual Debtors
    ├── Pre-due Reminder (ZH + EN)
    ├── Formal Notice (ZH + EN)
    └── Final Warning (ZH + EN)
```

Each template section should include:
- Placeholder tokens: `{{debtor_name}}`, `{{outstanding_amount}}`, `{{due_date}}`, `{{invoice_ref}}`
- Tone guidance note (Gentle or Firm)
- Chinese version + English version

---

## Phase 3: Configure

Once all tables and the templates page exist, save their IDs to agent memory.

Ask the client:
- "Which Telegram chat/thread should escalation alerts go to?"

Then save the following block to agent memory under the key **"AR Collections config"**:

```
AR Collections config:
  debtor_list_db: <notion_database_id>
  ar_records_db: <notion_database_id>
  collection_log_db: <notion_database_id>
  collection_templates_page: <notion_page_id>
  alert_chat_id: <telegram_chat_id>
  alert_thread_id: <telegram_thread_id>
  daily_schedule: 0 1 * * *
```

**daily_schedule** uses cron format. Default is `0 1 * * *` (1:00 AM daily). Adjust per client timezone/preference.

---

## Phase 4: Verify

Run a dry-run check before going live:

1. Add one test debtor to Debtor List
2. Add one test AR Record with a due date in the past (to trigger Friendly Follow-up stage)
3. Ask the agent: "Run collection check for [test debtor name]"
4. Verify it: reads the AR Record correctly, selects the right template, shows a draft message for confirmation
5. Do NOT send — confirm the logic is correct, then delete the test records

---

## Phase 5: Go Live

Inform the client:
- The daily schedule will run automatically at the configured time
- Routine reminder emails are sent without asking for confirmation
- Escalation alerts will arrive via Telegram — reply with a decision
- They can query the portfolio anytime by asking the agent
