# Financial Intelligence — Use Case

AI-powered financial analysis system that answers real-time financial questions in plain language, delivers automated weekly and monthly health analyses, and generates monthly and quarterly financial reports — grounded in 5 Notion financial databases.

**Category:** Internal Ops
**Data Foundation Required:** [Internal Ops](../../data-foundation/internal-ops/README.md) — Finance tables

---

## What This Use Case Does

### Feature 1 — 即時財務問答
Ask any financial question in plain language. The agent queries the 5 financial databases and returns precise numbers with context — cash balances by account, overdue receivables, upcoming payables, expense breakdown by category, and more.

### Feature 2 — 定期健康分析
Automated push to Telegram — no need to ask. Every Monday morning: a weekly cash snapshot covering balances, new invoices, and upcoming payments. Every 1st of the month: a full health report with P&L summary, cash flow trend, and anomaly alerts (overdue AR, unpaid AP, expense overruns).

### Feature 3 — 財務報表生成
Generate structured financial reports on demand or on schedule: monthly/quarterly P&L (損益表), cash flow summary (現金流摘要), and AR aging analysis (應收帳款老化分析). All output is formatted for direct use in management meetings.

---

## Required Data Tables

5 finance tables — all need to be created and shared with the Notion integration:

| Table | Abbreviation | Purpose |
|-------|-------------|---------|
| Chart of Accounts | COA | Account classification backbone |
| General Ledger | GL | Bottom-level transaction journal |
| Accounts Receivable | AR | Customer invoices and collection status |
| Accounts Payable | AP | Vendor bills and expense claims |
| Bank Feeds | Bank | Real cash in/out per bank account |

See `data-foundation/internal-ops/README.md` for full schema.

---

## Setup Steps

### Step 1 — Create Notion Databases

Create 5 databases following `data-foundation/internal-ops/README.md`.
Share all with your Notion integration.

### Step 2 — Install the Skill

```bash
cp -r skills/internal-ops/financial-intelligence ~/.hermes/skills/internal-ops/
```

### Step 3 — Configure

```bash
cat config-template/env-template.txt >> ~/.hermes/.env
# Fill in all 5 DB IDs, NOTION_TOKEN, and Telegram details
```

### Step 4 — Set Up Weekly Snapshot (Optional)

```bash
# Every Monday 9am Taiwan time
hermes cron create \
  --schedule "0 1 * * 1" \
  --name "finance-weekly-snapshot" \
  --prompt "Run the Financial Intelligence weekly snapshot: query Bank Feeds, AR, AP and deliver a cash status summary to Telegram."
```

### Step 5 — Set Up Monthly Health Report (Optional)

```bash
# Every 1st of month 9am Taiwan time
hermes cron create \
  --schedule "0 1 1 * *" \
  --name "finance-monthly-report" \
  --prompt "Run the Financial Intelligence monthly health report: query GL, COA, AR, AP, Bank Feeds for last month, compute P&L and cashflow trend, deliver to Telegram."
```

---

## How to Use

**Real-time questions:**
- 「我們現在帳上有多少錢？各個戶頭分別多少？」
- 「扣掉下週要付的款，實際可動用的還有多少？」
- 「有哪些發票已經逾期還沒收款？」
- 「這個月費用最大的幾個科目是什麼？」

**Generate reports:**
- 「幫我出 3 月份損益表」
- 「應收帳款老化分析，列出超過 30 天沒收的」
- 「這季的現金流量摘要」
- 「Q1 跟 Q2 的毛利對比」

---

## Folder Structure

```
financial-intelligence/
├── README.md
├── skills/
│   └── internal-ops/
│       └── financial-intelligence/
│           └── SKILL.md          ← Install this
└── config-template/
    └── env-template.txt
```
