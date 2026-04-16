---
name: notion-crm-lead-nurturing
description: Set up a lead nurturing CRM in Notion with Accounts, Contacts, Activities, Partner List, and Content Library. Includes daily cron job for follow-up reminders via Telegram.
version: 1.0.0
metadata:
  hermes:
    tags: [Notion, CRM, Lead Nurturing, Telegram, Cron]
---

# Notion CRM Lead Nurturing Setup

Use this skill to build or replicate a lead nurturing CRM in Notion for a client. Covers schema design, table relations, rollups, and daily cron reporting.

## Overview

Five tables:
1. **Accounts** — companies (customers or prospects)
2. **Contacts** — people, linked to Accounts and/or Partners
3. **Activities** — interaction log, linked to Accounts + Contacts
4. **Partner List** — resellers/partners (separate from Accounts)
5. **Content Library** — message templates, case studies, blog posts for nurturing

## Step 1: Gather Info from Client

Before building, ask:
1. What are your Pipeline Stages? (default: Lead → Qualified → Use Case Confirmed → Closing → Deployed → Live ✅ / Lost ❌)
2. What industries do you serve?
3. What channels do you use? (Email / WhatsApp / LINE / LinkedIn)
4. What activity types do you log? (default: 電話 / 線上會議 / 實體拜訪 / Demo / 報價/提案 / Email / WhatsApp/LINE / 合約簽署)
5. Telegram group topic for daily reports? (need chat_id + thread_id)
6. Follow-up cadence — how many days without activity triggers a warning? (default: 3)

## Step 2: Find or Create Tables

Search the Notion workspace first:
```bash
curl -s -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"query": ""}' | jq '[.results[] | {id, type: .object, title: (.title[0].plain_text // "?")}]'
```

Note: search returns `data_source_id` (the `id` field). To get `database_id`, fetch: `GET /v1/data_sources/{id}` and read `parent.database_id`.

## Step 3: Accounts Schema

Required properties:
- Name (title — auto-created)
- Stage (status)
- Country (select)
- Industry (select)
- Company Size (select: 1–10人 / 11–50人 / 51–200人 / 200人以上)
- Source (select: 自開發 / Reseller介紹 / 展會/活動 / 口碑介紹 / 名單開發工具 / 網路廣告)
- Est. Value (number)
- Owner (people)
- Next Follow-up (date)
- Website (url)
- Notes (rich_text)
- Partner (relation → Partner List, dual_property)

Stage options: Lead / Qualified / Use Case Confirmed / Closing / Deployed / Live ✅ / Lost ❌

## Step 4: Contacts Schema

- Name (title)
- Job Title (rich_text)
- Company (relation → Accounts, dual_property)
- Partner (relation → Partner List, dual_property)
- Decision Role (select: 決策者 / 影響者 / 執行者)
- Email (email)
- Phone / WhatsApp (phone_number)
- WhatsApp (phone_number)
- LINE ID (rich_text)
- LinkedIn (url)
- Preferred Channel (select: WhatsApp / Email / 電話 / LINE / LinkedIn)
- Interested Features (multi_select)
- Notes (rich_text)

## Step 5: Activities Schema

- Summary (title)
- Date (date)
- Type (select: 電話 / 線上會議 / 實體拜訪 / Demo / 報價/提案 / Email / WhatsApp/LINE / 合約簽署)
- Account (relation → Accounts, **dual_property** — enables rollups)
- Contact (relation → Contacts, **dual_property**)
- Partner (relation → Partner List, dual_property)
- Owner (people)
- Client Response (rich_text)
- Next Action (rich_text)
- Stage Advanced? (checkbox)

**Critical:** Account and Contact relations MUST be dual_property so Accounts and Contacts get back-references. Convert with PATCH if they were created as single_property.

## Step 6: Add Rollups to Accounts

After converting Activities.Account to dual_property, the synced property name will be something like `"Related to Activities (Account)"`. Use that exact name:

```json
{
  "Last Activity Date": {
    "rollup": {
      "relation_property_name": "Related to Activities (Account)",
      "rollup_property_name": "Date",
      "function": "latest_date"
    }
  },
  "Activity Count": {
    "rollup": {
      "relation_property_name": "Related to Activities (Account)",
      "rollup_property_name": "Date",
      "function": "count"
    }
  }
}
```

## Step 7: Content Library Schema

Create via `POST /v1/databases` with `parent.type: "page_id"`. Only "Name" (title) is auto-created; add others via PATCH:

- Type (select: Email模板 / WhatsApp/LINE訊息 / Blog貼文 / Success Story / 產品更新 / 活動/促銷)
- Stage Target (multi_select: Lead / Qualified / Use Case Confirmed / Closing / Deployed / All)
- Industry Target (multi_select: per client industries + All)
- Channel (multi_select: Email / WhatsApp / LINE / LinkedIn / 電話腳本)
- Status (select: 草稿 / 可用 / 封存)
- Tags (multi_select)

**Content goes in page body blocks, NOT in a table field.** When creating a Content Library entry, add content as `children` blocks on the page.

## Step 8: Daily Cron Job

```python
cronjob(
    action="create",
    name="CRM 每日彙報",
    schedule="0 1 * * *",  # 9am Taiwan time
    deliver="telegram:-100XXXXXXXXX:THREAD_ID",
    prompt="""
你是 CRM 每日彙報助理。讀取 Notion CRM，產出今天業務彙報（繁體中文，純文字）。

Notion API Key: {KEY}
Accounts data_source_id: {ID}
Activities data_source_id: {ID}

查詢用 POST https://api.notion.com/v1/data_sources/{id}/query

格式：
📋 BusyCow CRM 日報 — {今天日期}

🔴 今日需跟進（Next Follow-up 到期）
⚠️ 停滯警示（超過 {N} 天沒有 Activity）
📊 Pipeline 現況（各 Stage 人數）
💡 今日建議（1-2 個具體建議）

無資料時寫「暫無」。
"""
)
```

## Enhanced Daily Cron: Auto-Draft Follow-ups

The cron job can do more than just report — it can pre-write follow-up drafts and save them to Content Library before sending the report:

1. Read all Accounts + Activities
2. Find accounts needing follow-up (Next Follow-up overdue OR no Activity in N days)
3. For each, generate a WhatsApp follow-up draft (100-150 words, conversational, based on Stage + past Activity summaries)
4. Save draft to Content Library: Name = "跟進草稿｜{Account}｜{date}", Type = WhatsApp/LINE訊息, Status = 草稿
5. In the daily report, list each follow-up target with: "已為你準備跟進草稿，輸入「看 {客戶名稱} 草稿」即可查看"

## Email Integration (Himalaya)

To enable email sending from the CRM context, set up Himalaya:

```bash
# Install
curl -sSL https://raw.githubusercontent.com/pimalaya/himalaya/master/install.sh | PREFIX=~/.local sh

# Config at ~/.config/himalaya/config.toml
[accounts.busycow]
email = "busycow@dataxquad.com"
display-name = "BusyCow"
default = true

backend.type = "imap"
backend.host = "imap.gmail.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "busycow@dataxquad.com"
backend.auth.type = "password"
backend.auth.raw = "APP_PASSWORD_HERE"  # Gmail App Password (16 chars, no spaces)

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.gmail.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "busycow@dataxquad.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.raw = "APP_PASSWORD_HERE"
```

Send email via pipe:
```bash
cat << 'EOF' | ~/.local/bin/himalaya template send
From: busycow@dataxquad.com
To: recipient@example.com
Subject: Subject here

Body here.
EOF
```

## Conversational CRM Behavior Rules

When operating as a CRM assistant in chat:
1. **Missing fields** — when creating Account or Activity, always ask for missing fields before saving
2. **Next step** — after every follow-up activity is mentioned, always ask "那下一步要做什麼？" and record Next Action + Next Follow-up date in Notion
3. **Draft first** — when suggesting outreach, write a draft and ask for approval before sending
4. **Email confirmation** — before sending any email, always show: recipient list (name + email), subject, and content summary. Only send after user explicitly confirms. Prevents duplicate sends.

## Account Page Template (4 Parts)

Apply via PATCH /v1/blocks/{page_id}/children after creating each new Account page. All fields default to「（待填）」— the template doubles as a data collection checklist for the follow-up team.

- **Part 1 — 客戶資料**: 客戶概況 table (公司名稱/產業/市場角色/地點/團隊規模/年營業額/利潤率/接洽人), 團隊組成 table (角色/人數/說明), 現有系統 (ERP/作業工具/溝通工具)
- **Part 2 — Use Case 規劃**: Use Case 1 + Use Case 2, each with 現況流程/BusyCow解法/核心價值
- **Part 3 — ROI 試算**: 人力成本明細 (月薪/每月總成本/年度成本) + 回本試算 (方案A初次部署/方案B初次+續約/第二年起)
- **Part 4 — 潛在升級應用**: numbered list, blank

## Content Library: Mark as Sent + Log Activity

When Hunter confirms a draft should be sent:
1. Update Content Library page status to 已發送:
   ```bash
   PATCH /v1/pages/{page_id}  →  {"properties": {"Status": {"select": {"name": "已發送"}}}}
   ```
2. Create an Activity record (use `database_id`, not `data_source_id`):
   ```python
   payload = {
       "parent": {"database_id": "74091aa3-ed1e-4110-a515-7649a41a3ec9"},
       "properties": {
           "Summary": {"title": [{"text": {"content": "跟進訊息發送｜{Account}｜{date}"}}]},
           "Type": {"select": {"name": "訊息"}},
           "Date": {"date": {"start": "YYYY-MM-DD"}},
           "Account": {"relation": [{"id": "{account_page_id}"}]},
           "Next Action": {"rich_text": [{"text": {"content": "等待對方回覆"}}]}
       }
   }
   ```
   Note: Activities title field is **Summary** (not Name). No "Notes" field — use "Client Response" or "Next Action" for text.

3. Remind Hunter to manually copy-paste the message — Hermes cannot send WhatsApp/LINE directly.

## Pitfalls

1. **Two IDs per database** — `data_source_id` for query/PATCH schema, `database_id` for creating pages. Get `database_id` from `data_source.parent.database_id`.

2. **PATCH /databases/{id} → 404** — always use `PATCH /v1/data_sources/{id}` for schema changes.

3. **Creating DB** — use `POST /v1/databases` (not /data_sources), must include `"parent": {"type": "page_id", "page_id": "..."}` with explicit `type` field. Omitting `type` causes validation error.

4. **Title property** — auto-created as "Name", cannot add another title property. Don't include `"Title": {"title": {}}` in schema PATCH.

5. **Relation in schema** — use `"data_source_id"` inside relation definition, not `"database_id"`.

6. **Block annotations** — put in `rich_text[].annotations`, not inside `rich_text[].text`.

7. **Rollups need dual_property** — single_property relations can't be rolled up. Convert first.

8. **synced_property_name** — captured from the PATCH response when converting to dual_property. Save it; needed for rollup setup.

9. **Account `Stage` is `status` type, not `select`** — use `{"status": {"name": "Qualified"}}` not `{"select": ...}` when PATCHing Account pages. Using `select` causes a 400 error.

10. **Contacts → Company name lookup** — the `Company` field is a `relation` type. To get the company name, extract `relation[0].id` from the contact, then fetch `GET /v1/pages/{account_id}` and read `properties.Name.title[0].plain_text`.

11. **Activities correct `database_id`** — the Activities `database_id` for page creation is `74091aa3-ed1e-4110-a515-7649a41a3ec9` (get it via `GET /v1/data_sources/565f21bd-e6b5-4c80-90c1-ea2869d0357a` → `parent.database_id`). The memory shorthand ending in `-4110` may be truncated — always verify via the data_source endpoint.
