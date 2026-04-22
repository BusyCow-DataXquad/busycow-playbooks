# Discussion & Todo Tracker — Setup Guide

Instructions for the agent to set up this use case for a new client.

---

## Overview

This use case requires two linked Notion databases and access to the client's Business Core KB pages. Walk through the phases below in order. Do not skip the Adapt phase — the Relation between Internal Discussions and Task Tracker is required for the skill to function correctly.

---

## Phase 1: Discover

Search the client's Notion workspace to find any existing databases or pages that may already serve these functions.

### Search for existing databases

Run searches using these keywords (one at a time, note any matches):

- `discussion`
- `meeting notes`
- `task`
- `todo`
- `tracker`

For each match found, show the client the name and type (database / page), and ask:
> "Is this the database you want to use, or should we create a new one?"

If a suitable database exists, proceed to Adapt. If not, proceed to Build.

### Search for Business Core KB pages

The Business Core KB consists of standalone Notion pages — not databases. Search for pages containing these keywords:

- `brand` or `brand positioning`
- `target audience` or `TA`
- `pricing` or `business model`

Present the matches to the client and confirm which pages are the active Business Core KB:
> "I found these pages that look like Business Core documents — [list]. Are these the ones you want the agent to update when strategy decisions are made?"

Record the page IDs for all confirmed Business Core pages. These will be saved to agent memory in the Configure phase.

---

## Phase 2: Build

If the client does not have existing databases, create them in Notion now.

### Create: Internal Discussions

Create a new Notion database named **Internal Discussions** with the following suggested schema:

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Discussion title or auto-generated from date + type |
| Date | Date | When the discussion took place |
| Type | Select | Weekly Meeting / Call / Online / Ad-hoc |
| Status | Select | Pending / In Progress / Done |
| Output Type | Select | Strategy Update / Todo Extraction / Record Only |
| Output Note | Rich Text | Summary of key outputs or decisions |
| Participants | Rich Text | Names of everyone involved |
| Follow-up Date | Date | When to revisit, if applicable |
| Related Tasks | Relation → Task Tracker | Linked action items (set up after Task Tracker is created) |

Confirm the schema with the client before creating:
> "Here is the recommended schema for Internal Discussions. Would you like to adjust any fields before I create it?"

### Create: Task Tracker

Create a new Notion database named **Task Tracker** with the following suggested schema:

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Task description |
| Type | Select | Task / Pending Follow-up |
| Status | Select | Pending / In Progress / Done / Cancelled |
| Priority | Select | High / Medium / Low |
| Owner | Rich Text | Name of the person responsible |
| Due Date | Date | Target completion date |
| Source | Select | Discussion / Business Core / Self-initiated / Other |
| Note | Rich Text | Additional context or instructions |
| Related Discussion | Relation → Internal Discussions | The discussion this task came from |

Confirm with the client before creating:
> "Here is the recommended schema for Task Tracker. Would you like to adjust any fields?"

---

## Phase 3: Adapt

Once both databases exist (either discovered or built), verify the Relation between them.

### Check: Relation between databases

Check whether Internal Discussions has a **Related Tasks** field of type Relation pointing to Task Tracker, and whether Task Tracker has a **Related Discussion** field of type Relation pointing to Internal Discussions.

If the Relation is missing or only set up on one side, inform the client:
> "The two databases are not yet linked by a Relation property. This is required so that tasks can be traced back to the discussion they came from. Shall I walk you through adding the Relation in Notion, or would you prefer to do it yourself?"

Do not proceed until the Relation is in place on both sides.

### Confirm: Business Core KB page IDs

Verify that all confirmed Business Core KB page IDs from Phase 1 are recorded. Ask the client to confirm which documents are in scope for auto-update:
> "I'll update these Business Core pages when a discussion produces a strategy-level decision: [list confirmed pages]. Should I add or remove any from this list?"

---

## Phase 4: Configure

Once all databases are set up and the Relation is confirmed, collect and save the configuration values using the memory tool.

Retrieve each database ID from the Notion URL (the 32-character hex string after the last `/`). Retrieve each page ID in the same way.

Use the memory tool to save all IDs under a clearly labeled memory entry:

```
Discussion & Todo Tracker config:
  discussions_db: <Internal Discussions database ID>
  tasks_db: <Task Tracker database ID>
  kb_brand_page: <Brand Positioning page ID>
  kb_ta_page: <Target Audience page ID>
  kb_pricing_page: <Pricing / Business Model page ID>
```

Remind the client that the Notion integration must be granted access to all databases and pages listed above — otherwise the agent will not be able to read or write to them.

---

## Phase 5: Verify

Run a quick end-to-end check before going live.

1. Confirm the agent can read from Internal Discussions (try fetching any entry, or confirm the DB is empty and ready)
2. Confirm the agent can read from Task Tracker
3. Confirm the agent can read from at least one Business Core KB page
4. Confirm the Relation fields are visible in both databases

If all checks pass, confirm to the client:
> "Setup is complete. Internal Discussions and Task Tracker are connected, and Business Core KB pages are linked. You can now start logging discussions."

If any check fails, trace back to the relevant phase and resolve before confirming completion.

---

## Post-Setup

After verification, the client uses this use case daily via **SKILL.md**. No further setup is needed unless they want to add new Business Core KB pages or change the database schema.

To add more Business Core KB pages later, retrieve the page ID and save it to agent memory, then inform the agent which document it refers to.
