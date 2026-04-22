# HR 人事管理 — Use Case

AI-powered HR assistant that manages employee records, attendance and leave, approval workflows, and payroll tracking — all through natural conversation with Telegram or WhatsApp.

**Category:** Internal Ops
**Data Foundation Required:** [Internal Ops](../../data-foundation/internal-ops/README.md) — HR tables

---

## What This Use Case Does

### Feature 1 — 人員名冊管理
Query and update employee records through conversation: name, department, title, start date, tenure, employment status (active / probation / leave of absence / resigned). Add new employees with guided field prompts.

### Feature 2 — 出勤與請假查詢
Check who is on leave today, query attendance and overtime hours for a specific employee, check remaining annual leave balance. Covers all leave types: annual, sick, personal, compensatory, public duty.

### Feature 3 — 審批處理
HR or managers approve or reject leave applications and expense claims directly through chat — no need to open Notion manually. The agent finds the pending items, confirms the action, then updates the status.

### Feature 4 — 薪酬與報帳追蹤
Query monthly payroll status (paid / pending), view salary breakdown (base + overtime + allowance + deductions), track expense claim progress and find items that are pending review or have been rejected.

---

## Required Data Tables

6 tables, all under the Internal Ops workspace:

| Table | Purpose | Has Approval Status |
|-------|---------|-------------------|
| 員工名冊 | Employee master records | No |
| 出勤紀錄 | Daily attendance and overtime | No |
| 請假紀錄 | Leave applications and approvals | ✅ Yes |
| 年假管理 | Annual leave quota per employee per year | No |
| 薪酬發放紀錄 | Monthly payroll breakdown and status | ✅ Yes |
| 報帳請款紀錄 | Expense claims and reimbursement flow | ✅ Yes |

> Approval status is embedded directly in 請假紀錄 and 報帳請款紀錄. No separate approval table needed.

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
- 「林采恩什麼時候到職？現在年資多久了？」
- 「現在有誰是留職停薪狀態？」
- 「幫我新增一位員工：黃怡婷，業務部，到職日 5/1」

**Attendance and leave:**
- 「4 月 21 日有誰請假？」
- 「林采恩三月的加班時數總計多少？」
- 「張雅筑今年還剩幾天年假？」

**Approvals:**
- 「現在有哪些請假待核准？」
- 「核准張雅筑的事假申請（L-2026-0005）」
- 「這個月哪些報帳還在待審核或審核中？」

**Payroll and expenses:**
- 「3 月份薪資有哪些還是待發放狀態？」
- 「差旅住宿（台中）那筆報帳被退回了，原因是什麼？」
- 「陳昱廷 3 月實發多少？」

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
