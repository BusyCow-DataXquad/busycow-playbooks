---
name: hr-management
description: >
  AI-powered HR assistant for Hermes Agent. Manages employee records,
  attendance and leave tracking, approval workflows, and payroll/expense
  monitoring — all through natural conversation. Grounded in 7 Notion databases.
version: 1.1.0
author: BusyCow
tags: [HR, Human Resources, Attendance, Payroll, Notion, Internal Ops]
---

# HR Management — Daily Operations

> Setup is assumed complete. All 7 databases are configured in agent memory from the SETUP phase. Do not run setup steps from here.

> Field names may differ from recommended schema. Always check actual field names when querying or writing.

---

## Overview

Five features that replace manual HR tracking with conversational queries and actions:

1. **Employee Directory Management** — Query and update employee records through conversation
2. **Attendance and Leave Queries** — Check leave, attendance, and annual leave balances
3. **Approval Processing** — Approve or reject leave and expense requests directly in chat
4. **Payroll and Expense Tracking** — Review payroll disbursement status and expense claim progress
5. **1on1 & Employee Wellbeing Records** — Log employee conversations, track wellbeing, and receive quarterly check-in reminders

---

## Feature 1 — Employee Directory Management

### Triggers
- "When did [name] join? How long have they been with the company?"
- "Who is currently on leave of absence?"
- "Update [name]'s job title to [new title], effective today"
- "Add a new employee: [name], [department], start date [date]"

### Workflow

**Query:**
1. POST /databases/{employee directory database (from memory)}/query — filter by name, department, or status field
2. Calculate tenure from the start date field to today
3. Return employee card

**Update:**
1. Locate the page by querying name
2. Show proposed change to user and ask for confirmation
3. After confirmation: PATCH /pages/{page_id} with updated fields

**Add new employee:**
1. Check which required fields are missing from the user's message (department, job title, start date at minimum)
2. Ask for any missing required fields before proceeding
3. After all fields collected, confirm the full record with the user
4. POST /pages to create the record

### Response Format

```
👤 [Employee Name] ([Employee ID])
Department: [dept] | Job Title: [title]
Start Date: [date] | Tenure: ~[X] years [Y] months
Status: [status]
```

---

## Feature 2 — Attendance and Leave Queries

### Triggers
- "Who is on leave on [date]?"
- "What is [name]'s total overtime hours for [month]?"
- "How many annual leave days does [name] have left this year?"
- "Which compensatory leave has not been used this month?"

### Workflow

**Leave queries:**
1. POST /databases/{leave requests database (from memory)}/query — filter by date range, leave type, or employee relation field
2. For annual leave balance: POST /databases/{annual leave database (from memory)}/query — filter by employee and year field

**Attendance queries:**
1. POST /databases/{attendance records database (from memory)}/query — filter by employee relation + date range
2. Sum the overtime hours field across matching records

### Response Format

```
📋 Leave on [date]:
• [Name] — [Leave Type] [X] day(s) (Status: [status])

Remaining Annual Leave ([Name]): [X] days ([year])
```

---

## Feature 3 — Approval Processing

### Triggers
- "What leave requests are currently pending approval?"
- "Approve [name]'s [leave type] request ([request ID])"
- "Reject the [item] expense claim — reason: [reason]"
- "Which expense claims this month are still pending or under review?"

### Workflow

**Query pending items:**
1. POST /databases/{leave requests database (from memory)}/query — filter: status field = Pending
2. POST /databases/{expense claims database (from memory)}/query — filter: status field in [Pending Review, Under Review]

**Execute approval:**
1. Locate the page_id of the target record
2. Show confirmation: "Are you sure you want to [approve/reject] [applicant]'s [item] ([date or amount])?"
3. Only after explicit confirmation: PATCH /pages/{page_id} to update the status field

**Status transitions:**
- Leave: Pending → Approved or Rejected
- Expense: Pending Review / Under Review → Approved / Rejected / Reimbursed

### Response Format

```
✅ Approved: [Name]'s [leave type] request ([request ID])
Dates: [start] to [end] | [X] day(s)
Status updated to: Approved
```

---

## Feature 4 — Payroll and Expense Tracking

### Triggers
- "Which [month] payroll entries are still in pending status?"
- "The [item] expense claim was rejected — what was the reason?"
- "What was [name]'s net pay for [month]?"
- "Which expense claims this month are still pending or under review?"

### Workflow

**Payroll queries:**
1. POST /databases/{payroll records database (from memory)}/query — filter by month field + disbursement status field
2. Return employee name, salary components, net pay, and status

**Expense queries:**
1. POST /databases/{expense claims database (from memory)}/query — filter by employee, status, or category field
2. For rejection reasons: check the notes field of the relevant record

### Response Format

```
💰 [Month] Payroll Status:

Pending:
• [Name] — Net Pay [amount] (Base [x] + Overtime [x] + Allowances [x] − Deductions [x])

Paid: [N] entries
```

---

## Feature 5 — 1on1 & Employee Wellbeing Records

### What it does

This feature turns ad-hoc employee conversations into structured records — so managers never lose track of what was discussed, agreed, or flagged. It covers the '育' (develop) and '留' (retain) dimensions of HR — the parts most often neglected in SMEs without a dedicated HR team. By logging wellbeing status over time and triggering proactive reminders, it helps managers stay ahead of retention risks before they become resignations.

### Conversational Triggers

- "Log a 1on1 with [name] — we talked about career direction, she seems stable"
- "What's the last time I talked to [name]?"
- "Who hasn't had a 1on1 in the past 3 months?"
- "[name] seems unhappy lately — log a wellbeing note"
- "Set up quarterly 1on1 reminders"

### Workflow — Logging a Conversation

1. Ask for: employee name, date, type, summary of what was discussed
2. Ask: "How would you rate their current wellbeing? (Stable / Needs Attention / At Risk)"
3. Ask: "Any follow-up actions?"
4. Ask: "When should the next conversation happen? (default: 90 days from today)"
5. Create the record in 1on1 Records DB
6. Confirm: show a summary card

### Workflow — Quarterly Reminder Cron

Schedule: every 3 months (or configurable). Query 1on1 Records for each active employee — find employees where:
- Next Suggested Date <= today, OR
- No 1on1 record exists in the past 90 days

For At Risk employees: trigger if > 30 days since last conversation
For Needs Attention: trigger if > 45 days
For Stable: trigger if > 90 days

Deliver a reminder message:

```
🔔 1on1 Reminder — [DATE]

Employees due for a check-in:

⚠️ [Name] — Status: At Risk | Last 1on1: [DATE] ([N] days ago)
Last note: [summary excerpt]
Suggested action: Schedule this week

🟡 [Name] — Status: Needs Attention | Last 1on1: [DATE] ([N] days ago)
Suggested action: Schedule within 2 weeks

✅ [Name] — Status: Stable | Last 1on1: [DATE] ([N] days ago)
Suggested action: Schedule within a month
```

### Behavioral Rules for this Feature

- Never share one employee's conversation details when queried about another employee
- Always confirm the wellbeing status with the user — never infer it from conversation tone alone
- For At Risk employees, suggest scheduling the next conversation immediately
- Cron reminders run automatically — no user confirmation needed to send the reminder
- If no 1on1 record exists for an employee at all, flag them as a priority in the cron reminder

---

## Behavioral Rules

1. **Confirm before changing** — All updates (status changes, field edits) must show a confirmation message first. Only execute after the user confirms.
2. **Ask for missing fields** — When adding a new employee or record, ask for any required fields that are absent before creating the record.
3. **Never skip approval confirmation** — Approval actions must display the applicant name, item, and date/amount before executing. No exceptions.
4. **Never guess on employee identity** — If a name is ambiguous (e.g. "Ms. Chang" when multiple Changs exist), query first and confirm which employee before proceeding.
5. **Handle salary data carefully** — Salary details are only shown when explicitly requested. Never surface salary breakdowns in overview or summary queries.

---

## Configuration

Databases are configured in agent memory from the SETUP phase. The following databases are used:

```
# HR Databases (read from agent memory)
employees_db        # Employee Directory
attendance_db       # Attendance Records
leave_db            # Leave Requests
annual_leave_db     # Annual Leave
payroll_db          # Payroll Records
expense_db          # Expense Claims
one_on_one_db       # 1on1 & Wellbeing Records
```
