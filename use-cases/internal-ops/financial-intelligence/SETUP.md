# Financial Intelligence — Setup Guide

Run this setup once when deploying the Financial Intelligence use case for a new client. Follow each phase in order. Do not skip ahead.

---

## Phase 1 — Discovery

Search the client's Notion workspace for existing databases that may already serve financial or accounting functions.

**Search keywords to use:**
- Chart of Accounts, COA, Accounts, Account Code
- General Ledger, GL, Journal, Journal Entry, Transactions
- Accounts Receivable, AR, Invoices, Sales Invoices, Receivables
- Accounts Payable, AP, Bills, Vendor Bills, Payables, Expenses
- Bank, Bank Feed, Bank Transactions, Cash, Cash Flow

**Property patterns that indicate a match:**
- A database with Account Name, Account Type (Revenue/Expense/Asset/Liability), and an account code field — likely the Chart of Accounts
- A database with Debit/Credit amount fields and a date — likely the General Ledger
- A database with Invoice ID, Customer Name, Due Date, and a status like Paid/Overdue — likely Accounts Receivable
- A database with vendor/payee fields, an amount, and a status like Unpaid/Paid — likely Accounts Payable
- A database with transactions grouped by bank account name and positive/negative amounts — likely Bank Feeds

**Action:** Search Notion using the keywords above. List every database found with its name, a sample of its properties, and which financial function it likely serves. Report your findings to the user before proceeding to Phase 2.

---

## Phase 2 — Onboard

Before creating any databases, ask the user the following question:

> "Do you currently use accounting software like Xero or QuickBooks? If so, would you like to mirror that data into Notion so the AI can query it alongside your other Notion data, or would you prefer to use Notion as a standalone financial system without syncing from external software? Either approach works — I just need to know which path to take."

Wait for the user's answer and document their choice. The rest of setup proceeds the same way regardless of the answer, but note the approach chosen in the configuration summary at the end.

Review the discovery findings. For each of the 5 required databases, determine whether an existing database can serve that role or a new one must be created.

**The 5 required databases:**
1. Chart of Accounts (COA)
2. General Ledger (GL)
3. Accounts Receivable (AR)
4. Accounts Payable (AP)
5. Bank Feeds

**Note on structure:** These 5 tables mirror the architecture of real accounting systems. The COA serves as the classification backbone — Account IDs in GL, AR, AP, and Bank Feeds all reference COA entries.

For each missing database, show the user the recommended schema from the section below and ask: "Would you like me to create this database via the Notion API, or would you prefer to create it manually?"

- If creating via API: use POST /databases to create the database, then tell the user: "Please open Notion, find the newly created database, and share it with the Hermes integration so I can access it."
- If creating manually: walk the user through the field list and confirm when done.

---

## Phase 3 — Adapt

For each database that was found or just created, fetch its current schema using GET /databases/{id}.

Compare the actual property names and types against the recommended schema below. For any recommended field that is missing:

1. Show the user the gap: "The Accounts Receivable database is missing a 'Paid Amount' number field for tracking partial payments. Would you like me to add it?"
2. After confirmation, use PATCH /databases/{id} to add the missing field.

**Rules:**
- Never delete or rename fields that already exist in the client's database
- Never change a field type without explicit user confirmation
- If the client has a field that serves the same purpose under a different name, note it and move on — do not duplicate it

---

## Phase 4 — Configure

Once all databases are confirmed and accessible, save their IDs to `~/.hermes/.env`.

Retrieve each database ID from the Notion URL (the 32-character hex string after the last `/`).

Append to `~/.hermes/.env`:

```
NOTION_TOKEN=<value>
FINANCE_COA_DB=<Chart of Accounts database ID>
FINANCE_GL_DB=<General Ledger database ID>
FINANCE_AR_DB=<Accounts Receivable database ID>
FINANCE_AP_DB=<Accounts Payable database ID>
FINANCE_BANK_DB=<Bank Feeds database ID>
FINANCE_TELEGRAM_CHAT_ID=<Telegram chat ID for automated reports>
FINANCE_TELEGRAM_THREAD_ID=<Telegram thread/topic ID>
```

After saving, confirm with a summary:

> "Financial Intelligence setup is complete.
>
> Accounting approach: [Notion standalone / Mirroring from Xero / Mirroring from QuickBooks]
>
> The following databases are configured:
> - Chart of Accounts: [ID]
> - General Ledger: [ID]
> - Accounts Receivable: [ID]
> - Accounts Payable: [ID]
> - Bank Feeds: [ID]
>
> Telegram reporting: Chat [chat ID], Thread [thread ID]
>
> All values saved to ~/.hermes/.env. You can now load SKILL.md to activate financial intelligence operations."

---

## Recommended Schema

These are suggestions based on standard accounting structure. Adapt to what the client already has — field names and option values may differ. Do not force the client to match this exactly.

### Chart of Accounts (COA)

| Field | Type | Notes |
|-------|------|-------|
| Account Name | Title | e.g. Consulting Revenue, Office Rent |
| Account ID | Rich Text | e.g. 4100, 6200 |
| Account Type | Select | Revenue / Expense / Asset / Liability / Equity |

Common categories: Revenue (4xxx), Expense (5xxx–6xxx), Asset (1xxx), Liability (2xxx)

### General Ledger (GL)

| Field | Type | Notes |
|-------|------|-------|
| Journal ID | Title | e.g. J-2026-0001 |
| Date | Date | |
| Account ID | Rich Text | Maps to COA Account ID |
| Description | Rich Text | |
| Debit Amount | Number | |
| Credit Amount | Number | |

P&L is aggregated from this table: Revenue = sum of Credit for Revenue accounts; Expenses = sum of Debit for Expense accounts.

### Accounts Receivable (AR)

| Field | Type | Notes |
|-------|------|-------|
| Invoice ID | Title | e.g. INV-2026-001 |
| Customer Name | Rich Text | |
| Category | Select | e.g. Product Sales / Consulting / Other |
| Issue Date | Date | |
| Due Date | Date | |
| Total Amount | Number | |
| Paid Amount | Number | For partial payments |
| Status | Select | Sent / Paid / Overdue / Cancelled |

### Accounts Payable (AP)

| Field | Type | Notes |
|-------|------|-------|
| Bill ID | Title | e.g. BILL-2026-001 or EXP-2026-001 |
| Vendor / Employee | Rich Text | Vendor name or employee name |
| Category | Rich Text | Expense category (maps to COA) |
| Account ID | Rich Text | COA account code |
| Due Date | Date | |
| Amount | Number | |
| Status | Select | Unpaid / Awaiting Approval / Paid / Cancelled |

### Bank Feeds

| Field | Type | Notes |
|-------|------|-------|
| Transaction ID | Title | e.g. TXN-2026-001 |
| Bank Account | Select | e.g. HSBC-HKD Checking |
| Date | Date | |
| Amount | Number | Positive = inflow, Negative = outflow |
| Payee | Rich Text | Counterparty name |
| Category | Select | Corresponding COA account name |
| Reconciled | Checkbox | Whether reconciled against bank statement |

Cash balance calculation: GROUP BY Bank Account, SUM(Amount) = current balance per account.
