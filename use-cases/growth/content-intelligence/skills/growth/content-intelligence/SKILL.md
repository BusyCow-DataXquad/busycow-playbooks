---
name: content-intelligence
description: >
  AI-powered content intelligence skill for Hermes Agent.
  Combines company knowledge base, external source monitoring, and
  automated content suggestions to generate on-brand content and
  save to a Notion Content Library. Supports daily/weekly briefings
  with proactive topic recommendations.
version: 1.0.0
author: BusyCow
tags: [Content Creation, Content Intelligence, Notion, Brand, Marketing, Sources]
---

# Content Intelligence — AI 內容智慧系統

## Overview

This skill turns Hermes into a proactive, brand-aware content strategist. The agent:

1. Loads the company's **Knowledge Base** (brand positioning, TA, strategy, product)
2. Monitors **external sources** (blogs, news, newsletters) for fresh industry content
3. Proactively **suggests content topics** via daily or weekly briefing to Telegram
4. **Generates on-brand content** grounded in KB + source material on request
5. Saves all content to the **Notion Content Library** with proper metadata

The key principle: every piece of content must be consistent with the company's
positioning and tone — not generic AI output. Sources provide inspiration and
credibility; the KB ensures brand alignment.

---

## Client Setup Checklist

| Item | Description |
|------|-------------|
| `NOTION_TOKEN` | Notion integration API key |
| Knowledge Base | Core business docs (Notion pages or local `.md` files) |
| Sources DB ID | Notion database for tracking external sources |
| Content Library DB ID | Notion database for storing generated content |
| Telegram Chat + Thread | Where to deliver daily/weekly briefings |
| Brand tone & voice | Defined in KB Brand Positioning doc |
| Default language | zh-TW / en / etc. |

---

## Notion Databases Required

### 1. Knowledge Base (KB)

Stored as Notion pages or local markdown files. See `knowledge-base-template/` for structure.

| Document | Description |
|----------|-------------|
| Brand Positioning | Who we are, what we do, tone of voice |
| Target Audience | Who we're talking to, pain points, goals |
| Pricing & Business Model | Packages, pricing, key differentiators |
| Product / Service Overview | Features, use cases, what problems we solve |

### 2. Sources Table

Tracks external sources the agent monitors for content inspiration.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Source name (e.g. "HBR", "Tech in Asia") |
| URL | URL | Homepage or RSS feed URL |
| Type | Select | Blog / News / Newsletter / Competitor / Industry Report |
| Topics | Multi-select | Keywords this source covers |
| Language | Select | zh-TW / en / etc. |
| Status | Select | Active / Paused |
| Added By | Select | User / Agent Suggested |
| Last Fetched | Date | Auto-updated by agent |
| Notes | Rich Text | Why this source is relevant |

### 3. Content Library

Stores all generated content drafts and published pieces.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Content title |
| Type | Select | Email / WhatsApp Message / Blog Post / LinkedIn / Case Study / Phone Script / Other |
| Stage Target | Multi-select | Lead / Qualified / Closing / All |
| Industry Target | Multi-select | Client-defined industries + All |
| Channel | Multi-select | Email / WhatsApp / LINE / LinkedIn / etc. |
| Status | Select | Draft / Ready / Archived |
| Source | Relation → Sources | Which source(s) inspired this piece |
| Created | Created Time | Auto |
| Body | Page content (blocks) | Full content — stored in page body, NOT table field |

---

## Feature 1 — On-Demand Content Generation

### Conversational Triggers

- 「幫我寫一封開發信給製造業的客戶」
- 「寫一篇 LinkedIn 文，主題是 AI 如何幫業務節省時間」
- 「幫我做一個 WhatsApp 跟進訊息，給還沒回覆的 Lead」
- 「根據最新的 [source name] 文章，幫我寫一篇我們的觀點」

### Workflow

**Step 1 — Clarify the Request**

Before writing, confirm:
```
收到！確認幾個細節：

📌 主題：[restate]
📄 格式：[Email / WhatsApp / Blog / LinkedIn / ...]
🎯 目標對象：[from TA KB or user-specified]
🏷️ Pipeline Stage：[Lead / Qualified / Closing / All]
📰 參考來源：[specific source, or "agent picks best available"]
🌐 語言：[zh-TW / en]

這樣對嗎？
```

If the request is clear enough, skip confirmation and present the draft directly.

**Step 2 — Load Knowledge Base**

Always load relevant KB sections before writing:

| Content Type | Load These KB Sections |
|-------------|------------------------|
| Cold Email | Brand Positioning + TA + Product Overview |
| Follow-up | Brand Positioning + TA |
| Blog Post | Brand Positioning + TA + Product + Case Studies |
| Case Study | Product Overview + Pricing |
| LinkedIn | Brand Positioning + TA |
| Proposal | All sections |

**Step 3 — Pull Relevant Sources (if applicable)**

If the user references a source, or if relevant sources exist in the Sources Table:
1. Fetch the source URL content (web extract)
2. Summarize key points relevant to the topic
3. Use as supporting material — cite or paraphrase, don't copy

**Step 4 — Generate Content**

Rules:
- Match the brand's defined tone of voice
- No generic AI filler ("In today's fast-paced world...")
- Use specific, grounded language from KB
- Personalize to the target stage / industry
- Always include a clear CTA (unless format doesn't require it)

Present the draft:
```
✍️ 內容草稿

[TYPE] — [TOPIC]
目標對象：[audience] | 語言：[lang]
參考來源：[source name or "內部 KB"]

---

[GENERATED CONTENT]

---

需要調整嗎？（語氣 / 長度 / 角度）
確認後直接存入 Content Library。
```

**Step 5 — Save to Content Library**

After user confirms (or if user says "直接存" / "save it"):

```json
{
  "parent": {"database_id": "YOUR_CONTENT_LIBRARY_DB_ID"},
  "properties": {
    "Name": {"title": [{"text": {"content": "[Type] — [Topic] — [Date]"}}]},
    "Type": {"select": {"name": "[Email Template / Blog Post / ...]"}},
    "Stage Target": {"multi_select": [{"name": "[Lead / All / ...]"}]},
    "Industry Target": {"multi_select": [{"name": "[Industry or All]"}]},
    "Channel": {"multi_select": [{"name": "[Email / WhatsApp / ...]"}]},
    "Status": {"select": {"name": "草稿"}}
  },
  "children": [{"object": "block", "type": "paragraph",
    "paragraph": {"rich_text": [{"type": "text", "text": {"content": "[FULL CONTENT]"}}]}}]
}
```

Confirm: `✅ 已存入 Content Library — [Title]（狀態：草稿）`

---

## Feature 2 — Source Management

### Adding Sources (User-initiated)

User can add sources conversationally:
- 「幫我追蹤 Tech in Asia 的文章」
- 「加一個新聞來源：https://example.com」

Agent:
1. Fetches the URL to confirm it's accessible and relevant
2. Shows a summary: 「這個來源主要涵蓋 [topics]，建議標籤：[tags]。要加進來嗎？」
3. On confirm, creates a record in the Sources Table

### Agent-Suggested Sources

The agent can proactively discover new relevant sources based on the client's KB topics and industry tags.

**Rules:**
- Agent NEVER adds sources automatically
- Agent presents suggestions for user review:

```
📰 發現 3 個可能有用的來源：

1. [Source Name] — [URL]
   涵蓋主題：[topics]
   相關原因：[why it matches the KB]

2. [Source Name] — [URL]
   ...

要加哪幾個進去？
```

- Only add after explicit user confirmation

### Removing / Pausing Sources

- 「暫停追蹤 HBR」→ update Status to Paused
- 「移除這個來源」→ archive the record

---

## Feature 3 — Daily / Weekly Content Briefing

### Setup

Configure a cron job to deliver briefings to Telegram:

```python
cronjob(
    action="create",
    name="Content Intelligence 週報",
    schedule="0 1 * * 1",  # Every Monday 9am Taiwan time
    deliver="telegram:-100XXXXXXXXX:THREAD_ID",
    prompt="""
你是 Content Intelligence 助理。執行以下流程並用繁體中文純文字回覆：

1. 讀取 Sources Table（Status: Active）中的所有來源
2. 抓取每個來源最新內容（web extract）
3. 結合 KB（品牌定位 + TA），篩選出最有價值的 3–5 個主題
4. 生成本週撰文建議報告

格式：
📢 Content Intelligence 週報 — {本週日期}

🔥 本週推薦主題

1️⃣ [主題標題]
靈感來源：[Source Name]
為什麼值得寫：[1–2句說明與品牌/TA的關聯]
建議格式：[Blog / LinkedIn / Email / WhatsApp]
建議角度：[具體切入點]

2️⃣ ...

3️⃣ ...

💡 快速指令
「寫第 1 個」→ 直接生成該主題內容
「寫第 2 個，改成 Email 格式」→ 調整格式後生成
"""
)
```

### Briefing Frequency Options

| Schedule | Cron | Best For |
|----------|------|----------|
| Daily | `0 1 * * *` | High-volume content teams |
| Weekly (Monday) | `0 1 * * 1` | Most SMEs — recommended |
| Bi-weekly | `0 1 * * 1/2` | Low-frequency publishing |

### Acting on Briefing Suggestions

After the briefing is delivered, the user can respond directly in Telegram:
- 「寫第 1 個」→ agent generates full content
- 「寫第 2 個，改成 WhatsApp 格式」→ generates with format override
- 「第 3 個先存草稿，下週再寫」→ saves a placeholder to Content Library

---

## Batch Content Generation

For preparing a full content calendar at once:

**Trigger:** 「幫我準備這個月的 Lead 開發素材，Email + WhatsApp + LinkedIn 各一篇」

**Workflow:**
1. Confirm the brief once for all pieces
2. Pull relevant KB + sources
3. Generate all pieces sequentially
4. Present all drafts together for review
5. Save all confirmed pieces in one batch
6. Report: `✅ 已存入 N 篇內容至 Content Library`

---

## Content Refresh

**Trigger:** 「之前那篇開發信，幫我改成適合物流業的版本」

**Workflow:**
1. Search Content Library for the referenced piece
2. Load original content from page body
3. Generate updated version with requested changes
4. Present new draft alongside key changes
5. On confirm, update page body and reset Status to 草稿

---

## Behavioral Rules

1. **Always load KB first** — never write without referencing brand KB
2. **No hallucinated facts** — if the KB doesn't mention it, don't make it up
3. **Sources are inspiration, not copy-paste** — always rewrite in brand voice
4. **Ask before saving** — always show draft and get confirmation before writing to Notion (unless user says "直接存")
5. **Never auto-add sources** — always present suggestions for user approval first
6. **Avoid duplicate saves** — search Content Library before creating a new entry if user references an existing piece
7. **CTA is required** — every piece must have a clear next step unless the format doesn't call for it

---

## Customization Guide

| Setting | How to Configure |
|---------|-----------------|
| KB source location | Notion page IDs or `~/.hermes/kb/` local files |
| Briefing frequency | Daily / Weekly / Bi-weekly cron schedule |
| Content types available | Add/remove from Content Library Type select field |
| Pipeline stages | Match client's Accounts stage options |
| Target industries | Match client's industry focus |
| Tone of voice | Document in KB Brand Positioning section |
| Default language | `CONTENT_LANGUAGE` in `.env` |

---

## Configuration Block

```
# Client: [CLIENT NAME]
# Configured: [DATE]

# Notion
NOTION_TOKEN=                        # Notion integration token

# Knowledge Base — Option A: Notion Pages
KB_BRAND_POSITIONING_PAGE_ID=        # Page ID of brand positioning doc
KB_TARGET_AUDIENCE_PAGE_ID=          # Page ID of TA analysis doc
KB_PRICING_PAGE_ID=                  # Page ID of pricing/business model doc
KB_PRODUCT_PAGE_ID=                  # Page ID of product/service overview

# Knowledge Base — Option B: Local Files (~/.hermes/kb/)
# No env vars needed — agent reads from ~/.hermes/kb/ directly

# Notion Databases
SOURCES_DB=                          # Notion database ID for Sources Table
CONTENT_LIBRARY_DB=                  # Notion database ID for Content Library

# Telegram (for briefings)
CONTENT_TELEGRAM_CHAT_ID=            # e.g. -100XXXXXXXXX
CONTENT_TELEGRAM_THREAD_ID=          # Thread/topic ID

# Defaults
CONTENT_LANGUAGE=zh-TW               # Default output language
BRAND_TONE=professional              # professional / friendly / technical / casual
BRIEFING_FREQUENCY=weekly            # daily / weekly / biweekly
STALE_SOURCE_DAYS=7                  # Days before a source is considered stale
```
