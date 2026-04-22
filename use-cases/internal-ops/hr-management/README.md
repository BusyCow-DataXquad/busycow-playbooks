# HR Management — Use Case

AI-powered HR assistant that manages employee records, attendance and leave, approval workflows, and payroll tracking — all through natural conversation with Telegram or WhatsApp.

**Category:** Internal Ops
**Data Foundation Required:** [Internal Ops](../../data-foundation/internal-ops/README.md) — HR tables

---

## What This Use Case Does

### Feature 1 — Employee Directory Management
Query and update employee records through conversation: name, department, title, start date, tenure, employment status (active / probation / leave of absence / resigned). Add new employees with guided field prompts.

### Feature 2 — Attendance and Leave Queries
Check who is on leave today, query attendance and overtime hours for a specific employee, check remaining annual leave balance. Covers all leave types: annual, sick, personal, compensatory, public duty.

### Feature 3 — Approval Processing
HR or managers approve or reject leave applications and expense claims directly through chat — no need to open Notion manually. The agent finds the pending items, confirms the action, then updates the status.

### Feature 4 — Payroll and Expense Tracking
Query monthly payroll status (paid / pending), view salary breakdown (base + overtime + allowance + deductions), track expense claim progress and find items that are pending review or have been rejected.

---

## Required Data Tables

6 tables, all under the Internal Ops workspace:

| Table | Purpose | Has Approval Status |
|-------|---------|-------------------|
| Employee Directory | Employee master records | No |
| Attendance Records | Daily attendance and overtime | No |
| Leave Requests | Leave applications and approvals | ✅ Yes |
| Annual Leave | Annual leave quota per employee per year | No |
| Payroll Records | Monthly payroll breakdown and status | ✅ Yes |
| Expense Claims | Expense claims and reimbursement flow | ✅ Yes |

> Approval status is embedded directly in Leave Requests and Expense Claims. No separate approval table needed.

See `SKILL.md` for full schema of all 6 tables.

---

## Setup Steps

### Step 1 — Create Notion Databases

Create 6 databases in your Internal Ops Notion workspace. Share all with your Notion integration.
See `SKILL.md` for field-by-field schema.

### Step 2 — Install the Skill

```bash
cp -r skills/internal-ops/hr-management ~/.hermes/skills/internal-ops/
```

### Step 3 — Configure

```bash
cat config-template/env-template.txt >> ~/.hermes/.env
# Fill in all 6 DB IDs and NOTION_TOKEN
```

---

## How to Use

**Employee records:**
- "When did Lin Tsai-En join? How long have they been with the company?"
- "Who is currently on leave of absence?"
- "Add a new employee: Huang Yi-Ting, Sales department, start date 5/1"

**Attendance and leave:**
- "Who is on leave on April 21?"
- "What is Lin Tsai-En's total overtime hours for March?"
- "How many annual leave days does Chang Ya-Ju have left this year?"

**Approvals:**
- "What leave requests are currently pending approval?"
- "Approve Chang Ya-Ju's personal leave request (L-2026-0005)"
- "Which expense claims this month are still pending or under review?"

**Payroll and expenses:**
- "Which March payroll entries are still in pending status?"
- "The business trip accommodation (Taichung) expense claim was rejected — what was the reason?"
- "What was Chen Yu-Ting's net pay for March?"

---

## Folder Structure

```
hr-management/
├── README.md
├── skills/
│   └── internal-ops/
│       └── hr-management/
│           └── SKILL.md          ← Install this
└── config-template/
    └── env-template.txt
```
