---
name: content-creation
description: >
  AI-powered content creation skill for Hermes Agent.
  Reads the company's core business knowledge base (brand positioning,
  target audience, strategy, product info) and generates on-brand content
  based on user-specified topic and format. Saves output to a Notion
  Content Library automatically.
version: 1.0.0
author: BusyCow
tags: [Content Creation, Content Library, Notion, Brand, Marketing]
---

# Content Creation — AI 內容生成

## Overview

This skill turns Hermes into a brand-aware content writer. The agent:

1. Loads the company's **Knowledge Base** (who we are, what we do, our strategy)
2. Takes the user's **content request** (topic + format)
3. Generates **on-brand content** grounded in the KB
4. Saves the result to the **Content Library** in Notion with proper metadata

The key principle: every piece of content must be consistent with the company's
positioning, tone of voice, and target audience — not generic AI output.

---

## Client Setup Checklist

| Item | Description |
|------|-------------|
| `NOTION_TOKEN` | Notion integration API key |
| Knowledge Base source | Where core business docs live (see below) |
| Content Library DB ID | Notion database for storing generated content |
| Brand tone & voice | Formal / casual / technical / friendly |
| Default language | zh-TW / en / etc. |

---

## Knowledge Base

The Knowledge Base is the foundation of every content piece. It tells the agent
**who the company is** so the output stays on-brand.

### What to Include

| Document | Description |
|----------|-------------|
| Brand Positioning | Who we are, what we do, our unique value, tone of voice |
| Target Audience (TA) | Who we're talking to — pain points, goals, profile |
| Pricing & Business Model | How we make money, package tiers, key differentiators |
| Product / Service Overview | What we offer, key features, use cases |
| Case Studies / Success Stories | Real examples (optional but powerful) |

### Where to Store the KB

Choose one approach per client:

**Option A — Notion Pages (recommended)**
Store each document as a Notion page. At runtime, the agent fetches them via API.
Add page IDs to config:

```
KB_BRAND_POSITIONING_PAGE_ID=your_page_id
KB_TARGET_AUDIENCE_PAGE_ID=your_page_id
KB_PRICING_PAGE_ID=your_page_id
KB_PRODUCT_PAGE_ID=your_page_id
```

**Option B — Local Markdown Files**
Store as `.md` files in `~/.hermes/kb/`. The agent reads them at runtime.

```
~/.hermes/kb/brand-positioning.md
~/.hermes/kb/target-audience.md
~/.hermes/kb/pricing.md
~/.hermes/kb/product-overview.md
```

**Option C — Single Combined Document**
One Notion page or markdown file containing all KB sections.

```
KB_MASTER_PAGE_ID=your_page_id
```

---

## Content Types

Supported formats — adapt this list per client:

| Type | Description | Typical Length |
|------|-------------|----------------|
| Email Template | Cold outreach or nurturing email | 150–300 words |
| Follow-up Message | WhatsApp / LINE short follow-up | 50–100 words |
| Blog Post | Long-form thought leadership | 500–1200 words |
| LinkedIn Post | Professional social post | 100–300 words |
| Case Study | Problem → Solution → Result story | 300–600 words |
| Product Update | Announcement of new feature/service | 100–200 words |
| Event Promotion | Webinar, trade show, demo invite | 100–200 words |
| Phone Script | Call guide with key talking points | Bullet format |
| Proposal Summary | Executive summary for a proposal | 200–400 words |

---

## Conversational Trigger

The agent activates this skill when the user requests content creation.

**Example triggers:**
- 「幫我寫一封開發信給製造業的客戶」
- 「寫一篇 LinkedIn 文，主題是 AI 幫業務節省時間」
- 「幫我做一個 WhatsApp 跟進訊息，給還沒回覆的 Lead」
- 「寫一個 Case Study，客戶是物流業，導入了我們的排班自動化」

---

## Agent Workflow

### Step 1 — Clarify the Request

Before writing, confirm:

```
收到！幫你生成內容前，確認幾個細節：

📌 主題：[restate what user said]
📄 格式：[Email / WhatsApp / Blog / LinkedIn / ...]（如未指定，建議格式）
🎯 目標對象：[who will read this — from TA KB or user-specified]
🏷️ Pipeline Stage：[which stage is this for — Lead / Qualified / Closing / All]
🌐 語言：[zh-TW / en / 雙語]

這樣對嗎？有需要調整的嗎？
```

If the user's request is clear enough, skip directly to writing and present the draft.

### Step 2 — Load the Knowledge Base

Before generating, always load relevant KB documents:

```python
# Fetch from Notion (Option A)
# GET /v1/blocks/{KB_PAGE_ID}/children → extract text content

# Or read local file (Option B)
# read_file("~/.hermes/kb/brand-positioning.md")
```

Relevant KB sections to load based on content type:

| Content Type | Load These KB Sections |
|-------------|------------------------|
| Cold Email | Brand Positioning + TA + Product Overview |
| Follow-up | Brand Positioning + TA |
| Blog Post | Brand Positioning + TA + Product + Case Studies |
| Case Study | Product Overview + Pricing + Case Studies |
| LinkedIn | Brand Positioning + TA |
| Proposal Summary | All sections |

### Step 3 — Generate the Content

Write the content using KB as the grounding context. Rules:

- **Tone of voice**: match the brand's defined tone (formal/casual/etc.)
- **No generic AI filler**: avoid phrases like "In today's fast-paced world..." — use specific, grounded language
- **Personalization hooks**: if target is a specific stage or industry, tailor accordingly
- **Call to action**: always end with a clear CTA unless user says otherwise
- **Length**: follow the content type guidelines above

Present the draft clearly:

```
✍️ 內容草稿

[CONTENT TYPE] — [TOPIC]
目標對象：[audience]
語言：[language]

---

[GENERATED CONTENT]

---

需要調整嗎？（語氣 / 長度 / 角度 / 其他）
確認後我可以直接存入 Content Library。
```

### Step 4 — Save to Content Library

After user confirms (or if user says "直接存" / "save it"):

1. Create a new page in the Content Library Notion database
2. Set metadata fields:

```json
{
  "parent": {"database_id": "YOUR_CONTENT_LIBRARY_DB_ID"},
  "properties": {
    "Name": {"title": [{"text": {"content": "[Content Type] — [Topic] — [Date]"}}]},
    "Type": {"select": {"name": "[Email Template / WhatsApp Message / Blog Post / ...]"}},
    "Stage Target": {"multi_select": [{"name": "[Lead / Qualified / All / ...]"}]},
    "Industry Target": {"multi_select": [{"name": "[Industry or All]"}]},
    "Channel": {"multi_select": [{"name": "[Email / WhatsApp / LinkedIn / ...]"}]},
    "Status": {"select": {"name": "草稿"}}
  },
  "children": [
    {
      "object": "block",
      "type": "paragraph",
      "paragraph": {
        "rich_text": [{"type": "text", "text": {"content": "[FULL CONTENT HERE]"}}]
      }
    }
  ]
}
```

3. Confirm to the user:

```
✅ 已存入 Content Library

標題：[Name]
格式：[Type]
狀態：草稿

需要的時候輸入「找 [主題] 草稿」即可調出。
```

---

## Batch Content Generation

The user may request multiple pieces at once.

**Trigger:** 「幫我針對 Lead 階段的客戶準備這個月的開發素材，要有 Email、WhatsApp、LinkedIn 各一篇」

**Workflow:**
1. Confirm the brief once for all pieces (topic, audience, stage)
2. Generate all pieces sequentially
3. Present all drafts together for review
4. Save all confirmed pieces to Content Library in one go
5. Report: 「已存入 N 篇內容至 Content Library ✅」

---

## Content Refresh / Update

The user may want to update existing content in the library.

**Trigger:** 「之前那篇 Lead 開發信，幫我改成適合製造業的版本」

**Workflow:**
1. Search Content Library for the referenced piece
2. Load the original content from the page body
3. Generate an updated version with the requested changes
4. Present the new draft alongside key changes highlighted
5. On confirm, update the page body and reset Status to 草稿

---

## Quality Rules

The agent must follow these rules on every content piece:

1. **Always load KB first** — never write content without referencing the brand KB
2. **No hallucinated facts** — if the KB doesn't mention it, don't make it up
3. **Ask before saving** — always show the draft and get confirmation before writing to Notion (unless user explicitly says "直接存")
4. **Avoid duplicate saves** — search Content Library first if the user references an existing piece
5. **CTA is required** — every piece must have a clear next step for the reader, unless the format doesn't call for it (e.g. phone script)
6. **Language consistency** — if brand language is zh-TW, don't mix in casual English without reason

---

## Customization Guide

When deploying for a new client, configure:

| Setting | How to Set |
|---------|------------|
| KB source (Notion vs local files) | Set page IDs or file paths in `.env` |
| Content types available | Add/remove from the Type select in Content Library DB |
| Stage options | Match client's pipeline stages |
| Industry options | Match client's target industries |
| Tone of voice | Document in KB Brand Positioning section |
| Default language | Set `CONTENT_LANGUAGE` in `.env` |
| Content Library DB | Set `CONTENT_LIBRARY_DB` in `.env` |

---

## Configuration Block

```
# Client: [CLIENT NAME]
# Configured: [DATE]

# Notion
NOTION_TOKEN=                        # Notion integration token

# Knowledge Base (choose Option A or B)
# Option A — Notion Pages
KB_BRAND_POSITIONING_PAGE_ID=        # Page ID of brand positioning doc
KB_TARGET_AUDIENCE_PAGE_ID=          # Page ID of TA analysis doc
KB_PRICING_PAGE_ID=                  # Page ID of pricing/business model doc
KB_PRODUCT_PAGE_ID=                  # Page ID of product/service overview

# Option B — Local Files
# KB files stored in ~/.hermes/kb/

# Content Library
CONTENT_LIBRARY_DB=                  # Notion database ID for content storage

# Defaults
CONTENT_LANGUAGE=zh-TW               # Default output language
BRAND_TONE=                          # e.g. professional / friendly / technical
```
