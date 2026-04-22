# Use Case: Project Management

> Free the project owner from being a human information router.

## The Problem

Project information is scattered across WhatsApp groups, email threads, and individual notebooks. When anyone wants to know "what stage is this project at?" or "what did we last tell the client?", the answer is always: ask someone. PMs spend more time chasing status than doing real work.

## What This Use Case Does

This use case gives every project a structured home in Notion and puts the Agent in charge of the repetitive tracking work:

- New projects are opened with a matching todo template applied automatically
- Meeting notes and WhatsApp conversations are structured and logged in seconds
- Overdue items, stalled projects, and unresolved confirmations surface automatically
- Every morning, a project report lands in your chat without you asking for it
- Any question about any project gets a complete, synthesized answer instantly

## Data Architecture

This use case relies on **3 Notion databases** and **1 standalone template page**.

### 1. Project List (anchor table)

| Field | Type | Notes |
|-------|------|-------|
| Project Name | Title | Primary identifier |
| Project Type | Select | Construction/Installation, Service Deployment, Consulting, Product Delivery, Other |
| Client / Owner | Rich Text | Client name or internal owner |
| Status | Status | In Progress, On Hold, Completed |
| Start Date | Date | |
| Target Completion Date | Date | |
| Last Updated Date | Date | Auto-updated by Agent on every change |
| Current Milestone | Rich Text | Active phase or deliverable |
| Next Action | Rich Text | What needs to happen next |

### 2. Project Todos (recommended)

| Field | Type | Notes |
|-------|------|-------|
| Item Name | Title | |
| Project | Relation | → Project List |
| Item Type | Select | Task, Pending Confirmation, Pending Decision, Milestone |
| Priority | Select | High, Medium, Low |
| Owner | Rich Text | Person responsible |
| Source Discussion / Meeting | Rich Text | Where this item came from |
| Due Date | Date | |
| Status | Status | Pending, In Progress, Done |

### 3. Project Discussions (recommended)

| Field | Type | Notes |
|-------|------|-------|
| Summary | Title | One-line description of the discussion |
| Project | Relation | → Project List |
| Date | Date | |
| Channel | Select | WhatsApp, Email, Meeting, Telegram, Other |
| Participants | Rich Text | Who was involved |
| Key Decisions | Rich Text | Decisions made in this discussion |
| Pending Items | Rich Text | Items that need follow-up or confirmation |
| Pending Items Status | Select | Unresolved, Resolved, N/A |

### 4. Project Templates Page (standalone Notion page)

A single Notion page (not a database) containing 3 template groups:

**Construction / Installation** — 3 stages, 14 todos
- Pre-construction: site survey, permit application, materials procurement, subcontractor confirmation, safety briefing
- During construction: progress check D+7, progress check D+14, client walkthrough, issue log review, variation order confirmation
- Handover: final inspection, snag list resolution, handover document, warranty registration

**Service Deployment** — 3 stages, 11 todos
- Pre-deployment: requirements sign-off, environment setup, test plan approval, training schedule confirmed, data migration checklist
- Deployment: go-live confirmation, user acceptance test, issue log, rollback plan confirmed
- Post-deployment: 7-day review, documentation handover

**General** — 6 starter todos
- Kickoff meeting, scope confirmation, stakeholder alignment, first milestone set, comms channel established, review schedule agreed

The Agent reads the matching template group when opening a new project and creates the todo items automatically.

## Features

### 1. Project Opening & Template Application
When a user opens a new project, the Agent asks for the project type first. It then reads the matching template from the Templates Page and creates a complete starter todo set. The Agent confirms the list with the user before saving, and sets the owner and key dates.

### 2. Discussion & Note Capture
The user pastes meeting notes or describes a conversation in plain language. The Agent structures it: creates a Project Discussions record with key decisions and pending items extracted, creates Action Items in Project Todos with owners and due dates, and updates the project's Last Updated Date and Next Action field.

### 3. Auto Follow-up & Missing Info Alerts
The Agent runs a daily scan:
- Projects not updated in more than 7 days → alert
- Pending Confirmation items not resolved in more than 3 days → push reminder
- High-priority todos approaching due date → alert

All alerts are pushed to the user's active chat automatically.

### 4. Daily Project Report
Every morning the Agent scans all In Progress projects and compiles a report:
- Todos due today
- Overdue items
- Unresolved pending items
- Projects stalled for more than 7 days

The report is delivered automatically. The user can reply to any item to get full context.

### 5. Project & History Query
The user can ask any question about any project — current stage, last interaction, open items, who's responsible for what. The Agent queries across all 3 tables and responds with a unified status card.

## Quick Start

See [SETUP.md](./SETUP.md) to connect your Notion workspace.
See [SKILL.md](./SKILL.md) for Agent behavioral rules and prompt patterns.
