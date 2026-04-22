---
name: content-intelligence
description: >
  AI-powered content intelligence assistant for Hermes Agent.
  Discovers relevant external sources, delivers daily topic suggestions,
  and generates on-brand content (Blog, Social, Email, WhatsApp)
  grounded in the company KB and Content Styling Guide.
  Works with whatever databases and pages were set up during onboarding.
version: 2.0.0
author: BusyCow
tags: [Content Intelligence, Content Creation, Notion, Brand, Sources, Daily Briefing]
---

# Content Intelligence — Daily Operations

## Overview

Setup is already complete — databases and KB page IDs are in `~/.hermes/.env`. This skill covers daily content operations.

Features:
1. **Source Discovery** — reads brand KB and recommends external sources worth tracking; presents for approval before adding anything
2. **Daily Topic Briefing** — scans active sources every morning, cross-references KB, delivers 3–5 topic suggestions with format and angle recommendations
3. **Content Writing** — generates any content format grounded in KB and Content Styling Guide; shows draft before saving
4. **Content Styling Guide** — manages the brand's writing style page; loaded automatically before every draft; updated via conversation

The key rule: **always load the KB and Content Styling Guide before writing**. Never produce content without them.

> Field names in your client's databases may differ from the recommended schema. Always check the actual field names when querying or writing.

---

## Feature 1 — Source Discovery

### Triggers

- User asks: "Which external sources should we be tracking?"
- User pastes a URL: "Add this source: https://..."
- Proactively, when the sources database has fewer than 3 active sources

### Workflow — Agent-Suggested Sources

1. Load the brand KB (brand positioning, target audience, industry focus)
2. Based on the KB, identify relevant source types: industry news, competitor blogs, analyst research, newsletters, etc.
3. Web-search for 4–6 candidate sources that match the client's market and language
4. Present for review — **never add sources automatically**:

```
📰 Based on your KB, I recommend tracking these sources:

1. [Source Name] — [URL]
   Type: [Blog / News / Newsletter]
   Language: [zh-TW / en]
   Why it's relevant: [1 sentence tied to brand topic or target audience]

2. [Source Name] — [URL]
   ...

Which ones would you like to add?
```

5. On confirmation: create a record in the sources database with status set to Active and the added-by field set to "Agent Suggested"

### Workflow — User-Added Sources

1. User pastes a URL
2. Fetch and summarize what the source covers
3. Confirm before adding: "This source mainly covers [topics] — suggested language tag: [lang]. Would you like to add it?"
4. On confirmation: create record with the added-by field set to "User"

### Managing Sources

- "Pause tracking [Source]" → `PATCH /pages/{id}` — set status field to Paused
- "Remove this source" → archive the record (`archived: true`)
- "Show me all active sources" → query sources database filtered by Active status

---

## Feature 2 — Daily Topic Briefing

### Trigger

Scheduled cron job (default: 01:00 UTC = 09:00 Taiwan time)

### Workflow

1. Query the sources database — filter by Active status; retrieve all source URLs
2. Run web extraction on each source to get the latest 3–5 article titles and summaries
3. Load the brand KB (brand positioning + target audience)
4. From all source articles, select the 3–5 topics most worth writing about. Selection criteria:
   - Relevant to the brand's target audience
   - Can be used to express the brand's perspective
   - Current events are a bonus but not required
5. Deliver the briefing to Telegram

### Briefing format

```
📝 Today's Writing Topics (Content Intelligence)

1️⃣ [Topic Title]
Inspiration source: [Source Name]
Why it's worth writing: [1–2 sentences — relevance to brand or target audience]
Recommended format: [Blog / LinkedIn / IG / Email / WhatsApp]
Recommended angle: [Specific hook, 1 sentence]

2️⃣ ...

3️⃣ ...

💡 Reply "Write #N" or "Write #N as [format]" to get started.
```

### Acting on the briefing

After delivery, user may reply:
- "Write #1" → execute Feature 3 with suggested topic and format
- "Write #2 as Email format" → override format, then execute Feature 3
- "Save #3 as a draft title — I'll write it next week" → create a placeholder entry in Content Library (Status: Draft, empty body)

---

## Feature 3 — Content Writing

### Triggers

- "Write #N" (from the daily briefing)
- "Write a cold outreach email for manufacturing industry clients"
- "Write a LinkedIn post about how AI helps sales teams save time"
- "Based on today's Tech in Asia article, write a perspective piece from our angle"
- "Prepare this month's Lead outreach materials — one Email and one WhatsApp"

### Step 1 — Clarify (only if request is ambiguous)

```
Got it! Let me confirm a few details:

📌 Topic: [restate]
📄 Format: [Blog / LinkedIn / IG / Email / WhatsApp / ...]
🎯 Target audience: [from KB or user-specified]
🏷️ Stage: [Lead / Qualified / Closing / All]
🌐 Language: [zh-TW / en]

Does this look right?
```

If the request is clear (e.g. continuing from a briefing), skip clarification and proceed.

### Step 2 — Load KB and Styling Guide

Always load before writing. Use the IDs from `~/.hermes/.env`.

| Content Type | What to load |
|-------------|-------------|
| Blog / Long-form | Brand Positioning + Target Audience + Product Overview + Content Styling Guide |
| Social Media | Brand Positioning + Target Audience + Content Styling Guide |
| Cold Email | Brand Positioning + Target Audience + Product Overview + Content Styling Guide |
| Follow-up message | Brand Positioning + Target Audience + Content Styling Guide |
| Case Study | Product Overview + Pricing + Content Styling Guide |

The Content Styling Guide is always loaded — it defines format, tone, platform rules, SEO/AEO targets, and special requirements that override any generic defaults.

### Step 3 — Pull source material (if applicable)

If the user references a specific source or the topic came from the daily briefing:
1. Fetch the source URL (web extract)
2. Extract key points relevant to the topic
3. Use as supporting material — **always rewrite in brand voice, never copy-paste**

### Step 4 — Generate and present draft

Rules:
- Match the brand tone from the KB (no generic AI filler: "In today's fast-paced world...")
- Ground all product or service claims in the KB — do not invent features
- Include a clear CTA unless the format doesn't call for one
- Output in the language specified (default: CONTENT_LANGUAGE from `.env`)

Present the draft:

```
✍️ Content Draft

[TYPE] — [TOPIC]
Target audience: [audience] | Language: [lang] | Source: [source name or "Internal KB"]

---

[GENERATED CONTENT]

---

Any adjustments needed? (tone / length / angle)
Confirm to save to Content Library.
```

### Step 5 — Save to Content Library

After user confirmation (or "save it directly"):
- Create a new page in the Content Library database with the content metadata as properties
- Write the full content into the page body as Notion blocks
- Use the title format: `[Type] — [Topic] — [YYYY-MM-DD]`
- Set status to Draft unless the user specifies otherwise

Confirm: `✅ Saved to Content Library — [Title] (Status: Draft)`

### Batch writing

Trigger: "Prepare this month's Lead outreach materials — one Email, one WhatsApp, and one LinkedIn post"

1. Clarify once for all pieces (topic / angle / audience)
2. Load KB and relevant sources once
3. Generate all pieces sequentially
4. Present all drafts together for review
5. Save all confirmed pieces in one batch
6. Confirm: `✅ 3 pieces saved to Content Library`

### Content refresh

Trigger: "That cold outreach email we wrote before — can you adapt it for the property management industry?"

1. Search Content Library for the referenced piece
2. Load the original content from the page body
3. Generate an updated version
4. Present a summary of key changes + new draft
5. On confirmation: overwrite page body and reset status to Draft

---

## Feature 4 — Content Styling Guide

### What it is

A standalone Notion page (not a database) that defines how the brand writes. The agent reads this page before every content draft. The page ID is stored in `CONTENT_STYLING_GUIDE_PAGE_ID` in `~/.hermes/.env`.

### Conversational triggers

- "Set up our content style — let's go through it together"
- "Update our SEO keywords to include: AI agents, SME automation"
- "Change our LinkedIn post length to under 200 words"
- "Add a compliance rule: never mention competitor names"
- "What are our current AEO target questions?"

### Guided setup workflow

1. Load the current Content Styling Guide page (if it exists)
2. Walk through each section conversationally, one at a time
3. For each section, present the current value (if any) and ask what to update
4. After each answer, PATCH the relevant section in the Notion page
5. Confirm each update: `✅ Updated: [Section] — [Summary of change]`

### Direct update workflow

For targeted updates ("change our LinkedIn format to..."):
1. Load the relevant section of the Content Styling Guide
2. Apply the change to the Notion page
3. Confirm: `✅ [Section] updated`

### Page structure (recommended)

```
Content Styling Guide
├── 1. Brand Voice & Tone
│   ├── Overall style
│   ├── Preferred expressions
│   ├── Words / phrases to avoid
│   └── Target reader
├── 2. Post Format Preferences
│   ├── Preferred length
│   ├── Structure style (bullets / paragraphs / mixed)
│   ├── Opening hook style
│   ├── Closing style
│   ├── Emoji usage
│   └── Hashtag usage
├── 3. Platform-Specific Rules (table: Platform / Format Notes / Special Rules)
├── 4. SEO Keywords
│   ├── Primary keywords
│   ├── Secondary keywords
│   └── Keywords to avoid
├── 5. AEO — AI Engine Optimization
│   ├── Target questions (questions users ask AI assistants)
│   ├── Authority topics
│   └── Preferred answer format
└── 6. Special Requirements
    ├── Legal / compliance constraints
    ├── Market / regional notes
    └── Other custom rules
```

### Behavioral rules for this feature

- Always confirm before overwriting existing values — show the current value and ask if replacing
- If the styling guide doesn't exist yet, offer to create it from the standard template
- If a user requests content without a styling guide set up, generate the content but note: "Your Content Styling Guide isn't set up yet — I used general brand KB defaults. Would you like to set up the guide after this?"

---

## Behavioral Rules

1. **Always load KB + Content Styling Guide before writing** — brand positioning, target audience, and styling guide must all be loaded. No exceptions.
2. **No hallucinated product claims** — if the KB doesn't say it, don't write it
3. **Sources = inspiration, not copy-paste** — always rewrite in brand voice
4. **Show draft, get confirmation before saving** — unless user says "save it directly"
5. **Never auto-add sources** — always present candidates and wait for the user to choose
6. **Check for duplicates before saving** — search Content Library for similar existing pieces before creating a new one
7. **CTA required** — every piece needs a clear next step unless the format explicitly doesn't call for one

---

## Configuration

```
# Notion
NOTION_TOKEN=                          # Notion integration token

# Knowledge Base (Notion Page IDs)
KB_BRAND_POSITIONING_PAGE_ID=          # Brand identity / positioning page
KB_TARGET_AUDIENCE_PAGE_ID=            # Target audience analysis page
KB_PRICING_PAGE_ID=                    # Pricing and business model page
KB_PRODUCT_PAGE_ID=                    # Product / service overview page

# Notion Databases
SOURCES_DB=                            # Sources database ID
CONTENT_LIBRARY_DB=                    # Content Library database ID

# Content Styling Guide (standalone Notion page — not a database)
CONTENT_STYLING_GUIDE_PAGE_ID=         # Content Styling Guide page ID

# Telegram (for daily briefing delivery)
TELEGRAM_BOT_TOKEN=                    # Telegram bot token
CONTENT_TELEGRAM_CHAT_ID=             # e.g. -100XXXXXXXXX
CONTENT_TELEGRAM_THREAD_ID=           # Thread/topic ID

# Defaults
CONTENT_LANGUAGE=zh-TW                 # Default output language: zh-TW / en / etc.
```

> Field names in the client's databases may differ from the recommended schema described in SETUP.md. Always check the actual property names in each database before querying or writing. Use the field names as they exist in the client's Notion workspace.
