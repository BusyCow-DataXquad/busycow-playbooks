# Content Intelligence — Setup Instructions

> You are setting up the Content Intelligence use case for a client. Run through all four phases in order. Do not skip ahead. Report findings to the user before making any changes.

---

## What needs to be set up

This use case requires:
- Two Notion databases: Sources DB and Content Library
- Four Knowledge Base pages: Brand Positioning, Target Audience, Pricing, and Product Overview
- One standalone Notion page: Content Styling Guide

Some of these may already exist. Your job is to find what exists, adapt it where possible, and only create what is genuinely missing.

---

## Phase 1 — Discovery

Search the client's Notion workspace for each required item.

### Search keywords to use

Use the Notion API `POST /search` endpoint with the following queries:

| Target | Type | Search terms to try |
|--------|------|---------------------|
| Sources DB | database | "Sources", "Content Sources", "Media", "Feeds", "References" |
| Content Library | database | "Content Library", "Content", "Posts", "Drafts", "Templates" |
| Brand Positioning | page | "Brand Identity", "Brand Positioning", "About Us", "Company Profile", "Brand Voice" |
| Target Audience | page | "Target Audience", "TA", "ICP", "Ideal Customer", "Customer Profile" |
| Pricing | page | "Pricing", "Price List", "Packages", "Plans" |
| Product Overview | page | "Product", "Service", "Solution Overview", "What We Offer" |
| Content Styling Guide | page | "Content Styling Guide", "Style Guide", "Content Guide", "Writing Guide", "Brand Guide" |

### How to identify a match

- **Sources DB**: a database with a title field for source names, plus URL and status fields
- **Content Library**: a database with a title field for content names, plus a type/format field and a status field
- **KB pages**: Notion pages (not databases) that contain written descriptions of brand, audience, pricing, or product
- **Content Styling Guide**: a Notion page (not a database) containing writing style rules, tone guidelines, or platform rules

### Report before proceeding

After searching, report to the user:

- Which databases were found (name, Notion ID, brief description)
- Which KB pages were found (name, Notion ID)
- Whether the Content Styling Guide page was found
- Anything ambiguous or uncertain

Ask the user to confirm each match before proceeding to Phase 2.

---

## Phase 2 — Onboard

For each item that was not found (or rejected by the user), follow the steps below.

### For missing databases (Sources DB or Content Library)

1. Present the recommended schema (see Recommended Schema section below)
2. Ask: "Should I create this database via the Notion API, or would you prefer to create it manually and share it with me?"
3. If the user says to create via API: use `POST /databases` with the recommended schema
4. After creation, tell the user: "Please open Notion, find the new database, and share it with your Hermes integration — otherwise I won't be able to read or write to it."
5. Wait for the user to confirm before continuing

### For missing KB pages

Do not create KB pages automatically. Instead, tell the user:

"I couldn't find a [Brand Positioning / Target Audience / Pricing / Product Overview] page. You can create it in Notion and share it with me, or point me to the right existing page. This page doesn't need to follow a specific format — I just need to be able to read it."

Wait for the user to share the page ID or URL before moving on.

### For missing Content Styling Guide

The Content Styling Guide is a standalone Notion page — not a database. If it was not found:

1. Tell the user: "I couldn't find a Content Styling Guide page. This is where your brand's writing rules, post formats, SEO keywords, and platform-specific preferences are stored. I load it automatically before every content draft."

2. Ask: "Would you like me to create it from a standard template, or would you prefer to create it manually in Notion?"

3. If the user says to create via API: use `POST /pages` to create the page with the following 6-section template structure:

```
Content Styling Guide

1. Brand Voice & Tone
   - Overall style: (to be filled)
   - Preferred expressions: (to be filled)
   - Words / phrases to avoid: (to be filled)
   - Target reader description: (to be filled)

2. Post Format Preferences
   - Preferred length: (to be filled)
   - Structure style (bullets / paragraphs / mixed): (to be filled)
   - Opening hook style: (to be filled)
   - Closing style: (to be filled)
   - Emoji usage: (to be filled)
   - Hashtag usage: (to be filled)

3. Platform-Specific Rules
   [table: Platform | Format Notes | Special Rules]

4. SEO Keywords
   - Primary keywords: (to be filled)
   - Secondary keywords: (to be filled)
   - Keywords to avoid: (to be filled)

5. AEO — AI Engine Optimization
   - Target questions (questions users ask AI assistants): (to be filled)
   - Authority topics: (to be filled)
   - Preferred answer format: (to be filled)

6. Special Requirements
   - Legal / compliance constraints: (to be filled)
   - Market / regional notes: (to be filled)
   - Other custom rules: (to be filled)
```

4. After creation, tell the user: "Content Styling Guide created. Please share it with your Hermes integration in Notion. You can fill it in now or through conversation later — just tell me 'Let's set up our content style'."

---

## Phase 3 — Adapt

For each database that was found and confirmed:

1. Fetch the full schema using `GET /databases/{id}`
2. Compare properties to the recommended schema fields for that database
3. Identify fields that are in the recommended schema but missing from the client's database
4. Present the gaps to the user:

```
I found your [Database Name] database. Here are the recommended fields that are missing:

- [Field name] ([type]) — [what it's used for]
- [Field name] ([type]) — [what it's used for]

Would you like me to add these fields? I will not change or remove any of your existing fields.
```

5. If the user agrees: use `PATCH /databases/{id}` to add only the missing fields
6. Do NOT rename, delete, or modify any existing fields
7. Confirm: "Added [field name] to [database name]."

For KB pages and the Content Styling Guide: do not modify their structure. Just confirm they are accessible.

---

## Phase 4 — Configure

Once all items are found or created:

1. Use the memory tool to save all found/created IDs under a clearly labeled memory entry. Save the following structure:

```
Content Intelligence config:
  sources_db: <id>
  content_library_db: <id>
  kb_brand_page: <id>
  kb_ta_page: <id>
  kb_pricing_page: <id>
  kb_product_page: <id>
  content_styling_guide_page: <id>
  briefing_chat_id: <id>
  briefing_thread_id: <id>
  content_language: <language>
```

2. If any Telegram credentials are not yet in `~/.hermes/.env`, ask the user to provide them now
3. Ask the user to confirm the default content language if not already provided
4. The agent will read all IDs and delivery config from memory whenever this use case is active
5. Confirm setup is complete with a summary:

```
Content Intelligence setup complete.

Databases configured:
- Sources DB: [ID]
- Content Library: [ID]

Knowledge Base pages:
- Brand Positioning: [ID or "not configured"]
- Target Audience: [ID or "not configured"]
- Pricing: [ID or "not configured"]
- Product Overview: [ID or "not configured"]

Content Styling Guide: [ID or "not configured"]

Daily briefing: [chat ID] / thread [thread ID]
Default language: [content_language value]

Telegram: [configured / not configured]

All config saved to agent memory. You are ready to start creating content.
```

---

## Recommended Schema

> For reference — adapt to what the client has. These are suggestions, not requirements. If the client already has fields that serve the same purpose under different names, use those instead.

### Sources DB

Tracks external sources the agent monitors for content inspiration.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Source name (e.g. "Tech in Asia") |
| URL | URL | Homepage or RSS feed URL |
| Type | Select | Blog / News / Newsletter / Industry Report |
| Topics | Multi-select | Content themes this source covers |
| Language | Select | en / zh-TW / zh-HK / etc. |
| Status | Select | Active / Paused |
| Added By | Select | User / Agent Suggested |
| Last Fetched | Date | Auto-updated by agent after each scan |
| Notes | Rich Text | Why this source is relevant |

### Content Library

Stores all generated content drafts and published pieces. Content body is stored in Notion page blocks, not in table fields.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Content title |
| Type | Select | Email Template / WhatsApp/LINE Message / Blog Post / Social Post / Event/Promotion |
| Stage Target | Multi-select | Lead / Qualified / Closing / All |
| Industry Target | Multi-select | Client-defined industries + All |
| Channel | Multi-select | Email / WhatsApp / LINE / LinkedIn / Instagram / etc. |
| Status | Select | Draft / Ready / Archived |
| Tags | Multi-select | Flexible — match client's content taxonomy |
| Source | Relation | Links to the Sources DB entry that inspired this piece |
| Publish Date | Date | When this piece was published or is scheduled to go live |
| Keywords | Rich Text | SEO keywords embedded in this piece (long-form only) |

> Note: The Source relation field links back to the Sources DB so the agent can trace which source article inspired each content piece.
