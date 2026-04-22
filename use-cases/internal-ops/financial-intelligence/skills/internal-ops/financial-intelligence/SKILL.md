---
name: financial-intelligence
description: >
  AI-powered financial intelligence skill for Hermes Agent. Answers real-time
  financial questions in plain language, delivers automated weekly/monthly health
  analyses, and generates monthly and quarterly financial reports — all grounded
  in 5 Notion financial databases.
version: 1.0.0
author: BusyCow
tags: [Finance, Financial Analysis, Notion, Internal Ops, Cashflow, P&L]
---

# Financial Intelligence

## Overview

Three features, one financial nervous system:

1. **Real-Time Financial Q&A** — Ask any financial question in plain language; the AI queries all five tables and answers instantly
2. **Scheduled Health Analysis** — Weekly snapshot + monthly health report, proactively delivered without needing to ask
3. **Financial Report Generation** — P&L, cash flow summary, AR aging analysis — generated on demand or on schedule

Core principle: **All numbers must come from the data tables — no estimation or fabrication**. If data is insufficient to answer a question, state clearly which table is missing what data.

---

## Notion Databases Required

5 financial data tables, mirroring the underlying architecture of real accounting systems (Xero / QuickBooks / Dingxin).

| Table | Abbreviation | Purpose |
|--------|------|------|
| Chart of Accounts (COA) | COA | Classification backbone for all accounts |
| General Ledger (Journal Entries) | GL | Bottom-level transaction journal |
| Accounts Receivable (Sales Invoices) | AR | Receivables and invoice status |
| Accounts Payable (Bills & Expenses) | AP | Payables and expense claims |
| Bank Feeds (Cash Transactions) | Bank | Actual bank account transactions |

---

## Schema

### 1. Chart of Accounts (COA)

Classification backbone. The Account_ID in all other tables maps here.

| Field | Type | Notes |
|-------|------|-------|
| Account_Name | Title | e.g. Consulting Revenue, Office Rent |
| Account_ID | Rich Text | e.g. 4100, 6200 |
| Account_Type | Select | **Revenue / Expense / Asset / Liability / Equity** |

Common account categories:
- Revenue (4xxx): Product Sales Revenue, Consulting Revenue
- Expense (5xxx–6xxx): Cost of Goods Sold (5000), Payroll Expense (6100), Office Rent (6200), Travel Expenses (6300)
- Asset (1xxx): Bank Deposits (1100), Accounts Receivable (1200)
- Liability (2xxx): Accounts Payable (2100)

---

### 2. Accounts Receivable / Sales Invoices (AR)

Each customer receivable invoice. Query this table when asked "who owes us money?"

| Field | Type | Notes |
|-------|------|-------|
| Invoice_ID | Title | Format: INV-YYYY-NNN |
| Customer_Name | Rich Text | |
| Category | Select | Product Sales Revenue / Consulting Revenue / Other |
| Issue_Date | Date | Invoice issue date |
| Due_Date | Date | Due date |
| Total_Amount | Number | Invoice amount |
| Paid_Amount | Number | Amount received (for partial payments) |
| Status | Select | **Sent / Paid / Overdue / Cancelled** |

Overdue logic: Due_Date < TODAY AND Status != Paid → Overdue

---

### 3. Accounts Payable / Bills & Expenses (AP)

Each outgoing payment. Query this table when asked "how much do we need to pay next week?"

| Field | Type | Notes |
|-------|------|-------|
| Bill_ID | Title | Format: BILL-YYYY-NNN or EXP-YYYY-NNN |
| Vendor_Employee | Rich Text | Vendor name or employee name |
| Category | Rich Text | Expense category (maps to COA) |
| Account_ID | Rich Text | COA account code |
| Due_Date | Date | |
| Amount | Number | |
| Status | Select | **Unpaid / Awaiting Approval / Paid / Cancelled** |

---

### 4. Bank Feeds / Cash Transactions (Bank)

Every bank account transaction. Query this table when asked "how much is left in the account?"

| Field | Type | Notes |
|-------|------|-------|
| Transaction_ID | Title | Format: TXN-YYYY-NNN or OPEN-YYYY-NNN |
| Bank_Account | Select | Bank account name (e.g. HSBC-HKD Checking) |
| Date | Date | |
| Amount | Number | Positive = inflow, Negative = outflow |
| Payee | Rich Text | Transaction counterparty |
| Category | Select | Corresponding COA account name |
| Reconciled | Checkbox | Whether reconciled |

Cash balance calculation: GROUP BY Bank_Account, SUM(Amount) = current balance

---

### 5. General Ledger / Journal Entries (GL)

Bottom-level double-entry bookkeeping journal. P&L is aggregated from this table.

| Field | Type | Notes |
|-------|------|-------|
| Journal_ID | Title | Format: J-YYYY-NNNN |
| Date | Date | |
| Account_ID | Rich Text | Maps to COA |
| Description | Rich Text | Transaction description |
| Debit_Amount | Number | Debit amount |
| Credit_Amount | Number | Credit amount |

P&L calculation:
- Revenue = Total Credit_Amount for Revenue accounts
- Expenses = Total Debit_Amount for Expense accounts
- Gross Profit = Revenue − Cost of Goods Sold (5000) Debit
- Net Profit = Revenue − Total Expense Debit

---

## Feature 1 — Real-Time Financial Q&A

### Conversational Triggers

- "How much money do we have right now? How much is in each account?"
- "After deducting next week's payments, how much do we actually have available?"
- "The HKD 150,000 invoice from Star-O Technology is due — has it been received?"
- "What are the largest expense categories this month?"
- "Which invoices are overdue and haven't been collected?"

### Query Logic

**Cash Balance:**
```
Bank Feeds → GROUP BY Bank_Account → SUM(Amount) per account
Response format:
HSBC-HKD Checking: HKD 615,000
Cathay United-TWD Account: TWD 420,000
Citibank-USD Account: USD 420,000 (note: different currencies)
Total available cash: [list by currency — no cross-currency conversion]
```

**Available Cash After Deducting Pending Payments:**
```
Bank Feeds SUM - AP WHERE Status=Unpaid AND Due_Date <= end of this week SUM
```

**Overdue Receivables Query:**
```
AR WHERE Status=Overdue OR (Due_Date < TODAY AND Status != Paid)
→ List: Customer name, Invoice_ID, amount, days overdue
```

**Expense Account Analysis:**
```
GL WHERE Account_Type=Expense → JOIN COA → GROUP BY Account_Name → SUM(Debit_Amount)
→ Sort by amount descending
```

### Response Format

```
💰 Cash Status (as of today)

HSBC-HKD Checking: HKD 615,000
Cathay United-TWD Account: TWD 420,000
Citibank-USD Account: USD 420,000

⚠️ AP due next week: HKD 180,000 (Lian-O Computer BILL-2026-001, equipment purchase)
Available HKD after deduction: HKD 435,000
```

---

## Feature 2 — Scheduled Health Analysis

### Weekly Snapshot — Cron Setup

Every Monday at 9am, delivered alongside the Daily Report.

```
Cron schedule: 0 1 * * 1  (Monday 01:00 UTC = 09:00 TWN)

Prompt:
You are a financial analysis assistant. Execute the following steps and output in plain text:

1. Query Bank Feeds — calculate current balance for each account
2. Query AR — identify new invoices this week and overdue uncollected amounts
3. Query AP — identify payments due this week
4. Output format:

📊 Weekly Financial Snapshot (YYYY-MM-DD)

💵 Cash Position
[balance per account]

📥 Receivables Update
• New this week: [Invoice_ID] [Customer] [Amount]
• Overdue uncollected: [list Overdue items]

📤 Payables This Week
• Due for payment: [list AP items with Due_Date this week]

⚠️ Alerts: [anomalies, if any]
```

### Monthly Health Report — Cron Setup

Delivered on the 1st of each month at 9am, covering the previous month's health.

```
Cron schedule: 0 1 1 * *  (1st of month, 01:00 UTC)

Prompt:
You are a financial analysis assistant. Analyze the financial status for last month (YYYY-MM) and output a health report:

1. Query GL (last month) and calculate:
   - Total Revenue (sum of Credit_Amount for Revenue accounts)
   - Total Expenses (sum of Debit_Amount for Expense accounts)
   - Gross Profit = Revenue - COGS (5000)
   - Gross Margin = Gross Profit / Revenue
2. Query Bank Feeds (last month) — calculate net cash flow (inflows minus outflows)
3. Query AP — identify last month's unpaid bills (Status=Unpaid/Awaiting Approval)
4. Compare with the month before last to determine trend (↑ ↓ →)

Output format:
📋 Financial Health Report — YYYY-MM

💹 P&L Summary
Revenue: XXX | Expenses: XXX | Gross Profit: XXX (Gross Margin XX%)
Trend: vs. prior month ↑↓→ XX%

💵 Cash Flow
Opening Balance: XXX → Closing Balance: XXX (Net Change: XXX)

⚠️ Anomaly Alerts
[Expense overrun / Overdue receivables / Unpaid bills / etc.]

📌 Points to Watch
[1–2 specific recommendations]
```

---

## Feature 3 — Financial Report Generation

### Conversational Triggers

- "Generate the March P&L statement for me"
- "Show me the cash flow summary for this quarter"
- "AR aging analysis — list all invoices uncollected for more than 30 days"
- "Gross profit comparison between Q1 and Q2"

### P&L Report

```
Query GL WHERE Date in [month range] → JOIN COA
→ Group and calculate:
  Revenue accounts: sum of Credit_Amount → revenue per account
  COGS (5000): sum of Debit_Amount
  Operating Expenses (6xxx): list Debit totals by account

Output:
P&L Statement — YYYY-MM

📈 Revenue
  Product Sales Revenue (4000): XXX
  Consulting Revenue (4100): XXX
  ──────────────
  Total Revenue: XXX

📉 Costs & Expenses
  Cost of Goods Sold (5000): XXX
  Payroll Expense (6100): XXX
  Office Rent (6200): XXX
  Travel Expenses (6300): XXX
  ──────────────
  Total Expenses: XXX

💰 Gross Profit: XXX (Gross Margin XX%)
💰 Net Profit: XXX (Net Margin XX%)
```

### AR Aging Analysis

```
Query AR WHERE Status != Paid AND Status != Cancelled
Calculate days overdue = TODAY - Due_Date

Output:
AR Aging Analysis (as of YYYY-MM-DD)

  0–30 days: [list]
  31–60 days: [list] ⚠️
  60+ days: [list] 🔴

Total Outstanding: XXX
Recommend immediate follow-up: [customers overdue 30+ days]
```

### Cash Flow Summary

```
Query Bank Feeds WHERE Date in [period]
→ Categorize: Opening Balance, Inflows, Outflows, Closing Balance

Output:
Cash Flow Summary — YYYY-MM

Opening Balance: XXX
  (+) Receipts: [key items]
  (-) Payments: [key items]
Closing Balance: XXX
Net Change: XXX
```

---

## Behavioral Rules

1. **Numbers must come from data** — Never estimate or fabricate financial figures; if data is insufficient, specify which table and which field is missing
2. **Currencies presented separately** — Multi-currency accounts must not be cross-converted; list each currency independently
3. **Confirm time range** — If a question does not specify a time range, first ask "Do you mean this month or last month?" before querying
4. **Proactively flag anomalies** — If overdue receivables, unpaid bills, or expense overruns are detected during a query, always append a ⚠️ alert at the end of the response
5. **Weekly/monthly reports run automatically** — Cron jobs do not require user triggering; query data and output directly without asking for confirmation
6. **Reports generated on demand** — When a report is requested, query the complete dataset and output all at once in a clean, organized format

---

## Configuration

```
# Notion
NOTION_TOKEN=*** Notion integration token

# Finance Databases
FINANCE_COA_DB=                       # Chart of Accounts DB ID
FINANCE_GL_DB=                        # General Ledger DB ID
FINANCE_AR_DB=                        # Accounts Receivable DB ID
FINANCE_AP_DB=                        # Accounts Payable DB ID
FINANCE_BANK_DB=                      # Bank Feeds DB ID

# Telegram (for weekly/monthly reports)
FINANCE_TELEGRAM_CHAT_ID=             # e.g. -100XXXXXXXXX
FINANCE_TELEGRAM_THREAD_ID=           # Thread/topic ID for finance reports
```
