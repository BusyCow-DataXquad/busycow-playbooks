# Internal Ops Data Foundation

Shared database schema for all Internal Ops use cases.
Install only the tables required by the use cases you've selected.

---

## Dependency Map

| Use Case | Required Tables |
|----------|----------------|
| HR Management | Employee Directory, Attendance Records, Leave Requests, Annual Leave, Payroll Records, Expense Claims |
| Financial Intelligence | Chart of Accounts, General Ledger, Accounts Receivable, Accounts Payable, Bank Feeds |
| Discussion & Todo Tracker | Internal Discussions, Task Tracker |

---

## Data Architecture — HR Module

```
┌─────────────────────────────────────────────────────┐
│  MASTER RECORD — Who exists                         │
│  Employee Directory (employee profiles, status, tenure) │
└───┬───────────────┬──────────────┬──────────────────┘
    │               │              │
┌───▼───────┐ ┌────▼──────┐ ┌────▼──────────────────┐
│ Attendance │ │  Leave    │ │  Payroll Records       │
│ Records   │ │ Requests  │ │ (monthly payroll       │
│ (daily    │ │ (leave    │ │ breakdown + status)    │
│ clock-in  │ │ requests  │ └────────────────────────┘
│ overtime) │ │ approval) │
└───────────┘ └────▼──────┘
                   │
         ┌─────────▼────────┐  ┌────────────────────────┐
         │  Annual Leave    │  │  Expense Claims        │
         │ (annual quota +  │  │ (expense claims +      │
         │  balance per yr) │  │  reimbursement flow)   │
         └──────────────────┘  └────────────────────────┘
```

**Core Design Principles:**
- Approval status is embedded directly in Leave Requests and Expense Claims — no separate approval table needed
- Employee Directory is the relation anchor for all 5 other tables — create it first
- Annual Leave has one row per employee per year; HR sets the quota manually and the system tracks the balance

---

## Tables

### Employee Directory (Master Record)

The anchor table. All other tables relate back to this one.

| Field | Type | Notes |
|-------|------|-------|
| Employee Name | Title | Full name |
| English Name | Rich Text | |
| Employee ID | Rich Text | Format: EMP-001 |
| Department | Select | Sales / Engineering / Product / Design / HR / Operations / Finance / Other |
| Job Title | Rich Text | |
| Start Date | Date | |
| End Date | Date | Leave blank for active employees |
| Status | Select | Active / Probation / Leave of Absence / Resigned |
| Notes | Rich Text | Free-form notes |

---

### Attendance Records

Daily clock-in/out and overtime records. One row per employee per day (for days with notable records).

| Field | Type | Notes |
|-------|------|-------|
| Attendance Record | Title | Format: A-YYYY-MMDD-[surname] |
| Employee | Relation → Employee Directory | Enable dual property |
| Clock-In Time | Date (datetime) | |
| Clock-Out Time | Date (datetime) | |
| Total Overtime Hours | Number | Unit: hours |
| Overtime Reason | Rich Text | Required if overtime > 0 |

---

### Leave Requests

Leave applications with embedded approval status. One row per leave request.

| Field | Type | Notes |
|-------|------|-------|
| Leave Request | Title | Format: L-YYYY-NNNN [name] [leave-type] |
| Employee | Relation → Employee Directory | Enable dual property |
| Leave Type | Select | Annual / Paid Leave / Sick / Personal / Public Duty / Compensatory / Other |
| Start Date | Date | |
| End Date | Date | |
| Days | Number | Supports 0.5 |
| Reason | Rich Text | |
| Status | Select | **Pending / Approved / Rejected / Cancelled** |

> Approval happens by updating the Status field — no separate approval table needed.

---

### Annual Leave

Annual leave quota and balance per employee per year. One row per employee per year.

| Field | Type | Notes |
|-------|------|-------|
| Annual Leave Record | Title | Format: YYYY-[name] |
| Employee | Relation → Employee Directory | |
| Year | Rich Text | e.g. "2026" |
| Total Annual Leave Days | Number | Set by HR at year start |
| Used Days | Number | Update when leave is approved |
| Remaining Days | Number | = Total Days − Used Days |
| Notes | Rich Text | e.g. "On leave of absence" |

---

### Payroll Records

Monthly payroll breakdown and disbursement status. One row per employee per month.

| Field | Type | Notes |
|-------|------|-------|
| Payroll Record | Title | Format: P-YYYY-MM-[name] |
| Employee | Relation → Employee Directory | |
| Month | Rich Text | Format: YYYY-MM |
| Base Salary | Number | |
| Overtime Pay | Number | |
| Allowances | Number | |
| Deductions | Number | |
| Net Pay | Number | = Base Salary + Overtime Pay + Allowances − Deductions |
| Disbursement Status | Select | **Paid / Pending / Awaiting Confirmation** |

---

### Expense Claims

Employee expense claims with full reimbursement workflow. One row per claim.

| Field | Type | Notes |
|-------|------|-------|
| Expense Item | Title | Descriptive name of the expense |
| Employee | Relation → Employee Directory | |
| Category | Select | Travel / Transportation / Meals / Accommodation / Software/Subscriptions / Office Supplies / Other |
| Expense Date | Date | Date of expense |
| Amount | Number | |
| Status | Select | **Draft / Pending Review / Under Review / Approved / Rejected / Reimbursed** |
| Notes | Rich Text | Attachment notes, rejection reasons, etc. |
| Cost Classification | Select | OPEX (Operating Expense) / COGS (Cost of Goods Sold) / None |
| Misc Expense | Checkbox | |

> Rejection reasons go in the Notes field. The agent surfaces this when queried.

---

## Setup Order

Create tables in this order (relations depend on Employee Directory):

1. **Employee Directory** — no dependencies, create first
2. **Attendance Records** → relate to Employee Directory
3. **Leave Requests** → relate to Employee Directory
4. **Annual Leave** → relate to Employee Directory
5. **Payroll Records** → relate to Employee Directory
6. **Expense Claims** → relate to Employee Directory

After setup:
- Share all 6 tables with your Notion integration
- Copy all database IDs to `~/.hermes/.env` (see `use-cases/internal-ops/hr-management/config-template/`)
- Confirm dual_property relations on Attendance Records and Leave Requests

---

## Finance Module — Financial Intelligence

5 standalone tables. No relations between them (each table is self-contained, linked by Account_ID text field to COA).

```
┌──────────────────────────────────────────────────────────┐
│  CLASSIFICATION — Account taxonomy                        │
│  Chart of Accounts (COA) — all Account_IDs live here     │
└──────┬───────────────┬──────────────┬────────────────────┘
       │               │              │
┌──────▼──────┐ ┌──────▼──────┐ ┌────▼──────────────────┐
│  General    │ │  Accounts   │ │  Accounts Payable      │
│  Ledger     │ │  Receivable │ │  (Bills & Expenses)    │
│  (GL)       │ │  (AR)       │ │                        │
│  Raw        │ │  Sales      │ │  Bills & Expense       │
│  Ledger     │ │  Invoices   │ │  Records               │
└─────────────┘ └─────────────┘ └────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│  Bank Feeds (Cash Transactions) — Actual Bank Transactions│
│  Each row = one transaction, Amount +/- per account      │
└──────────────────────────────────────────────────────────┘
```

**Core Design Principles:**
- COA is the account classification backbone for all tables; Account_ID follows a unified naming convention (4xxx=Revenue, 5xxx=COGS, 6xxx=Expense, 1xxx=Asset, 2xxx=Liability)
- Income statement is calculated from GL: Revenue Credit − Expense Debit
- Cash balance is calculated from Bank Feeds: SUM(Amount) per Bank_Account
- AR/AP tracks receivable and payable statuses — not used for bookkeeping purposes

### Chart of Accounts (COA)

| Field | Type | Notes |
|-------|------|-------|
| Account_Name | Title | e.g. Consulting Revenue, Office Rent |
| Account_ID | Rich Text | e.g. 4100, 6200 |
| Account_Type | Select | Revenue / Expense / Asset / Liability / Equity |

Standard Account_ID ranges: Revenue 4xxx, COGS 5xxx, Operating Expense 6xxx, Asset 1xxx, Liability 2xxx

---

### Accounts Receivable / Sales Invoices (AR)

| Field | Type | Notes |
|-------|------|-------|
| Invoice_ID | Title | Format: INV-YYYY-NNN |
| Customer_Name | Rich Text | |
| Category | Select | Product Sales Revenue / Consulting Revenue / Other |
| Issue_Date | Date | |
| Due_Date | Date | |
| Total_Amount | Number | |
| Paid_Amount | Number | For partial payments |
| Status | Select | **Sent / Paid / Overdue / Cancelled** |

---

### Accounts Payable / Bills & Expenses (AP)

| Field | Type | Notes |
|-------|------|-------|
| Bill_ID | Title | Format: BILL-YYYY-NNN or EXP-YYYY-NNN |
| Vendor_Employee | Rich Text | Supplier or employee name |
| Category | Rich Text | Expense category name |
| Account_ID | Rich Text | COA code |
| Due_Date | Date | |
| Amount | Number | |
| Status | Select | **Unpaid / Awaiting Approval / Paid / Cancelled** |

---

### Bank Feeds / Cash Transactions

| Field | Type | Notes |
|-------|------|-------|
| Transaction_ID | Title | Format: TXN-YYYY-NNN or OPEN-YYYY-NNN |
| Bank_Account | Select | Per bank account (e.g. HSBC-HKD Current) |
| Date | Date | |
| Amount | Number | Positive = inflow, Negative = outflow |
| Payee | Rich Text | Transaction counterparty |
| Category | Select | COA account name |
| Reconciled | Checkbox | |

Opening balance row uses Category = Opening Balance and Amount = opening balance.

---

### General Ledger / Journal Entries (GL)

| Field | Type | Notes |
|-------|------|-------|
| Journal_ID | Title | Format: J-YYYY-NNNN |
| Date | Date | |
| Account_ID | Rich Text | COA code |
| Description | Rich Text | Transaction description |
| Debit_Amount | Number | |
| Credit_Amount | Number | |

---

### Finance Setup Order

No inter-table relations — create in any order:

1. **Chart of Accounts** — define all account codes first
2. **Accounts Receivable** — customer invoices
3. **Accounts Payable** — vendor bills and expenses
4. **Bank Feeds** — import bank statements (use opening balance rows)
5. **General Ledger** — journal entries referencing COA Account_IDs

After setup:
- Share all 5 tables with your Notion integration
- Copy all database IDs to `~/.hermes/.env` (see `use-cases/internal-ops/financial-intelligence/config-template/`)

---

## Discussion Module — Discussion & Todo Tracker

2 tables for logging internal discussions and tracking extracted action items.

### Internal Discussions

Logs every internal discussion, meeting, or ad-hoc conversation.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Short label for the discussion |
| Date | Date | When the discussion took place |
| Type | Select | e.g. Weekly Meeting / Phone / Online / Ad-hoc |
| Status | Select | Pending / In Progress / Completed |
| Output Type | Select | Strategy Update / Action Items Extracted / Log Only |
| Output Note | Rich Text | Summary of outcomes or key decisions |
| Participants | Rich Text | Who was involved |
| Follow-up Date | Date | When to revisit if needed |
| Related Tasks | Relation → Task Tracker | Tasks extracted from this discussion (dual property) |

**Used by:** Discussion & Todo Tracker

---

### Task Tracker

Tracks all actionable items and unresolved issues extracted from discussions.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | What needs to be done or discussed |
| Type | Select | Task / Follow-up Discussion |
| Status | Select | Pending / In Progress / Completed / Cancelled |
| Priority | Select | High / Medium / Low |
| Owner | Rich Text | Who is responsible |
| Due Date | Date | Deadline |
| Source | Select | Discussion / Business Core / Self-Initiated / Other |
| Note | Rich Text | Extra context |
| Related Discussion | Relation → Internal Discussions | Source discussion (auto back-reference) |

**Used by:** Discussion & Todo Tracker

---

## Discussion Module Setup Order

1. Create **Internal Discussions** database in Notion
2. Create **Task Tracker** database in Notion
3. Add a **Relation** from Internal Discussions → Task Tracker (dual property — back-reference auto-named, rename to "Related Discussion")
4. Share both databases with your Hermes integration
5. Add both database IDs to `~/.hermes/.env` (see `use-cases/internal-ops/discussion-todo-tracker/config-template/`)
