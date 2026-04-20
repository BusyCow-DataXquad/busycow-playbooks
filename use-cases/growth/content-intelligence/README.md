# Content Intelligence — Use Case

An AI-powered content system that monitors external sources, proactively suggests topics,
and generates on-brand content grounded in your company's knowledge base.
All content is saved automatically to the Notion Content Library.

**Category:** Growth
**Data Foundation Required:** [Growth](../../data-foundation/growth/README.md) — Content Library only

---

## What This Use Case Does

- On-demand content generation (Email, WhatsApp, Blog, LinkedIn, Case Study, and more)
- External source monitoring (blogs, news, newsletters) as content inspiration
- Agent-suggested new sources — with user approval before adding
- Daily or weekly content briefing to Telegram with topic recommendations
- Auto-save to Notion Content Library with metadata
- Batch generation for full content calendars

---

## Required Data Tables

Install these tables from `data-foundation/growth/` before activating this use case:

| Table | Layer | Purpose |
|-------|-------|---------|
| Content Library | Standalone | Stores all generated content |

> **Also required (created separately):** Sources DB — tracks external sources to monitor

**Sources DB schema:**

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Source name |
| URL | URL | |
| Type | Select | Blog / News / Newsletter / Competitor / Industry Report |
| Topics | Multi-select | Content themes covered |
| Language | Select | en / zh-TW / zh-HK / etc. |
| Status | Select | Active / Paused |
| Added By | Select | User / Agent Suggested |
| Last Fetched | Date | |
| Notes | Rich Text | |

---

## Required Knowledge Base

This use case reads from your Core Business KB to generate on-brand content.
Ensure the following are filled in and accessible:

| KB Document | Why It's Needed |
|-------------|----------------|
| Brand Identity | Sets tone, voice, and boundaries for all content |
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
# Fill in all values
```

### Step 5 — Set Up Briefing (Optional)

The agent will guide you through setting up the weekly/daily cron job on first run.

---

## How to Use

**Generate content on demand:**
- 「幫我寫一封開發信給製造業的客戶」
- 「寫一篇 LinkedIn 文，主題是 AI 如何幫業務節省時間」
- 「根據最新的 Tech in Asia 文章，寫一篇我們的觀點」

**Manage sources:**
- 「幫我追蹤 Tech in Asia 的文章」
- 「有沒有推薦的新來源？」
- 「暫停追蹤 HBR」

**Act on weekly briefing:**
- 「寫第 1 個」
- 「寫第 2 個，改成 Email 格式」

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
