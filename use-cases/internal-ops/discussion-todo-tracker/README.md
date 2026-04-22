# Discussion & Todo Tracker

Log internal discussions in plain language. Extract todos, preserve open issues, and keep your Business Core KB in sync when strategy shifts.

---

## What it does

Every company runs on internal discussions — weekly syncs, quick calls, Telegram threads. The problem is most of it disappears: decisions half-made, action items in someone's head, follow-ups that never happen.

This use case gives Hermes a structured way to handle all of it through natural conversation:

1. **Log** — paste meeting notes or describe a conversation; the agent structures it into a Discussion record
2. **Extract todos** — action items become Task records with owner and due date
3. **Preserve open issues** — unresolved topics become Pending Follow-up tasks so nothing slips
4. **Update Business Core** — strategy decisions trigger a direct update to the relevant KB document, always with your confirmation first

---

## Sub-functions

| # | Function | Description |
|---|----------|-------------|
| 1 | Plain-language input | Paste notes or describe a conversation in any format — no reformatting required |
| 2 | Key info extraction + Business Core update | Strategy shifts → agent proposes update to KB, confirms before writing |
| 3 | Todo extraction | Action items → Task Tracker with owner and deadline |
| 4 | Open issue preservation | Unresolved topics → Pending Follow-up tasks |
| 5 | Status tracking | Every Discussion has a Status — nothing is Done until fully processed |

---

## Recommended Data Schema

### Internal Discussions

| Field | Type | Options |
|-------|------|---------|
| Name | Title | Discussion title or auto-generated |
| Date | Date | |
| Type | Select | Weekly Meeting / Call / Online / Ad-hoc |
| Status | Select | Pending / In Progress / Done |
| Output Type | Select | Strategy Update / Todo Extraction / Record Only |
| Output Note | Rich Text | Summary of decisions and outputs |
| Participants | Rich Text | Everyone present |
| Follow-up Date | Date | Scheduled next touchpoint |
| Related Tasks | Relation → Task Tracker | Linked action items |

### Task Tracker

| Field | Type | Options |
|-------|------|---------|
| Name | Title | Task description |
| Type | Select | Task / Pending Follow-up |
| Status | Select | Pending / In Progress / Done / Cancelled |
| Priority | Select | High / Medium / Low |
| Owner | Rich Text | Person responsible |
| Due Date | Date | |
| Source | Select | Discussion / Business Core / Self-initiated / Other |
| Note | Rich Text | Context and instructions |
| Related Discussion | Relation → Internal Discussions | Source discussion |

The two databases are linked by a Relation on both sides. This is required for the skill to function correctly.

### Business Core KB

Standalone Notion pages — not databases. Typically covers:

- Brand positioning
- Target audience analysis
- Pricing and business model

The agent reads these pages for context and writes to them when a discussion produces a confirmed strategy-level decision.

---

## Files

| File | Purpose |
|------|---------|
| `SETUP.md` | Agent-facing setup instructions — run once per client |
| `SKILL.md` | This file — daily use reference and behavioral rules |
| `config-template/env-template.txt` | Environment variable template |

---

## Installation

### 1. Run Setup

Open `SETUP.md` and follow the phases with your agent. Setup covers discovery, database creation, Relation verification, and configuration.

### 2. Configure environment

```bash
cat config-template/env-template.txt >> ~/.hermes/.env
# Fill in: DISCUSSIONS_DB_ID, TASKS_DB_ID, and Business Core page IDs
```

### 3. Install the skill

```bash
cp SKILL.md ~/.hermes/skills/internal-ops/discussion-todo-tracker/
```

### 4. Start using

Paste meeting notes or describe a conversation. The agent handles the rest.

---

## Example interactions

> "Weekly sync notes: decided to put Singapore market on hold. Kevin to wrap up the Onnet contract by end of month. Pricing discussion — no decision, carry to next week."

→ Agent creates Discussion record, flags Singapore decision as a potential Business Core update (asks to confirm), logs Kevin's task with deadline, preserves pricing as a Pending Follow-up. Asks if there is anything else before marking Done.

---

> "What happened with the pricing topic we raised last week?"

→ Agent searches Task Tracker for Pending Follow-up items related to pricing, reports status, asks if you want to act on it now.

---

## Requirements

- Notion integration token with access to both databases and all Business Core KB pages
- Both databases created and linked via Relation before first use
- Business Core KB page IDs added to `~/.hermes/.env`

→ [Setup guide](./SETUP.md)
