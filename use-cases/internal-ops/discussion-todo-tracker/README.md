# Discussion & Todo Tracker

Log meetings in plain language, extract todos and open issues, auto-update Business Core when strategy shifts.

---

## What it does

Every company has discussions that matter — weekly syncs, quick calls, Telegram threads. The problem is most of it vanishes: decisions unmade, todos floating in someone's head, half-finished conversations nobody follows up on.

This use case gives the agent a structured way to handle all of it:

1. **Log** — paste meeting notes or describe a conversation in plain language; the agent structures it into a Discussion record
2. **Extract todos** — any action items get pulled out and logged in Task Tracker with owner and due date
3. **Preserve open issues** — topics that weren't resolved become "Pending Follow-up" tasks so nothing slips through
4. **Update Business Core** — if a discussion changes strategy, pricing, or positioning, the agent updates the relevant KB document directly

---

## Sub-functions

| # | Function | Description |
|---|----------|-------------|
| 1 | Plain-language input | Paste meeting notes or describe a conversation; agent handles structuring |
| 2 | Key info extraction + Business Core update | Strategy shifts → agent updates KB directly |
| 3 | Todo extraction | Action items → Task Tracker with owner + deadline |
| 4 | Open issue preservation | Unresolved topics → "Pending Follow-up" tasks |
| 5 | Status tracking | Every discussion has a Status — nothing is "done" until it's processed |

---

## Required Data Tables

| Table | Source |
|-------|--------|
| Internal Discussions | `data-foundation/internal-ops/` |
| Task Tracker | `data-foundation/internal-ops/` |

→ [Data setup guide](../../../data-foundation/internal-ops/README.md)

Also requires **Core Business KB** pages (for the Business Core update function).

---

## Installation

### 1. Set up data tables

Follow [data-foundation/internal-ops/README.md](../../../data-foundation/internal-ops/README.md).

### 2. Install the skill

```bash
cp -r skills/internal-ops/discussion-todo-tracker \
      ~/.hermes/skills/internal-ops/
```

### 3. Configure

```bash
cat config-template/env-template.txt >> ~/.hermes/.env
# Fill in DISCUSSIONS_DB_ID, TASKS_DB_ID, and Business Core page IDs
```

---

## Example interactions

> "Weekly sync notes: decided to put Singapore market on hold, Kevin to wrap up the Onnet contract within two weeks, pricing adjustment to be discussed next week"

→ Agent creates Discussion record, extracts 1 strategy update (→ Business Core), 1 todo (Kevin / Onnet contract), 1 open issue (pricing → Pending Follow-up task)

---

> "That pricing topic we discussed last time — did we ever reach a conclusion?"

→ Agent checks Task Tracker for Pending Follow-up items related to pricing, reports status, asks if you want to schedule a follow-up
