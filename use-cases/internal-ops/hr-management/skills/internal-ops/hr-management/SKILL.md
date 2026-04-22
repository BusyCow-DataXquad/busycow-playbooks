---
name: hr-management
description: >
  AI-powered HR assistant for Hermes Agent. Manages employee records,
  attendance and leave tracking, approval workflows, and payroll/expense
  monitoring — all through natural conversation. Grounded in 6 Notion databases.
version: 1.0.0
author: BusyCow
tags: [HR, Human Resources, Attendance, Payroll, Notion, Internal Ops]
---

# HR Management — AI HR Administration System

## Overview

Transforms HR administration from "track manually, fill manually, notify manually" into actions you can query or approve in a single message.

Four core features:
1. **Employee Directory Management** — conversational queries and updates to employee records
2. **Attendance and Leave Queries** — check attendance records, leave status, and remaining annual leave
3. **Approval Processing** — approve or reject leave / expense requests
4. **Payroll and Expense Tracking** — check payroll disbursement status and expense claim progress

---

## Notion Databases Required

6 tables, all under the Internal Ops workspace.

| Table | Purpose | Approval Status Field |
|--------|------|------------|
| Employee Directory | Employee basic info and employment history | None (factual only) |
| Attendance Records | Daily clock-in/out times and overtime hours | None |
| Leave Requests | All leave type applications and approvals | Pending / Approved / Rejected / Cancelled |
| Annual Leave | Annual quota and remaining days per employee | None |
| Payroll Records | Monthly payroll breakdown and disbursement status | Paid / Pending / Awaiting Confirmation |
| Expense Claims | Employee expense submissions and approval workflow | Draft / Pending Review / Under Review / Approved / Rejected / Reimbursed |

> Approval status is embedded directly in Leave Requests and Expense Claims — no separate approval table needed.

### Employee Directory Schema

| Field | Type | Notes |
|------|------|------|
| Employee Name | Title | |
| English Name | Rich Text | |
| Employee ID | Rich Text | Format: EMP-001 |
| Department | Select | Sales / Engineering / Product / Design / HR / Operations |
| Job Title | Rich Text | |
| Start Date | Date | |
| End Date | Date | Leave blank for active employees |
| Status | Select | Active / Probation / Leave of Absence / Resigned |
| Notes | Rich Text | |

### Attendance Records Schema

| Field | Type | Notes |
|------|------|------|
| Attendance Record | Title | Format: A-YYYY-MMDD-[surname] |
| Employee | Relation → Employee Directory | |
| Clock-In Time | Date (datetime) | |
| Clock-Out Time | Date (datetime) | |
| Total Overtime Hours | Number | Unit: hours |
| Overtime Reason | Rich Text | |

### Leave Requests Schema

| Field | Type | Notes |
|------|------|------|
| Leave Request | Title | Format: L-YYYY-NNNN [name] [leave-type] |
| Employee | Relation → Employee Directory | |
| Leave Type | Select | Annual / Paid Leave / Sick / Personal / Public Duty / Compensatory / Other |
| Start Date | Date | |
| End Date | Date | |
| Days | Number | Supports 0.5 days |
| Reason | Rich Text | |
| Status | Select | Pending / Approved / Rejected / Cancelled |

### Annual Leave Schema

| Field | Type | Notes |
|------|------|------|
| Annual Leave Record | Title | Format: YYYY-[name] |
| Employee | Relation → Employee Directory | |
| Year | Rich Text | |
| Total Annual Leave Days | Number | |
| Used Days | Number | |
| Remaining Days | Number | |
| Notes | Rich Text | e.g. "On leave of absence" |

### Payroll Records Schema

| Field | Type | Notes |
|------|------|------|
| Payroll Record | Title | Format: P-YYYY-MM-[name] |
| Employee | Relation → Employee Directory | |
| Month | Rich Text | Format: YYYY-MM |
| Base Salary | Number | |
| Overtime Pay | Number | |
| Allowances | Number | |
| Deductions | Number | |
| Net Pay | Number | |
| Disbursement Status | Select | Paid / Pending / Awaiting Confirmation |

### Expense Claims Schema

| Field | Type | Notes |
|------|------|------|
| Expense Item | Title | Descriptive name |
| Employee | Relation → Employee Directory | |
| Category | Select | Travel / Transportation / Meals / Accommodation / Software/Subscriptions / Office Supplies / Other |
| Expense Date | Date | |
| Amount | Number | |
| Status | Select | Draft / Pending Review / Under Review / Approved / Rejected / Reimbursed |
| Notes | Rich Text | Attachment notes, rejection reasons, etc. |
| Cost Classification | Select | OPEX / COGS / None |
| Misc Expense | Checkbox | |

---

## Feature 1 — Employee Directory Management

### Conversational Triggers

- "When did Lin Tsai-En join? How long have they been with the company?"
- "Who is currently on leave of absence?"
- "Update Wang Bo-Kai's job title to Senior Operations, effective today"
- "Add a new employee: Huang Yi-Ting, Sales department, start date 5/1"

### Workflow

1. Query: POST /databases/{HR_EMPLOYEES_DB}/query — filter by name, department, or status
2. Update: PATCH /pages/{page_id} — confirm changes before applying
3. Add new: proactively ask for missing fields (department, job title, start date at minimum), then POST /pages after confirmation

Response format (query):
```
👤 Wang Bo-Kai (EMP-005)
Department: Operations | Job Title: Operations Specialist
Start Date: 2022-11-21 | Tenure: ~3 years 5 months
Status: On Leave of Absence
```

---

## Feature 2 — Attendance and Leave Queries

### Conversational Triggers

- "Who is on leave on April 21?"
- "What is Lin Tsai-En's total overtime hours for March?"
- "How many annual leave days does Chang Ya-Ju have left this year?"
- "Which compensatory leave has not been used this month?"

### Workflow

**Leave queries:**
- POST /databases/{HR_LEAVE_DB}/query — filter by Start Date / Leave Type / Employee
- Annual leave: POST /databases/{HR_ANNUAL_LEAVE_DB}/query — filter by Employee

**Attendance queries:**
- POST /databases/{HR_ATTENDANCE_DB}/query — filter by Employee + date range
- Sum Total Overtime Hours

Response format (leave query):
```
📋 Leave on April 21:
• Chang Ya-Ju — Personal Leave 1 day (Status: Pending)

Remaining Annual Leave (Chang Ya-Ju): 7 days (2026)
```

---

## Feature 3 — Approval Processing

### Conversational Triggers

- "What leave requests are currently pending approval?"
- "Approve Chang Ya-Ju's personal leave request (L-2026-0005)"
- "What is the status of the Taipei business trip HSR expense claim?"
- "Which expense claims this month are still pending or under review?"

### Workflow

**Query pending items:**
- POST /databases/{HR_LEAVE_DB}/query — filter: Status = Pending
- POST /databases/{HR_EXPENSE_DB}/query — filter: Status in [Pending Review, Under Review, Rejected]

**Execute approval:**
1. Locate the page_id of the relevant record
2. Confirm: "Are you sure you want to approve [applicant]'s [leave type/item] ([date/amount])?"
3. After confirmation: PATCH /pages/{page_id} to update the Status field

**Approval status transitions:**
- Leave: Pending → Approved / Rejected
- Expense: Pending Review / Under Review → Approved / Rejected / Reimbursed

> Important: Never skip the confirmation step — HR or the manager must confirm before any update is applied.

---

## Feature 4 — Payroll and Expense Tracking

### Conversational Triggers

- "Which March payroll entries are still in pending status?"
- "The business trip accommodation (Taichung) expense claim was rejected — what was the reason?"
- "What was Chen Yu-Ting's net pay for March? What was the overtime pay?"
- "Which expense claims this month are still pending or under review?"

### Workflow

**Payroll queries:**
- POST /databases/{HR_PAYROLL_DB}/query — filter by Month + Disbursement Status
- List employee name, base salary, overtime pay, allowances, deductions, net pay

**Expense queries:**
- POST /databases/{HR_EXPENSE_DB}/query — filter by Employee / Status / Category
- Rejection reasons are in the Notes field

Response format (payroll):
```
💰 2026-03 Payroll Status:
Pending:
• Lin Tsai-En — Net Pay NT$51,200 (Base 45,000 + Overtime 4,200 + Allowances 2,000)

Paid: 4 entries (omitted)
```

---

## Behavioral Rules

1. **Confirm before changing** — all updates (status changes, data edits) must display a confirmation message first; only execute after confirmation is received
2. **Ask for missing fields** — when adding an employee or record, proactively ask for any required fields that are missing (department, start date, etc.)
3. **No skipping approval confirmation** — approval actions must list the applicant, item, and date/amount; only update after confirmation
4. **Never guess on employee identity** — if a name is ambiguous (e.g. "Ms. Chang"), query first to confirm which employee before proceeding
5. **Handle sensitive data carefully** — salary details are only shown when explicitly requested; never expand them in overview queries

---

## Configuration

```
# Notion
NOTION_TOKEN=*** Notion integration token

# HR Databases
HR_EMPLOYEES_DB=                      # Employee Directory DB ID
HR_ATTENDANCE_DB=                     # Attendance Records DB ID
HR_LEAVE_DB=                          # Leave Requests DB ID
HR_ANNUAL_LEAVE_DB=                   # Annual Leave DB ID
HR_PAYROLL_DB=                        # Payroll Records DB ID
HR_EXPENSE_DB=                        # Expense Claims DB ID
```
