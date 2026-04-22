# Content Intelligence — Use Case

Creating consistent, on-brand content is one of the most time-consuming tasks for small business teams. Without a system, posts get written in a rush, brand voice drifts across channels, and good topic ideas get lost because nobody had time to act on them. The Content Intelligence use case gives your team an AI content assistant that already knows your brand: it monitors relevant external sources, delivers fresh topic ideas every morning, and writes on-brand content — grounded in your company knowledge base — ready to publish or tweak.

Every piece of content this assistant produces is anchored to your brand positioning, target audience, and content styling guide. It will never invent product claims or copy-paste from sources. It shows you a draft and waits for your approval before saving anything.

---

## Features

- **Source Discovery** — recommends external blogs, news sites, and newsletters relevant to your business; presents candidates for your approval before adding anything
- **Daily Topic Briefing** — every morning, scans active sources and delivers 3–5 specific topic suggestions with recommended format and angle; reply "Write #1" to start immediately
- **Content Writing** — generates any content format grounded in the company KB and Content Styling Guide:
  - *Short-form* (Social Media, WhatsApp, LINE, Email): writes with a strong opening hook, auto-recommends relevant hashtags based on topic and platform
  - *Long-form* (Blog, Article, Newsletter, Case Study): runs keyword research first, embeds SEO keywords naturally throughout, writes in-depth
- **Content Styling Guide** — a Notion page where your brand's voice, format rules, SEO keywords, AEO targets, and platform-specific rules are defined; loaded automatically before every draft; updated via conversation

---

## What's in This Package

| File | Purpose |
|------|---------|
| README.md | This file — human overview, features, and install steps |
| SETUP.md | Agent-facing setup instructions — run once during onboarding |
| SKILL.md | Agent-facing daily operating instructions — loaded on every use |
| config-template/env-template.txt | Environment variable template — copy and fill in |

---

## Install Steps

### Step 1 — Copy config template

```bash
cp use-cases/sales-growth/content-intelligence/config-template/env-template.txt ~/.hermes/.env
```

Open `~/.hermes/.env` and fill in whatever values you already know. Leave the rest blank — SETUP.md will guide the agent through finding or creating everything.

### Step 2 — Copy SKILL.md to your Hermes skills folder

```bash
mkdir -p ~/.hermes/skills/growth/content-intelligence
cp use-cases/sales-growth/content-intelligence/SKILL.md ~/.hermes/skills/growth/content-intelligence/SKILL.md
```

### Step 3 — Run the setup

Tell Hermes:

> "Run the Content Intelligence setup — use SETUP.md from the content-intelligence use case."

The agent will search for existing Notion databases and pages, help create anything missing (including the Content Styling Guide), adapt schemas to add recommended fields, and save all IDs to `~/.hermes/.env`.

### Step 4 — Set up daily topic briefing (optional)

```bash
hermes cron create \
  --schedule "0 1 * * *" \
  --name "content-intelligence-daily" \
  --prompt "Run the Content Intelligence daily briefing: scan Active Sources, suggest 3-5 topics grounded in KB, deliver to Telegram."
```

Adjust the schedule to match your timezone (01:00 UTC = 09:00 Taiwan time).

### Step 5 — Set up Telegram

1. Create a bot via @BotFather and copy the bot token
2. Add the bot to your team group or channel
3. Note the Chat ID and Thread ID (if using topics)
4. Add `TELEGRAM_BOT_TOKEN`, `CONTENT_TELEGRAM_CHAT_ID`, and `CONTENT_TELEGRAM_THREAD_ID` to `~/.hermes/.env`
