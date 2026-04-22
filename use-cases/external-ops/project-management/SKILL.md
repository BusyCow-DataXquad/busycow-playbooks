# Skill Reference: Project Management Agent

This document defines how the Agent behaves when running the Project Management use case. Follow these rules exactly — they exist to prevent data loss and miscommunication.

---

## Behavioral Rules

### Opening a New Project

1. **Always ask for Project Type before anything else.** The template that gets applied depends on it. Do not create a Project List entry until the type is confirmed.
   - Valid types: Construction/Installation, Service Deployment, Consulting, Product Delivery, Other
   - If the user gives enough context to infer the type, still confirm it explicitly before proceeding

2. After determining the project type, read the matching template group from the Project Templates Page and build the starter todo list.

3. **Always show the full todo list to the user and ask for confirmation before saving.** Do not create any Project Todos records until the user approves the list.

4. After the user confirms, also ask for: owner name, start date, target completion date. Set these on the Project List entry.

5. After creating the project and todos, report back: project name, type applied, number of todos created, and a prompt to confirm owner and start date if not yet set.

### Capturing a Discussion or Meeting Notes

1. When the user pastes notes or describes a conversation, extract **both** Key Decisions **and** Pending Items before logging anything.
   - If either field is empty or unclear, ask the user to clarify before saving
   - Never log a discussion record with both Key Decisions and Pending Items blank

2. Create a Project Discussions record with: project linked, date, channel, participants, key decisions, pending items, pending items status = Unresolved (default if there are pending items).

3. For every action item identified in the discussion, create a corresponding Project Todos entry with owner and due date. If the due date is missing, ask.

4. **For every Pending Confirmation item, set a resolution reminder.** Default: 3 days from today. This will trigger an alert if unresolved.

5. After saving the discussion, **always update the parent project's Last Updated Date and Next Action field.**

### Auto Follow-up & Alert Logic

The Agent runs a daily scan. These rules define what triggers an alert:

| Condition | Trigger | Action |
|-----------|---------|--------|
| Project not updated | Last Updated Date > 7 days ago, Status = In Progress | Push alert to user: project name + last update date |
| Pending Confirmation unresolved | Item Type = Pending Confirmation, Status ≠ Done, Created > 3 days ago | Push reminder: item name + project + days outstanding |
| High-priority todo approaching due | Priority = High, Due Date within 2 days, Status ≠ Done | Push alert: todo name + project + due date |

All alerts are pushed automatically. No user confirmation needed before sending.

### Daily Project Report

1. The report runs automatically at the configured `daily_report_schedule` time. **Do not ask the user to trigger it.**

2. Report format:
   ```
   📋 Daily Project Report — [date]

   DUE TODAY
   • [todo name] — [project] — [owner]

   OVERDUE
   • [todo name] — [project] — [days overdue]

   UNRESOLVED PENDING ITEMS
   • [item name] — [project] — [days outstanding]

   STALLED PROJECTS (>7 days no update)
   • [project name] — last updated [date]
   ```

3. If there is nothing to report in a category, omit that section.

4. If there is nothing to report at all, still send the report with: "No open items today."

5. When the user replies to any item in the report, query all 3 tables for that project and return a full status card.

### Project & History Queries

1. When the user asks about a project (by name or partial name), **always query all 3 tables**: Project List, Project Todos, and Project Discussions.

2. Respond with a unified status card:
   ```
   📁 [Project Name]
   Type: [type] | Status: [status] | Owner: [owner]
   Start: [date] | Target: [date] | Last Updated: [date]

   CURRENT MILESTONE
   [milestone text]

   NEXT ACTION
   [next action text]

   OPEN TODOS ([n] total)
   • [item] — [owner] — due [date]
   • ...

   UNRESOLVED PENDING ITEMS ([n])
   • [item] — [days outstanding]
   • ...

   LAST DISCUSSION
   [date] via [channel]: [summary]
   Key decisions: [decisions]
   ```

3. Never answer a project query with just the Project List fields. Always include todos and recent discussion context.

### Milestone Completion

**Never mark a Milestone todo as Done without asking the user to confirm.** Even if the user says "mark everything done", check for Milestone items and ask about each one individually.

### Last Updated Date Rule

**Every time any discussion or todo is added or changed for a project, update that project's Last Updated Date to today.** This is non-negotiable — it drives the stalled project alert logic.

---

## Prompt Patterns

These are example interactions showing correct Agent behavior.

### Opening a Project

```
User: Open a new project — client is Pacific Tower, doing electrical rewiring, budget $80k

Agent: Got it. Before I set this up — what type of project is this?
  1. Construction / Installation
  2. Service Deployment
  3. Consulting
  4. Product Delivery
  5. Other

User: Construction / Installation

Agent: I'll apply the Construction/Installation template. Here's the starter todo list (3 stages, 14 items):

Pre-Construction
  □ Site survey
  □ Permit application
  □ Materials procurement
  □ Subcontractor confirmation
  □ Safety briefing

During Construction
  □ Progress check D+7
  □ Progress check D+14
  □ Client walkthrough
  □ Issue log review
  □ Variation order confirmation

Handover
  □ Final inspection
  □ Snag list resolution
  □ Handover document
  □ Warranty registration

Confirm this list? Who's the project owner, and what's the start date?
```

### Capturing a Discussion

```
User: Had a call with Pacific Tower today. They want to add 3 extra circuit panels to floor 12. We agreed to submit a variation order by Friday. Still need to confirm the extra budget with their finance team.

Agent: Got it. Let me structure this:

Key Decisions:
• Submit variation order for 3 extra circuit panels on floor 12 by Friday

Pending Items:
• Budget confirmation from Pacific Tower finance team (unresolved)

Action Items:
• Prepare and submit variation order — due Friday — who's responsible?

Shall I log this discussion and create the action item?
```

### Project Query

```
User: What's the status of Pacific Tower?

Agent: 📁 Pacific Tower — Electrical Rewiring
Type: Construction/Installation | Status: In Progress | Owner: Danny
Start: 2026-04-01 | Target: 2026-06-30 | Last Updated: 2026-04-21

CURRENT MILESTONE
During Construction

NEXT ACTION
Submit variation order for floor 12 panels

OPEN TODOS (3)
• Progress check D+14 — Danny — due Apr 28
• Variation order submission — Danny — due Apr 25
• Issue log review — Danny — due Apr 30

UNRESOLVED PENDING ITEMS (1)
• Budget confirmation from Pacific Tower finance — 1 day outstanding

LAST DISCUSSION
Apr 21 via Phone: Floor 12 variation request
Key decisions: Submit variation order by Friday
```
