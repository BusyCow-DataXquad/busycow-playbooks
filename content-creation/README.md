# Content Creation Playbook

An AI-powered content generation system that produces on-brand marketing and sales content grounded in your company's core business knowledge base.

## What's Included

- Brand-aware content generation (Email, WhatsApp, Blog, LinkedIn, Case Study, and more)
- Automatic Knowledge Base loading before every content piece
- Confirmation flow before saving — no surprise writes
- Auto-save to Notion Content Library with proper metadata
- Batch generation for multiple content pieces at once

## Stack

- Hermes Agent (AI writer)
- Notion (Content Library storage)
- Knowledge Base (Notion pages or local markdown files)

---

## Setup Steps

### Step 1 — Prepare Your Knowledge Base

Create the following documents (as Notion pages or local `.md` files):

| Document | What to Include |
|----------|----------------|
| Brand Positioning | Who you are, what you do, unique value, tone of voice |
| Target Audience | Who you're talking to, their pain points and goals |
| Pricing & Business Model | Packages, pricing logic, key differentiators |
| Product / Service Overview | Features, use cases, what problems you solve |

See `knowledge-base-template/` for starter templates.

### Step 2 — Set Up Notion Content Library

Create a Notion database with these fields (or reuse an existing one):

| Field | Type |
|-------|------|
| Name | Title |
| Type | Select |
| Stage Target | Multi-select |
| Industry Target | Multi-select |
| Channel | Multi-select |
| Status | Select (Draft / Ready / Archived) |

Share the database with your Notion integration.

### Step 3 — Install the Skill

```bash
cp -r skills/sales-growth/content-creation ~/.hermes/skills/sales-growth/
```

### Step 4 — Configure

Fill in `config-template/env-template.txt` and add to `~/.hermes/.env`.

---

## How to Use

Just talk to the agent:

- 「幫我寫一封開發信給製造業的客戶」
- 「寫一篇 LinkedIn 文，主題是 AI 如何幫業務節省時間」
- 「幫我準備這個月 Lead 階段的開發素材，Email + WhatsApp 各一篇」

The agent will confirm the brief, load your KB, generate a draft, and save to Content Library on your approval.

---

## Folder Structure

```
content-creation/
├── README.md
├── skills/
│   └── sales-growth/
│       └── content-creation/
│           └── SKILL.md              ← Core skill (install this)
├── knowledge-base-template/
│   ├── brand-positioning.md          ← Fill in your brand info
│   ├── target-audience.md            ← Fill in your TA profile
│   ├── pricing-and-business.md       ← Fill in your pricing model
│   └── product-overview.md           ← Fill in your product info
└── config-template/
    └── env-template.txt              ← Environment variables
```
