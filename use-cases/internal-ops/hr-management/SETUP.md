# HR Management — Setup Guide

Run this setup once when deploying the HR Management use case for a new client. Follow each phase in order. Do not skip ahead.

---

## Phase 1 — Discovery

Search the client's Notion workspace for existing databases that may already serve HR functions.

**Search keywords to use:**
- Employee, Staff, Team, People, Personnel
- Attendance, Clock-in, Timesheet, Overtime
- Leave, Time Off, Annual Leave, Vacation, Sick Leave
- Payroll, Salary, Compensation, Wages
- Expense, Reimbursement, Claims
- 1on1, one-on-one, wellbeing, employee conversation, retention

**Property patterns that indicate a match:**
- A database with a name field plus Department, Job Title, and a Status field with values like Active / Resigned — likely the Employee Directory
- A database with Date fields and an Employee relation — likely Attendance Records or Leave Requests
- A database with Amount/Number fields and a Status like Paid / Pending — likely Payroll or Expenses
- A database with a Year field and remaining days — likely Annual Leave quotas

**Action:** Search Notion using the keywords above. List every database you find with its name, a sample of its properties, and which HR function it likely serves. Report your findings to the user before proceeding to Phase 2.

---

## Phase 2 — Onboard

Review the discovery findings with the user. For each of the 7 required databases, determine whether an existing database can serve that role or a new one must be created.

**Dependency rule — Employee Directory first:**
The Employee Directory must exist before any other database is created or connected, because all other tables (Attendance, Leave, Payroll, Expense, 1on1 Records) use a relation field that points back to it. Create or confirm the Employee Directory first, then proceed to the remaining six databases in any order.

**The 7 required databases:**
1. Employee Directory
2. Attendance Records
3. Leave Requests
4. Annual Leave
5. Payroll Records
6. Expense Claims
7. 1on1 Records

For each missing database, show the user the recommended schema from the section below and ask: "Would you like me to create this database via the Notion API, or would you prefer to create it manually?"

- If creating via API: use POST /databases to create the database, then tell the user: "Please open Notion, find the newly created database, and share it with the Hermes integration so I can access it."
- If creating manually: walk the user through the field list and confirm when done.

---

## Phase 3 — Adapt

For each database that was found or just created, fetch its current schema using GET /databases/{id}.

Compare the actual property names and types against the recommended schema below. For any recommended field that is missing:

1. Show the user the gap: "The Leave Requests database is missing a 'Days' number field. Would you like me to add it?"
2. After confirmation, use PATCH /databases/{id} to add the missing field.

**Rules:**
- Never delete or rename fields that already exist in the client's database
- Never change a field type without explicit user confirmation
- If the client has a field that serves the same purpose under a different name, note it and move on — do not duplicate it

---

## Phase 4 — Configure

Once all databases are confirmed and accessible, retrieve each database ID from the Notion URL (the 32-character hex string after the last `/`).

Use the memory tool to save all IDs under a clearly labeled memory entry:

```
HR Management config:
  employees_db: <Employee Directory database ID>
  attendance_db: <Attendance Records database ID>
  leave_db: <Leave Requests database ID>
  annual_leave_db: <Annual Leave database ID>
  payroll_db: <Payroll Records database ID>
  expense_db: <Expense Claims database ID>
  one_on_one_db: <1on1 Records database ID>
```

After saving to memory, confirm with a summary:

> "HR Management setup is complete. The following databases are configured in agent memory:
> - Employee Directory: [ID]
> - Attendance Records: [ID]
> - Leave Requests: [ID]
> - Annual Leave: [ID]
> - Payroll Records: [ID]
> - Expense Claims: [ID]
> - 1on1 Records: [ID]
>
> You can now load SKILL.md to activate daily HR operations."

---

## Recommended Schema

These are suggestions based on common HR setups. Adapt to what the client already has — field names and option values may differ. Do not force the client to match this exactly.

### Employee Directory

| Field | Type | Notes |
|-------|------|-------|
| Employee Name | Title | Primary identifier |
| English Name | Rich Text | Optional transliteration |
| Employee ID | Rich Text | e.g. EMP-001 |
| Department | Select | e.g. Sales, Engineering, HR, Operations |
| Job Title | Rich Text | |
| Start Date | Date | |
| End Date | Date | Leave blank for active employees |
| Status | Select | Active / Probation / Leave of Absence / Resigned |
| Notes | Rich Text | |

### Attendance Records

| Field | Type | Notes |
|-------|------|-------|
| Record Name | Title | e.g. A-2026-0421-Lin |
| Employee | Relation → Employee Directory | |
| Clock-In Time | Date (with time) | |
| Clock-Out Time | Date (with time) | |
| Total Overtime Hours | Number | In hours |
| Overtime Reason | Rich Text | |

### Leave Requests

| Field | Type | Notes |
|-------|------|-------|
| Leave Request | Title | e.g. L-2026-0005 [Name] [Type] |
| Employee | Relation → Employee Directory | |
| Leave Type | Select | Annual / Sick / Personal / Public Duty / Compensatory / Other |
| Start Date | Date | |
| End Date | Date | |
| Days | Number | Supports 0.5 increments |
| Reason | Rich Text | |
| Status | Select | Pending / Approved / Rejected / Cancelled |

### Annual Leave

| Field | Type | Notes |
|-------|------|-------|
| Record Name | Title | e.g. 2026-Lin |
| Employee | Relation → Employee Directory | |
| Year | Rich Text | e.g. 2026 |
| Total Annual Leave Days | Number | |
| Used Days | Number | |
| Remaining Days | Number | |
| Notes | Rich Text | e.g. "On leave of absence" |

### Payroll Records

| Field | Type | Notes |
|-------|------|-------|
| Payroll Record | Title | e.g. P-2026-03-Lin |
| Employee | Relation → Employee Directory | |
| Month | Rich Text | e.g. 2026-03 |
| Base Salary | Number | |
| Overtime Pay | Number | |
| Allowances | Number | |
| Deductions | Number | |
| Net Pay | Number | |
| Disbursement Status | Select | Paid / Pending / Awaiting Confirmation |

### Expense Claims

| Field | Type | Notes |
|-------|------|-------|
| Expense Item | Title | Descriptive name |
| Employee | Relation → Employee Directory | |
| Category | Select | Travel / Transportation / Meals / Accommodation / Software / Office Supplies / Other |
| Expense Date | Date | |
| Amount | Number | |
| Status | Select | Draft / Pending Review / Under Review / Approved / Rejected / Reimbursed |
| Notes | Rich Text | Rejection reasons, attachment notes |
| Cost Classification | Select | OPEX / COGS / None |

### 1on1 Records

> Note: Create AFTER Employee Directory (it depends on the relation).

| Field | Type | Notes |
|-------|------|-------|
| Conversation Title | Title | |
| Employee | Relation → Employee Directory | Dual property — Employee Directory should show '1on1 Records' backlink |
| Conversation Date | Date | |
| Type | Select | Regular 1on1 / Wellbeing Check-in / Performance Discussion / Salary Discussion / Career Planning / Issue Resolution / Other |
| Facilitator | Rich Text | Who led the conversation |
| Employee Wellbeing Status | Select | Stable / Needs Attention / At Risk / Not Assessed |
| Next Suggested Date | Date | |
| Summary | Rich Text | Key points from the conversation |
| Next Actions | Rich Text | Follow-up commitments |
| Days Since Last 1on1 | Number | For cron job calculations |
