# Content Intelligence — Use Case

An AI-powered content system that helps you find the right sources, get daily topic suggestions, and generate on-brand content — all grounded in your company's knowledge base and saved automatically to Notion.

**Category:** Growth
**Data Foundation Required:** [Growth](../../data-foundation/growth/README.md) — Content Library only

---

## What This Use Case Does

### Feature 1 — Source Discovery
Reads your Core Business KB (brand positioning, TA, industry focus) and recommends external blogs, news sites, and newsletters worth tracking as writing inspiration. Suggestions are presented for your approval before being added to the Sources DB. You can also add sources manually at any time.

### Feature 2 — Daily Topic Briefing
Every morning at 9am (alongside the Daily CRM Report), the agent scans Active Sources for fresh articles, cross-references your brand KB and target audience, then delivers 3–5 specific topic suggestions — each with a recommended format and angle. Reply "Write #1" to start writing immediately.

### Feature 3 — Content Writing
Generate any type of content through natural conversation: long-form Blog posts, Social Media posts (LinkedIn, IG, Facebook), Email campaigns, WhatsApp messages, and more. The agent always references the brand KB to ensure tone and positioning consistency. All confirmed content is saved to the Content Library with metadata.

---

## Required Data Tables

Install these tables from `data-foundation/growth/` before activating this use case:

| Table | Layer | Purpose |
|-------|-------|---------|
| Content Library | Standalone | Stores all generated content |

> **Also required (create separately):** Sources DB — tracks external sources to monitor. Schema below.

**Sources DB schema:**

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Source name (e.g. "Tech in Asia") |
| URL | URL | Homepage or RSS feed URL |
| Type | Select | Blog / News / Newsletter / Industry Report |
| Topics | Multi-select | Content themes covered |
| Language | Select | en / zh-TW / zh-HK / etc. |
| Status | Select | Active / Paused |
| Added By | Select | User / Agent Suggested |
| Last Fetched | Date | Auto-updated by agent |
| Notes | Rich Text | Why this source is relevant |

---

## Required Knowledge Base

This use case reads from your Core Business KB to generate on-brand content.
Ensure the following are filled in and accessible:

| KB Document | Why It's Needed |
|-------------|----------------|
| Brand Identity | Sets tone, voice, and positioning for all content |
| Target Audience | Informs who the content is written for |
| Offer | Grounds product/service mentions in accurate detail |

See `data-foundation/core-business/` for setup instructions.

---

## Setup Steps

### Step 1 — Prepare Knowledge Base

Fill in `data-foundation/core-business/templates/` and set up as Notion pages or local files.

### Step 2 — Create Notion Databases

1. Create Content Library from `data-foundation/growth/README.md`
2. Create Sources DB (schema above) — share both with your Notion integration

### Step 3 — Install the Skill

```bash
cp -r skills/growth/content-intelligence ~/.hermes/skills/growth/
```

### Step 4 — Configure

```bash
cat config-template/env-template.txt >> ~/.hermes/.env
# Fill in NOTION_TOKEN, SOURCES_DB, CONTENT_LIBRARY_DB, KB page IDs
```

### Step 5 — Set Up Daily Briefing (Optional)

The agent will guide you through setting up the daily cron job on first run, or you can set it up manually:

```bash
# Daily 9am Taiwan time (01:00 UTC)
hermes cron create \
  --schedule "0 1 * * *" \
  --name "content-intelligence-daily" \
  --prompt "Run the Content Intelligence daily briefing: scan Active Sources, suggest 3-5 topics grounded in KB, deliver to Telegram."
```

---

## How to Use

**Discover sources:**
- "Which external sources do you think we should be tracking?"
- "Add a new source: https://example.com"
- "Pause tracking HBR"

**Act on daily briefing:**
- "Write #1"
- "Write #2, but make it Email format"
- "Save #3 as a draft title — I'll write it next week"

**Write content on demand:**
- "Write a LinkedIn post about how AI helps sales teams save time"
- "Based on today's Tech in Asia article, write a perspective piece from our angle"
- "Prepare this month's lead outreach materials — one Email and one WhatsApp"

---

## Folder Structure

```
content-intelligence/
├── README.md
├── skills/
│   └── growth/
│       └── content-intelligence/
│           └── SKILL.md          ← Install this
└── config-template/
    └── env-template.txt
```
