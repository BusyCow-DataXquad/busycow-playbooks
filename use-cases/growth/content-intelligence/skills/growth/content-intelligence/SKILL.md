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

# Content Intelligence — AI 內容智慧系統

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
| Type | Select | Email 模板 / WhatsApp/LINE 訊息 / Blog 貼文 / Success Story / 產品更新 / 活動/促銷 |
| Stage Target | Multi-select | Lead / Qualified / Use Case Confirmed / Closing / All |
| Industry Target | Multi-select | Client-defined industries + All |
| Channel | Multi-select | Email / WhatsApp / LINE / LinkedIn / etc. |
| Status | Select | 草稿 / 可用 / 封存 |
| Tags | Multi-select | Flexible — match client's content taxonomy |
| Body | Page blocks | Full content — NEVER in table field |

---

## Feature 1 — Source Discovery

### When to trigger
- User asks: 「你覺得我們應該追蹤哪些外部來源？」
- User pastes a URL: 「加這個來源：https://...」
- Proactively, when Sources DB has fewer than 3 Active sources

### Workflow — Agent-Suggested Sources

1. Load Core Business KB (Brand Positioning + TA + Industry Focus)
2. Based on KB, identify relevant source types (industry news, competitor blogs, analyst research, etc.)
3. Web-search for 4–6 candidate sources that match the client's market and language
4. Present for review — **NEVER add sources automatically**:

```
📰 根據你們的 KB，我建議追蹤以下來源：

1. [Source Name] — [URL]
   類型：[Blog / News / Newsletter]
   語言：[zh-TW / en]
   為什麼相關：[1 sentence tied to KB topic or TA]

2. [Source Name] — [URL]
   ...

要加哪幾個進來？
```

5. On confirm: create record in Sources DB with `Status: Active`, `Added By: Agent Suggested`

### Workflow — User-Added Sources

1. User pastes a URL
2. Fetch the URL, summarize what it covers
3. Confirm: 「這個來源主要涵蓋 [topics]，建議語言標籤 [lang]，要加進來嗎？」
4. On confirm: create record with `Added By: User`

### Managing Sources

- 「暫停追蹤 HBR」→ `PATCH /pages/{id}` set `Status: Paused`
- 「移除這個來源」→ archive the record (`archived: true`)

---

## Feature 2 — Daily Topic Briefing

### Cron Job Setup

This feature runs as part of the Daily Report cron (or standalone). Runs at 01:00 UTC = 09:00 Taiwan time.

```
Schedule: 0 1 * * *
Prompt (append to Daily CRM Report prompt, or run as separate cron):

---
## Content Intelligence 部分

1. 查詢 Sources DB（Status: Active），取所有來源 URL
2. 對每個來源執行 web_extract，抓取最新 3–5 篇文章標題與摘要
3. 載入 Core Business KB（品牌定位 + TA）
4. 從所有來源文章中，挑出最值得寫成內容的 3–5 個主題
   篩選標準：與品牌 TA 相關 / 可借力表達品牌觀點 / 近期時事加分
5. 輸出格式：

📝 今日可寫主題（Content Intelligence）

1️⃣ [主題標題]
靈感來源：[Source Name]
為什麼值得寫：[1–2句，說明與品牌/TA的關聯]
建議格式：[Blog / LinkedIn / IG / Email / WhatsApp]
建議角度：[具體切入點，1句話]

2️⃣ ...

3️⃣ ...

💡 回覆「寫第 N 個」或「寫第 N 個，改成 [格式]」即可開始
---
```

### Acting on Briefing

After the briefing is delivered in Telegram, user can reply:
- 「寫第 1 個」→ execute Feature 3 with the suggested topic + format
- 「寫第 2 個，改成 Email 格式」→ override format, then execute Feature 3
- 「第 3 個先存草稿標題，下週再寫」→ create placeholder entry in Content Library (Status: 草稿, empty body)

---

## Feature 3 — Content Writing

### Conversational Triggers

- 「寫第 N 個」 (from briefing)
- 「幫我寫一封開發信給製造業的客戶」
- 「寫一篇 LinkedIn 文，主題是 AI 如何幫業務節省時間」
- 「根據今天的 Tech in Asia，幫我寫一篇我們的觀點」
- 「幫我準備這個月的 Lead 素材，Email + WhatsApp 各一篇」

### Step 1 — Clarify (only if request is ambiguous)

```
收到！確認幾個細節：

📌 主題：[restate]
📄 格式：[Blog / LinkedIn / IG / Email / WhatsApp / ...]
🎯 目標對象：[from TA KB or user-specified]
🏷️ Stage：[Lead / Qualified / Closing / All]
🌐 語言：[zh-TW / en]

這樣對嗎？
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
✍️ 內容草稿

[TYPE] — [TOPIC]
目標對象：[audience] | 語言：[lang] | 參考來源：[source or "內部 KB"]

---

[GENERATED CONTENT]

---

需要調整嗎？（語氣 / 長度 / 角度）
確認後存入 Content Library。
```

### Step 5 — Save to Content Library

After user confirms (or says「直接存」):

```json
{
  "parent": {"database_id": "CONTENT_LIBRARY_DB"},
  "properties": {
    "Name": {"title": [{"text": {"content": "[Type] — [Topic] — [YYYY-MM-DD]"}}]},
    "Type": {"select": {"name": "[Email 模板 / Blog 貼文 / WhatsApp/LINE 訊息 / ...]"}},
    "Stage Target": {"multi_select": [{"name": "[Lead / All / ...]"}]},
    "Industry Target": {"multi_select": [{"name": "[Industry or All]"}]},
    "Channel": {"multi_select": [{"name": "[Email / WhatsApp / LinkedIn / ...]"}]},
    "Status": {"select": {"name": "草稿"}}
  },
  "children": [
    {"object": "block", "type": "paragraph",
     "paragraph": {"rich_text": [{"type": "text", "text": {"content": "[FULL CONTENT]"}}]}}
  ]
}
```

Confirm: `✅ 已存入 Content Library — [Title]（狀態：草稿）`

### Batch Writing

**Trigger:** 「幫我準備這個月的 Lead 開發素材，Email + WhatsApp + LinkedIn 各一篇」

1. Clarify once for all pieces (topic/angle/audience)
2. Load KB + relevant sources once
3. Generate all pieces sequentially
4. Present all drafts together for review
5. Save all confirmed pieces in one batch
6. Confirm: `✅ 已存入 3 篇內容至 Content Library`

### Content Refresh

**Trigger:** 「之前那封開發信，幫我改成適合物業管理業的版本」

1. Search Content Library for the referenced piece
2. Load original content from page body
3. Generate updated version
4. Present diff: 主要修改點 + new draft
5. On confirm, overwrite page body and reset Status to 草稿

---

## Behavioral Rules

1. **Always load KB before writing** — brand positioning and TA must be loaded first
2. **No hallucinated product claims** — if the KB doesn't mention it, don't make it up
3. **Sources = inspiration, not copy-paste** — always rewrite in brand voice
4. **Show draft, get confirm before saving** — unless user says「直接存」
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
