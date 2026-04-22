---
name: financial-intelligence
description: >
  AI-powered financial intelligence skill for Hermes Agent. Answers real-time
  financial questions in plain language, delivers automated weekly/monthly health
  analyses, and generates monthly and quarterly financial reports — all grounded
  in 5 Notion financial databases.
version: 1.1.0
author: BusyCow
tags: [Finance, Financial Analysis, Notion, Internal Ops, Cashflow, P&L]
---

# Financial Intelligence — Daily Operations

> Setup is assumed complete. All 5 databases are configured in agent memory from the SETUP phase. Do not run setup steps from here.

> Field names may differ from recommended schema. Always check actual field names when querying or writing.

---

## Overview

Three features, one financial nervous system:

1. **Real-Time Financial Q&A** — Ask any financial question in plain language; the agent queries all five tables and answers with precise figures
2. **Scheduled Health Analysis** — Weekly cash snapshot (every Monday) and monthly health report (1st of month), delivered to Telegram automatically without needing to ask
3. **Financial Report Generation** — P&L statements, AR aging analysis, and cash flow summaries — generated on demand or on schedule

Core principle: All numbers must come from the data tables — no estimation or fabrication. If data is insufficient to answer a question, state clearly which table is missing what data.

---

## Feature 1 — Real-Time Financial Q&A

### Triggers
- "How much money do we have right now? How much is in each account?"
- "After deducting next week's payments, how much do we actually have available?"
- "Which invoices are overdue and haven't been collected yet?"
- "What are the largest expense categories this month?"
- "Has the [customer] invoice been paid yet?"

### Workflow

**Cash balance:**
1. POST /databases/{bank feeds database (from memory)}/query — retrieve all transactions
2. GROUP BY bank account field, SUM the amount field per account
3. Present each account and currency separately — never convert across currencies

**Available cash after deductions:**
1. Calculate current balance per account (as above)
2. POST /databases/{accounts payable database (from memory)}/query — filter by status = Unpaid/Awaiting Approval AND due date <= end of this week
3. Subtract pending AP from the relevant bank account balance

**Overdue receivables:**
1. POST /databases/{accounts receivable database (from memory)}/query — filter: status = Overdue OR (due date < today AND status != Paid)
2. List: customer name, invoice ID, amount, days overdue

**Expense breakdown:**
1. POST /databases/{general ledger database (from memory)}/query — filter by date range, join to COA by account ID field
2. GROUP BY account name, SUM debit amount field
3. Sort descending by total

### Response Format

```
💰 Cash Status (as of today)

[Bank Account Name]: [Currency] [Amount]
[Bank Account Name]: [Currency] [Amount]

⚠️ AP due this week: [Currency] [Amount] ([vendor] [bill ID])
Available [currency] after deduction: [Currency] [Amount]
```

---

## Feature 2 — Scheduled Health Analysis

These reports run automatically on a cron schedule. Do not wait for user prompting. Query the data and output immediately.

### Weekly Snapshot

**Schedule:** Every Monday at 09:00 local time (cron: `0 1 * * 1` UTC)

**Steps:**
1. Query Bank Feeds — calculate current balance per account
2. Query AR — find new invoices issued this week; find overdue uncollected invoices
3. Query AP — find payments due this week

**Output format:**
```
📊 Weekly Financial Snapshot ([date])

💵 Cash Position
[account]: [currency] [balance]

📥 Receivables Update
• New this week: [Invoice ID] [Customer] [Amount]
• Overdue uncollected: [list]

📤 Payables This Week
• Due: [Bill ID] [Vendor] [Amount] due [date]

⚠️ Alerts: [anomalies if any, otherwise omit]
```

### Monthly Health Report

**Schedule:** 1st of each month at 09:00 local time (cron: `0 1 1 * *` UTC), covering the previous month

**Steps:**
1. Query GL for last month — calculate total revenue (credit amount for revenue accounts) and total expenses (debit amount for expense accounts)
2. Compute gross profit (revenue minus cost of goods sold) and gross margin
3. Query Bank Feeds for last month — calculate net cash flow (inflows minus outflows)
4. Query AP — identify last month's unpaid bills
5. Compare with the month before to determine trend (↑ ↓ →)

**Output format:**
```
📋 Financial Health Report — [YYYY-MM]

💹 P&L Summary
Revenue: [amount] | Expenses: [amount] | Gross Profit: [amount] (Gross Margin [X]%)
Trend: vs. prior month [↑↓→] [X]%

💵 Cash Flow
Opening Balance: [amount] → Closing Balance: [amount] (Net Change: [amount])

⚠️ Anomaly Alerts
[Expense overrun / Overdue receivables / Unpaid bills — list specifics]

📌 Points to Watch
[1–2 specific observations or recommendations]
```

---

## Feature 3 — Financial Report Generation

### Triggers
- "Generate the [month] P&L statement"
- "AR aging analysis — list all invoices overdue more than 30 days"
- "Cash flow summary for [month or quarter]"
- "Gross profit comparison between Q1 and Q2"

### P&L Report

1. POST /databases/{general ledger database (from memory)}/query — filter by date range, join to COA
2. Group: Revenue accounts → sum credit amount field; Expense accounts → sum debit amount field
3. Compute: Gross Profit = Revenue − COGS; Net Profit = Revenue − Total Expenses

**Output format:**
```
P&L Statement — [YYYY-MM]

📈 Revenue
  [Account Name]: [amount]
  ──────────────
  Total Revenue: [amount]

📉 Costs & Expenses
  [Account Name]: [amount]
  ──────────────
  Total Expenses: [amount]

💰 Gross Profit: [amount] (Gross Margin [X]%)
💰 Net Profit: [amount] (Net Margin [X]%)
```

### AR Aging Analysis

1. POST /databases/{accounts receivable database (from memory)}/query — filter: status not Paid and not Cancelled
2. Calculate days overdue = today − due date field
3. Bucket by aging band

**Output format:**
```
AR Aging Analysis (as of [date])

  0–30 days: [list]
  31–60 days: [list] ⚠️
  60+ days: [list] 🔴

Total Outstanding: [amount]
Recommend follow-up: [customers overdue 30+ days]
```

### Cash Flow Summary

1. POST /databases/{bank feeds database (from memory)}/query — filter by date range
2. Identify opening balance, inflows, outflows, closing balance

**Output format:**
```
Cash Flow Summary — [period]

Opening Balance: [amount]
  (+) Receipts: [key items]
  (-) Payments: [key items]
Closing Balance: [amount]
Net Change: [amount]
```

---

## Behavioral Rules

1. **Numbers must come from data** — Never estimate or fabricate financial figures. If data is insufficient, state which table is missing what field.
2. **Currencies presented separately** — Multi-currency accounts must not be cross-converted or combined. List each currency independently.
3. **Confirm time range** — If a question does not specify a time range, ask "Do you mean this month or last month?" before querying.
4. **Proactively flag anomalies** — If overdue receivables, unpaid bills, or expense overruns are detected during any query, always append a ⚠️ alert at the end of the response.
5. **Scheduled reports run automatically** — Cron jobs do not require user confirmation. Query the data and output the report immediately.
6. **On-demand reports output completely** — When a report is requested, query the full dataset and deliver the complete formatted output in one response.

---

## Configuration

Databases and delivery config are configured in agent memory from the SETUP phase. The following values are used:

```
# Finance Databases (read from agent memory)
coa_db              # Chart of Accounts
gl_db               # General Ledger
ar_db               # Accounts Receivable
ap_db               # Accounts Payable
bank_db             # Bank Feeds

# Telegram reporting (read from agent memory)
report_chat_id      # Chat ID for weekly/monthly automated reports
report_thread_id    # Thread/topic ID for finance reports
```
