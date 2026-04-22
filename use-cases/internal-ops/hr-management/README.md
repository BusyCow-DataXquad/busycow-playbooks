# HR Management

Small and mid-sized businesses lose hours every week chasing HR information across spreadsheets, email threads, and chat messages. When a manager needs to approve a leave request, check someone's remaining annual leave, or find out who is on leave today, the answer is never in one place. The result: slow decisions, payroll errors, and frustrated employees.

This use case turns your Notion workspace into a conversational HR system. Managers and HR staff can query employee records, check attendance and leave balances, approve requests, and track payroll — all by sending a plain-language message. No dashboards to open, no spreadsheets to dig through.

---

## Features

| Feature | Description |
|---------|-------------|
| Employee Directory Management | Query and update employee records — name, department, title, start date, tenure, and employment status — through conversation |
| Attendance and Leave Queries | Check who is on leave today, review overtime hours for a specific employee, or look up remaining annual leave balance |
| Approval Processing | Approve or reject leave applications and expense claims directly in chat — the agent finds the pending item, confirms, then updates Notion |
| Payroll and Expense Tracking | Query monthly payroll disbursement status, view salary breakdowns, and track expense claim progress |
| 1on1 & Employee Wellbeing Records | Log every meaningful employee conversation, track wellbeing status (Stable / Needs Attention / At Risk), and receive quarterly reminders to check in with employees who haven't been contacted recently. |

---

## Required Data Tables

| Table | Notes |
|-------|-------|
| Employee Directory | Master record — all other tables relate to this one |
| Attendance Records | Daily clock-in/out and overtime |
| Leave Requests | Leave applications with embedded approval status |
| Annual Leave | Annual quota and balance per employee per year |
| Payroll Records | Monthly payroll breakdown and disbursement status |
| Expense Claims | Employee expense claims with reimbursement workflow |
| 1on1 Records | one row per conversation; links to Employee Directory via Relation |

---

## What's in This Package

```
hr-management/
├── README.md                  ← This file (human guide)
├── SETUP.md                   ← Agent runs this once to configure databases
├── SKILL.md                   ← Agent loads this for daily HR operations
└── config-template/
    └── env-template.txt       ← Environment variable template
```

---

## Install Steps

### Step 1 — Run Setup

Ask Hermes to run the HR Management setup:

> "Run the HR Management setup from SETUP.md"

The agent will search your Notion workspace for existing HR databases, help create any that are missing, adapt schemas to match the recommended fields, and save all database IDs to `~/.hermes/.env`.

### Step 2 — Load the Skill

Copy SKILL.md into the Hermes skills directory:

```bash
cp SKILL.md ~/.hermes/skills/internal-ops/hr-management/SKILL.md
```

### Step 3 — Verify Configuration

```bash
grep "HR_" ~/.hermes/.env
```

All 6 database IDs should be present. If any are missing, re-run the setup or add them manually using the template in `config-template/env-template.txt`.

### Step 4 — Test

Send a message to Hermes:

> "Who is currently on leave of absence?"

If the agent queries Notion and returns a result, setup is complete.
