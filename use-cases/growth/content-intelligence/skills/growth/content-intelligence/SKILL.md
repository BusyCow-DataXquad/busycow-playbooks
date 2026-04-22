---
name: content-intelligence
description: >
  AI-powered content intelligence skill for Hermes Agent.
  Reads the company's Core Business KB, discovers relevant external sources,
  delivers daily topic suggestions alongside the CRM report, and generates
  on-brand content (Blog, Social Media, Email, WhatsApp) saved to Notion Content Library.
version: 2.0.0
author: BusyCow
tags: [Content Intelligence, Content Creation, Notion, Brand, Sources, Daily Report]
---

# Content Intelligence — AI Content Intelligence System

## Overview

Three features, one consistent loop: find the right sources → get daily topic ideas → write content.

1. **Source Discovery** — reads Core Business KB, recommends external sources worth tracking, saves to Sources DB after user approval
2. **Daily Topic Briefing** — every morning at 9am, scans Active Sources + KB and delivers 3–5 actionable topic suggestions
3. **Content Writing** — generates any format (Blog / Social / Email / WhatsApp) grounded in KB, saves to Content Library on confirm

The key rule: **every piece of content must be grounded in the company KB** — never write from scratch without referencing brand positioning and TA.

---

## Notion Databases Required

### Sources DB

Tracks external sources the agent monitors for content inspiration.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Source name (e.g. "Tech in Asia") |
| URL | URL | Homepage or RSS feed URL |
| Type | Select | Blog / News / Newsletter / Industry Report |
| Topics | Multi-select | Keywords this source covers |
| Language | Select | zh-TW / en / zh-HK / etc. |
| Status | Select | Active / Paused |
| Added By | Select | User / Agent Suggested |
| Last Fetched | Date | Auto-updated by agent |
| Notes | Rich Text | Why this source is relevant |

### Content Library

Stores all generated content drafts and published pieces. Content body is stored in **page blocks**, not table fields.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Content title |
| Type | Select | Email Template / WhatsApp/LINE Message / Blog Post / Success Story / Product Update / Event/Promotion |
| Stage Target | Multi-select | Lead / Qualified / Use Case Confirmed / Closing / All |
| Industry Target | Multi-select | Client-defined industries + All |
| Channel | Multi-select | Email / WhatsApp / LINE / LinkedIn / etc. |
| Status | Select | Draft / Ready / Archived |
| Tags | Multi-select | Flexible — match client's content taxonomy |
| Body | Page blocks | Full content — NEVER in table field |

---

## Feature 1 — Source Discovery

### When to trigger
- User asks: "Which external sources do you think we should be tracking?"
- User pastes a URL: "Add this source: https://..."
- Proactively, when Sources DB has fewer than 3 Active sources

### Workflow — Agent-Suggested Sources

1. Load Core Business KB (Brand Positioning + TA + Industry Focus)
2. Based on KB, identify relevant source types (industry news, competitor blogs, analyst research, etc.)
3. Web-search for 4–6 candidate sources that match the client's market and language
4. Present for review — **NEVER add sources automatically**:

```
📰 Based on your KB, I recommend tracking the following sources:

1. [Source Name] — [URL]
   Type: [Blog / News / Newsletter]
   Language: [zh-TW / en]
   Why it's relevant: [1 sentence tied to KB topic or TA]

2. [Source Name] — [URL]
   ...

Which ones would you like to add?
```

5. On confirm: create record in Sources DB with `Status: Active`, `Added By: Agent Suggested`

### Workflow — User-Added Sources

1. User pastes a URL
2. Fetch the URL, summarize what it covers
3. Confirm: "This source mainly covers [topics] — suggested language tag: [lang]. Would you like to add it?"
4. On confirm: create record with `Added By: User`

### Managing Sources

- "Pause tracking HBR" → `PATCH /pages/{id}` set `Status: Paused`
- "Remove this source" → archive the record (`archived: true`)

---

## Feature 2 — Daily Topic Briefing

### Cron Job Setup

This feature runs as part of the Daily Report cron (or standalone). Runs at 01:00 UTC = 09:00 Taiwan time.

```
Schedule: 0 1 * * *
Prompt (append to Daily CRM Report prompt, or run as separate cron):

---
## Content Intelligence Section

1. Query Sources DB (Status: Active), retrieve all source URLs
2. Run web_extract on each source to fetch the latest 3–5 article titles and summaries
3. Load Core Business KB (Brand Positioning + TA)
4. From all source articles, select the 3–5 topics most worth writing about
   Selection criteria: relevant to brand TA / can be leveraged to express brand perspective / current events a bonus
5. Output format:

📝 Today's Writing Topics (Content Intelligence)

1️⃣ [Topic Title]
Inspiration source: [Source Name]
Why it's worth writing: [1–2 sentences explaining relevance to brand/TA]
Recommended format: [Blog / LinkedIn / IG / Email / WhatsApp]
Recommended angle: [Specific hook, 1 sentence]

2️⃣ ...

3️⃣ ...

💡 Reply "Write #N" or "Write #N as [format]" to get started
---
```

### Acting on Briefing

After the briefing is delivered in Telegram, user can reply:
- "Write #1" → execute Feature 3 with the suggested topic + format
- "Write #2 as Email format" → override format, then execute Feature 3
- "Save #3 as a draft title — I'll write it next week" → create placeholder entry in Content Library (Status: Draft, empty body)

---

## Feature 3 — Content Writing

### Conversational Triggers

- "Write #N" (from briefing)
- "Write a cold outreach email for manufacturing industry clients"
- "Write a LinkedIn post about how AI helps sales teams save time"
- "Based on today's Tech in Asia, write a perspective piece from our angle"
- "Prepare this month's Lead outreach materials — one Email and one WhatsApp"

### Step 1 — Clarify (only if request is ambiguous)

```
Got it! Let me confirm a few details:

📌 Topic: [restate]
📄 Format: [Blog / LinkedIn / IG / Email / WhatsApp / ...]
🎯 Target audience: [from TA KB or user-specified]
🏷️ Stage: [Lead / Qualified / Closing / All]
🌐 Language: [zh-TW / en]

Does this look right?
```

If the request is clear enough (e.g. direct continuation from briefing), skip and go to Step 2.

### Step 2 — Load KB

Always load before writing:

| Content Type | KB Sections to Load |
|-------------|---------------------|
| Blog / Long-form | Brand Positioning + TA + Product Overview |
| Social Media | Brand Positioning + TA |
| Cold Email | Brand Positioning + TA + Product Overview |
| Follow-up | Brand Positioning + TA |
| Case Study | Product Overview + Pricing |

### Step 3 — Pull Source Material (if applicable)

If the user references a source, or if the topic came from the daily briefing:
1. Fetch the source URL (web extract)
2. Extract key points relevant to the topic
3. Use as supporting material — **always rewrite in brand voice, never copy-paste**

### Step 4 — Generate and Present

Rules:
- Match brand tone of voice from KB (no generic AI filler: "In today's fast-paced world...")
- Ground product/service claims in KB — no hallucinated features
- Include a clear CTA unless the format doesn't call for one
- Output in the specified language

Present the draft:

```
✍️ Content Draft

[TYPE] — [TOPIC]
Target audience: [audience] | Language: [lang] | Reference source: [source or "Internal KB"]

---

[GENERATED CONTENT]

---

Any adjustments needed? (tone / length / angle)
Confirm to save to Content Library.
```

### Step 5 — Save to Content Library

After user confirms (or says "save it directly"):

```json
{
  "parent": {"database_id": "CONTENT_LIBRARY_DB"},
  "properties": {
    "Name": {"title": [{"text": {"content": "[Type] — [Topic] — [YYYY-MM-DD]"}}]},
    "Type": {"select": {"name": "[Email Template / Blog Post / WhatsApp/LINE Message / ...]"}},
    "Stage Target": {"multi_select": [{"name": "[Lead / All / ...]"}]},
    "Industry Target": {"multi_select": [{"name": "[Industry or All]"}]},
    "Channel": {"multi_select": [{"name": "[Email / WhatsApp / LinkedIn / ...]"}]},
    "Status": {"select": {"name": "Draft"}}
  },
  "children": [
    {"object": "block", "type": "paragraph",
     "paragraph": {"rich_text": [{"type": "text", "text": {"content": "[FULL CONTENT]"}}]}}
  ]
}
```

Confirm: `✅ Saved to Content Library — [Title] (Status: Draft)`

### Batch Writing

**Trigger:** "Prepare this month's Lead outreach materials — one Email, one WhatsApp, and one LinkedIn post"

1. Clarify once for all pieces (topic/angle/audience)
2. Load KB + relevant sources once
3. Generate all pieces sequentially
4. Present all drafts together for review
5. Save all confirmed pieces in one batch
6. Confirm: `✅ 3 pieces saved to Content Library`

### Content Refresh

**Trigger:** "That cold outreach email we wrote before — can you adapt it for the property management industry?"

1. Search Content Library for the referenced piece
2. Load original content from page body
3. Generate updated version
4. Present diff: key changes summary + new draft
5. On confirm, overwrite page body and reset Status to Draft

---

## Behavioral Rules

1. **Always load KB before writing** — brand positioning and TA must be loaded first
2. **No hallucinated product claims** — if the KB doesn't mention it, don't make it up
3. **Sources = inspiration, not copy-paste** — always rewrite in brand voice
4. **Show draft, get confirm before saving** — unless user says "save it directly"
5. **Never auto-add sources** — always present candidates for user approval
6. **Check for duplicates** — before saving, check if a similar piece exists in Content Library
7. **CTA required** — every piece needs a clear next step (unless format doesn't call for it)

---

## Configuration

```
# Notion
NOTION_TOKEN=                        # Notion integration token

# Knowledge Base (Notion Page IDs)
KB_BRAND_POSITIONING_PAGE_ID=        # Brand identity doc
KB_TARGET_AUDIENCE_PAGE_ID=          # TA analysis doc
KB_PRICING_PAGE_ID=                  # Pricing & business model
KB_PRODUCT_PAGE_ID=                  # Product/service overview

# Notion Databases
SOURCES_DB=                          # Sources DB ID
CONTENT_LIBRARY_DB=                  # Content Library DB ID

# Telegram (for briefings)
CONTENT_TELEGRAM_CHAT_ID=            # e.g. -100XXXXXXXXX
CONTENT_TELEGRAM_THREAD_ID=          # Thread/topic ID

# Defaults
CONTENT_LANGUAGE=zh-TW               # Default output language
```
