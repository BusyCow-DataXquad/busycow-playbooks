# Content Intelligence Playbook

An AI-powered content intelligence system that monitors external sources, proactively suggests topics, and generates on-brand content grounded in your company's knowledge base — all saved automatically to a Notion Content Library.

## What's Included

- On-demand content generation (Email, WhatsApp, Blog, LinkedIn, Case Study, and more)
- External source monitoring (blogs, news, newsletters) as content inspiration
- Agent-suggested new sources — with user approval before adding
- Daily or weekly content briefing to Telegram with topic recommendations
- Auto-save to Notion Content Library with metadata
- Batch generation for full content calendars

## Stack

- Hermes Agent (AI writer + strategist)
- Notion (Sources Table + Content Library)
- Knowledge Base (Notion pages or local markdown files)
- Telegram (daily/weekly briefing delivery)

---

## Setup Steps

### Step 1 — Prepare Your Knowledge Base

Fill in the templates in `knowledge-base-template/`:

| File | What to Fill In |
|------|----------------|
| `brand-positioning.md` | Who you are, what you do, tone of voice |
| `target-audience.md` | Who you're talking to, their pain points |
| `pricing-and-business.md` | Packages, pricing, differentiators |
| `product-overview.md` | Features, use cases, how you solve problems |

Store as Notion pages (recommended) or local files at `~/.hermes/kb/`.

### Step 2 — Create Notion Databases

You need 2 databases:

**Sources Table** — tracks external sources to monitor
| Field | Type |
|-------|------|
| Name | Title |
| URL | URL |
| Type | Select (Blog / News / Newsletter / Competitor / Industry Report) |
| Topics | Multi-select |
| Language | Select |
| Status | Select (Active / Paused) |
| Added By | Select (User / Agent Suggested) |
| Last Fetched | Date |
| Notes | Rich Text |

**Content Library** — stores generated content
| Field | Type |
|-------|------|
| Name | Title |
| Type | Select (Email / WhatsApp Message / Blog Post / LinkedIn / Case Study / Other) |
| Stage Target | Multi-select |
| Industry Target | Multi-select |
| Channel | Multi-select |
| Status | Select (Draft / Ready / Archived) |
| Source | Relation → Sources |
| Created | Created Time |

Share both databases with your Notion integration.

### Step 3 — Install the Skill

```bash
cp -r skills/sales-growth/content-intelligence ~/.hermes/skills/sales-growth/
```

### Step 4 — Configure

```bash
cat config-template/env-template.txt >> ~/.hermes/.env
# Fill in the values
```

### Step 5 — Set Up Briefing (Optional)

The agent will guide you through setting up the weekly/daily cron job on first run,
or you can set it up manually following the instructions in `SKILL.md`.

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
│   └── sales-growth/
│       └── content-intelligence/
│           └── SKILL.md                  ← Core skill (install this)
├── knowledge-base-template/
│   ├── brand-positioning.md
│   ├── target-audience.md
│   ├── pricing-and-business.md
│   └── product-overview.md
└── config-template/
    └── env-template.txt
```
