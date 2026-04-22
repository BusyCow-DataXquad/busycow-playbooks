# Financial Intelligence

Growing businesses often have money spread across multiple bank accounts, outstanding invoices, upcoming vendor payments, and a general ledger no one has time to read. Getting a clear answer to "how much cash do we actually have right now?" or "which invoices are overdue?" requires opening multiple tools, exporting spreadsheets, or waiting for the accountant to reply. By the time the answer arrives, decisions have already been made on guesswork.

This use case gives business owners and finance leads a direct line to their financial data. Ask questions in plain language, get precise answers from your Notion financial databases. Automated weekly snapshots and monthly health reports arrive without asking. Structured P&L, AR aging, and cash flow reports are generated on demand. All numbers come from the actual data — no estimates, no fabrication.

---

## Features

| Feature | Description |
|---------|-------------|
| Real-Time Financial Q&A | Ask any financial question and get precise numbers — cash balances, overdue receivables, upcoming payments, expense breakdowns |
| Scheduled Health Analysis | Automated weekly cash snapshot (every Monday) and monthly health report (1st of month) delivered to Telegram without prompting |
| Financial Report Generation | Generate P&L statements, AR aging analysis, and cash flow summaries on demand or on schedule |

---

## What's in This Package

```
financial-intelligence/
├── README.md                  ← This file (human guide)
├── SETUP.md                   ← Agent runs this once to configure databases
├── SKILL.md                   ← Agent loads this for daily financial operations
└── config-template/
    └── env-template.txt       ← Environment variable template
```

---

## Install Steps

### Step 1 — Run Setup

Ask Hermes to run the Financial Intelligence setup:

> "Run the Financial Intelligence setup from SETUP.md"

The agent will search your Notion workspace for existing financial databases, determine whether you are mirroring data from accounting software or running Notion as standalone, help create any missing databases, adapt schemas, and save all IDs to `~/.hermes/.env`.

### Step 2 — Load the Skill

Copy SKILL.md into the Hermes skills directory:

```bash
cp SKILL.md ~/.hermes/skills/internal-ops/financial-intelligence/SKILL.md
```

### Step 3 — Verify Configuration

```bash
grep "FINANCE_" ~/.hermes/.env
```

All 5 database IDs and Telegram settings should be present. If any are missing, re-run setup or add them manually using `config-template/env-template.txt`.

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

### Step 6 — Test

Send a message to Hermes:

> "How much money do we have right now?"

If the agent queries Bank Feeds and returns balances by account, setup is complete.
