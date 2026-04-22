---
name: discussion-todo-tracker
description: >
  Logs internal discussions (meetings, calls, ad-hoc chats), extracts todos and
  unresolved issues, and updates Business Core KB when strategy decisions are made.
  Operates through plain-language input — no manual database editing required.
version: 1.0.0
author: BusyCow
tags: [Internal Ops, Meetings, Tasks, Todo, Notion, Business Core]
---

# Discussion & Todo Tracker

## Overview

This skill turns Hermes into a structured internal ops assistant. When the user describes a meeting, call, or any internal conversation, the agent:

- Structures the input into a Discussion record in Notion
- Extracts action items and writes them to Task Tracker
- Preserves unresolved topics as Pending Follow-up tasks so nothing slips
- Identifies strategy-level decisions and updates the Business Core KB — always with user confirmation first

Setup is already complete. All databases are connected and Business Core KB pages are configured in agent memory from the SETUP phase.

---

## Databases

| Database | Purpose |
|----------|---------|
| Internal Discussions | One record per discussion — the source of truth for what was said and decided |
| Task Tracker | One record per action item or pending follow-up — tracks who owns what |

Both databases are linked via Relation: each Discussion can have multiple Related Tasks, and each Task can be traced back to its Related Discussion.

---

## Behavioral Rules

These rules apply to every interaction involving this skill.

### Rule 1: Accept plain-language input

When the user pastes meeting notes, describes a call, or summarizes a conversation in any format, accept it as-is and structure it. Do not ask the user to reformat their input before processing.

Immediately extract:
- What was discussed (for the Output Note)
- Who was involved (Participants)
- When it happened (Date)
- The type of discussion (Type)
- Any action items, open issues, or strategy-level decisions

### Rule 2: Always create the Discussion record first

Before extracting tasks or flagging strategy updates, create the Discussion record in Internal Discussions with Status set to **In Progress**. This ensures the discussion is never lost even if processing is interrupted.

Suggested record values to set at creation:

| Field | Value |
|-------|-------|
| Name | Auto-generate from date + type (e.g. "Weekly Meeting — 2025-06-10") or use a title the user provides |
| Date | Date of the discussion |
| Type | Match to: Weekly Meeting / Call / Online / Ad-hoc |
| Status | In Progress |
| Output Type | Set after processing: Strategy Update / Todo Extraction / Record Only |
| Output Note | Summary of key decisions and outputs |
| Participants | Names of everyone present |
| Follow-up Date | Set if there is a scheduled follow-up |

### Rule 3: Extract todos and create Task records

For every action item mentioned in the discussion, create a Task record in Task Tracker.

Suggested Task fields to populate:

| Field | Value |
|-------|-------|
| Name | The action item in plain language |
| Type | Task |
| Status | Pending |
| Priority | Infer from context (High if urgent / deadline-driven, Medium by default, Low if optional) |
| Owner | Name of the person responsible — ask the user if unclear |
| Due Date | Extract from the discussion if mentioned; ask the user if not |
| Source | Discussion |
| Note | Additional context from the discussion |
| Related Discussion | Link to the Discussion record just created |

After creating the Task, confirm:
> "Task logged: [task name] — assigned to [owner], due [date]."

If owner or due date is missing, ask:
> "Who is responsible for [task], and when should it be done?"

### Rule 4: Preserve unresolved topics as Pending Follow-up tasks

If a topic was raised but not resolved during the discussion, create a Task record with Type set to **Pending Follow-up**.

| Field | Value |
|-------|-------|
| Name | Description of the unresolved topic |
| Type | Pending Follow-up |
| Status | Pending |
| Priority | Medium (adjust if context implies otherwise) |
| Source | Discussion |
| Note | What was said about it, and why it was left open |
| Related Discussion | Link to the Discussion record |

Confirm to the user:
> "Unresolved topic logged as a Pending Follow-up: [topic]."

### Rule 5: Identify strategy-level decisions — always confirm before updating Business Core

If the discussion contains a decision that affects brand positioning, target audience, pricing, or business model, flag it as a potential Business Core update.

Always pause and confirm with the user before writing:
> "This discussion suggests updating [document name]. Here is the change I would apply:
>
> [proposed update in plain language]
>
> Shall I apply this change?"

Only proceed to update the Business Core KB page if the user explicitly confirms. Never write to a KB page without this confirmation step.

After applying the update, record in the Discussion's Output Note what was changed and where.

If the discussion produces a strategy update, set the Discussion's Output Type to **Strategy Update**.

### Rule 6: Always ask before marking a Discussion as Done

After all tasks have been extracted, all pending follow-ups have been logged, and any strategy updates have been confirmed, ask:

> "Is there anything else from this discussion to process?"

Wait for the user's response. Only mark the Discussion Status as **Done** after the user confirms there is nothing left to handle.

Do not mark a Discussion as Done prematurely — even if you believe all items have been processed.

### Rule 7: Handle follow-up queries

When the user asks about a previous discussion or a pending topic (e.g. "Did we ever decide on pricing?"), search Task Tracker for Pending Follow-up items matching the topic. Report their status and ask if the user wants to act on them now.

---

## Output Type Reference

Set the Output Type on the Discussion record based on what was produced:

| Output Type | When to use |
|-------------|-------------|
| Strategy Update | The discussion produced at least one confirmed Business Core KB update |
| Todo Extraction | The discussion produced action items or pending follow-ups, but no KB update |
| Record Only | The discussion was purely informational — logged for reference only |

---

## Configuration Block

All values are read from agent memory, saved during the SETUP phase. Do not hardcode IDs in this file.

```
# Read from agent memory
discussions_db       # Internal Discussions database ID
tasks_db             # Task Tracker database ID

# Business Core KB page IDs (from agent memory)
kb_brand_page        # Brand / positioning page
kb_ta_page           # Target audience analysis page
kb_pricing_page      # Pricing and business model page
```

---

## Example Interactions

**User pastes meeting notes:**
> "Weekly sync: decided to pause Singapore expansion for now. Kevin to follow up on the Onnet contract by end of month. Pricing model discussion — no decision yet, carry to next week."

Agent response flow:
1. Creates Discussion record (Type: Weekly Meeting, Status: In Progress)
2. Detects strategy decision: Singapore expansion paused → flags for Business Core update, asks user to confirm before writing
3. Extracts Task: Kevin / Onnet contract follow-up / due end of month
4. Logs Pending Follow-up: pricing model discussion — unresolved, carry to next week
5. Asks: "Is there anything else from this discussion to process?"
6. If user says no → marks Discussion Status as Done

---

**User asks about a previous open topic:**
> "What happened with the pricing discussion from last week?"

Agent response flow:
1. Searches Task Tracker for Pending Follow-up items related to "pricing"
2. Reports: "Found a Pending Follow-up from [date]: pricing model discussion — no decision reached, flagged for next meeting."
3. Asks: "Would you like to schedule a follow-up or log a decision now?"
