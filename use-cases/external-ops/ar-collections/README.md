# AR Collections (欠款自動催收)

**Category:** External Ops
**Complexity:** High
**Trigger:** Scheduled daily + on-demand query

---

## What This Does

AR Collections automates the entire debt collection SOP for SMEs. Most SMEs fail at collections not because they don't care, but because the process is exhausting — knowing who to chase, when to escalate, and what to say takes time and emotional energy that most teams don't have.

This use case runs a daily background schedule that tracks every debtor, triggers the right message at the right time, and only surfaces to the owner when a decision is genuinely needed.

---

## Core Value

- No one falls through the cracks — every AR record is tracked daily
- The right message, at the right time, in the right tone — matched by debtor type and collection stage
- Owner only intervenes at escalation decisions — not routine follow-ups
- Full audit trail — every action, response, and promise date is logged

---

## Collection Rhythm

The engine of this use case. Runs daily, triggered by days overdue relative to each AR Record's due date.

| Days Relative to Due Date | Stage               | Action                                      |
|---------------------------|---------------------|---------------------------------------------|
| -30 days (before due)     | Pre-due Reminder    | Send gentle payment reminder                |
| -7 days (before due)      | Payment Confirmation| Send confirmation nudge                     |
| +1 to +7 days overdue     | Friendly Follow-up  | Friendly but direct follow-up               |
| +8 to +30 days overdue    | Formal Notice       | Formal tone, reference invoice/agreement    |
| +31 to +60 days overdue   | Final Warning       | Firm language, reference legal consequences |
| +60 days overdue          | Escalation          | Push alert to owner for decision            |
| Promise date passed       | Promise Follow-up   | Trigger follow-up or escalation alert       |

---

## Features

### 1. Daily Collection Schedule
Runs automatically every day (default: 1:00 AM). Scans all AR Records, calculates days overdue, updates Collection Stage, and triggers the appropriate action — send email, log activity, or push an alert to the owner. No manual input required for routine reminders.

### 2. Message Template Matching
Reads the debtor's type (Corporate / Individual) and current collection stage. Selects the matching template from the Collection Message Templates page, personalizes it with the debtor's name, outstanding amount, and relevant dates, then sends via email and logs the action in the Collection Log.

### 3. Conversation & Response Logging
Log any interaction in plain language — "Called Wang Dawen today, he said he'll pay by end of month." The agent structures it into a proper Collection Log entry with debtor relation, channel, stage, response, and promise date.

### 4. Escalation Alerts
Automatically pushes to the owner (via Telegram) when:
- AR Record is overdue >60 days with no response
- High-amount debtor has not responded across multiple stages
- A promise date passes without the status changing to Settled

### 5. Portfolio Overview
Answer on-demand questions:
- "Who owes us money right now?"
- "Which debtors are overdue by more than 30 days?"
- "How much did we collect this month?"
- "Which cases have had no contact in the past 2 weeks?"

---

## Recommended Data Tables

See [SETUP.md](./SETUP.md) for full schema and setup instructions.

1. **Debtor List** — one row per debtor; all AR Records and Collection Logs relate back to this
2. **AR Records** — one row per invoice or debt; tracks amount, due date, stage, and next follow-up
3. **Collection Log** — one row per collection action; full audit trail of messages, responses, and promises
4. **Collection Message Templates** — standalone Notion page (not a database); 6 template groups by debtor type × stage, Chinese + English versions

---

## Files

| File | Purpose |
|------|---------|
| `README.md` | Overview, features, collection rhythm |
| `SETUP.md` | Step-by-step onboarding for a new client |
| `SKILL.md` | Agent behavioral rules and decision logic |
| `config-template/env-template.txt` | Environment variable template |
