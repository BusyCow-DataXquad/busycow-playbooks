---
name: financial-intelligence
description: >
  AI-powered financial intelligence skill for Hermes Agent. Answers real-time
  financial questions in plain language, delivers automated weekly/monthly health
  analyses, and generates monthly and quarterly financial reports — all grounded
  in 5 Notion financial databases.
version: 1.0.0
author: BusyCow
tags: [Finance, Financial Analysis, Notion, Internal Ops, Cashflow, P&L]
---

# Financial Intelligence — 財務智慧分析

## Overview

三個功能，一套財務神經系統：

1. **即時財務問答** — 用白話文問任何財務問題，AI 查五張表即時回答
2. **定期健康分析** — 每週快照 + 每月健康報告，主動推送，不需要問
3. **財務報表生成** — 損益表、現金流摘要、AR 老化分析，按需或定期輸出

核心原則：**所有數字必須來自資料表，不能估算或捏造**。若資料不足以回答問題，直接告知哪張表缺少什麼資料。

---

## Notion Databases Required

5 張財務資料表，對應真實會計系統（Xero / QuickBooks / 鼎新）的底層架構。

| 資料表 | 縮寫 | 用途 |
|--------|------|------|
| Chart of Accounts (COA) | COA | 所有科目的分類骨幹 |
| General Ledger (Journal Entries) | GL | 最底層交易流水帳 |
| Accounts Receivable (Sales Invoices) | AR | 應收款與發票狀態 |
| Accounts Payable (Bills & Expenses) | AP | 應付款與費用單 |
| Bank Feeds (Cash Transactions) | Bank | 銀行帳戶真實進出 |

---

## Schema

### 1. Chart of Accounts (COA)

分類骨幹。所有其他表的 Account_ID 都對應到這裡。

| Field | Type | Notes |
|-------|------|-------|
| Account_Name | Title | e.g. 顧問服務收入、辦公室租金 |
| Account_ID | Rich Text | e.g. 4100, 6200 |
| Account_Type | Select | **Revenue / Expense / Asset / Liability / Equity** |

常見科目分類：
- Revenue (4xxx)：產品銷貨收入、顧問服務收入
- Expense (5xxx–6xxx)：銷貨成本(5000)、薪資費用(6100)、辦公室租金(6200)、差旅費(6300)
- Asset (1xxx)：銀行存款(1100)、應收帳款(1200)
- Liability (2xxx)：應付帳款(2100)

---

### 2. Accounts Receivable / Sales Invoices (AR)

每筆客戶應收發票。老闆問「誰欠我們錢」查此表。

| Field | Type | Notes |
|-------|------|-------|
| Invoice_ID | Title | Format: INV-YYYY-NNN |
| Customer_Name | Rich Text | |
| Category | Select | 產品銷貨收入 / 顧問服務收入 / 其他 |
| Issue_Date | Date | 開立日期 |
| Due_Date | Date | 到期日 |
| Total_Amount | Number | 發票金額 |
| Paid_Amount | Number | 已收金額（部分收款用） |
| Status | Select | **Sent / Paid / Overdue / Cancelled** |

逾期判斷：Due_Date < 今日 AND Status != Paid → Overdue

---

### 3. Accounts Payable / Bills & Expenses (AP)

每筆即將付出去的款項。老闆問「下週要付多少」查此表。

| Field | Type | Notes |
|-------|------|-------|
| Bill_ID | Title | Format: BILL-YYYY-NNN or EXP-YYYY-NNN |
| Vendor_Employee | Rich Text | 供應商名稱或員工姓名 |
| Category | Rich Text | 費用類別（對應 COA） |
| Account_ID | Rich Text | COA 科目代號 |
| Due_Date | Date | |
| Amount | Number | |
| Status | Select | **Unpaid / Awaiting Approval / Paid / Cancelled** |

---

### 4. Bank Feeds / Cash Transactions (Bank)

銀行帳戶每筆進出紀錄。老闆問「帳上剩多少」查此表。

| Field | Type | Notes |
|-------|------|-------|
| Transaction_ID | Title | Format: TXN-YYYY-NNN or OPEN-YYYY-NNN |
| Bank_Account | Select | 各銀行帳戶名稱（e.g. 匯豐-港幣支存） |
| Date | Date | |
| Amount | Number | 正數=收入，負數=支出 |
| Payee | Rich Text | 交易對象 |
| Category | Select | 對應 COA 科目名稱 |
| Reconciled | Checkbox | 是否已對帳 |

現金餘額計算：依 Bank_Account 分組，SUM(Amount) = 當前餘額

---

### 5. General Ledger / Journal Entries (GL)

最底層複式記帳流水。損益表由此表彙總。

| Field | Type | Notes |
|-------|------|-------|
| Journal_ID | Title | Format: J-YYYY-NNNN |
| Date | Date | |
| Account_ID | Rich Text | 對應 COA |
| Description | Rich Text | 交易說明 |
| Debit_Amount | Number | 借方金額 |
| Credit_Amount | Number | 貸方金額 |

損益計算：
- 收入 = Revenue 科目 Credit_Amount 合計
- 費用 = Expense 科目 Debit_Amount 合計
- 毛利 = 收入 - 銷貨成本(5000) Debit
- 淨利 = 收入 - 所有 Expense Debit 合計

---

## Feature 1 — 即時財務問答

### Conversational Triggers

- 「我們現在帳上有多少錢？各個戶頭分別多少？」
- 「扣掉下週要付的款，實際可動用的還有多少？」
- 「星O科技那筆 150,000 到期了，有沒有收到？」
- 「這個月費用最大的幾個科目是什麼？」
- 「有哪些發票已經逾期還沒收款？」

### Query Logic

**現金餘額：**
```
Bank Feeds → GROUP BY Bank_Account → SUM(Amount) per account
回答格式：
匯豐-港幣支存：HKD 615,000
國泰世華-台幣帳戶：TWD 420,000
花旗-美金帳戶：USD 420,000（請注意幣別不同）
合計可動用現金：[列出各幣別，不做跨幣換算]
```

**扣除待付款後可動用現金：**
```
Bank Feeds SUM - AP WHERE Status=Unpaid AND Due_Date <= 本週末 SUM
```

**應收逾期查詢：**
```
AR WHERE Status=Overdue OR (Due_Date < TODAY AND Status != Paid)
→ 列出：客戶名稱、Invoice_ID、金額、逾期天數
```

**費用科目分析：**
```
GL WHERE Account_Type=Expense → JOIN COA → GROUP BY Account_Name → SUM(Debit_Amount)
→ 按金額降序排列
```

### Response Format

```
💰 現金狀況（截至今日）

匯豐-港幣支存：HKD 615,000
國泰世華-台幣帳戶：TWD 420,000
花旗-美金帳戶：USD 420,000

⚠️ 下週到期應付款：HKD 180,000（聯O電腦 BILL-2026-001，設備採購）
扣除後港幣可動用：HKD 435,000
```

---

## Feature 2 — 定期健康分析

### Weekly Snapshot — Cron Setup

每週一早上 9am，隨 Daily Report 推送。

```
Cron schedule: 0 1 * * 1  (Monday 01:00 UTC = 09:00 TWN)

Prompt:
你是財務分析助理。執行以下流程並以繁體中文純文字輸出：

1. 查詢 Bank Feeds，計算各戶頭當前餘額
2. 查詢 AR，找出本週新增發票與逾期未收款
3. 查詢 AP，找出本週到期應付款
4. 輸出格式：

📊 本週財務快照（YYYY-MM-DD）

💵 現金水位
[各帳戶餘額]

📥 應收動態
• 本週新增：[Invoice_ID] [客戶] [金額]
• 逾期未收：[列出 Overdue 項目]

📤 本週應付
• 到期待付：[列出 Due_Date 在本週的 AP]

⚠️ 注意事項：[異常項目，如有]
```

### Monthly Health Report — Cron Setup

每月 1 日早上 9am 推送上月健康報告。

```
Cron schedule: 0 1 1 * *  (1st of month, 01:00 UTC)

Prompt:
你是財務分析助理。分析上個月（YYYY-MM）的財務狀況並輸出健康報告：

1. 查詢 GL（上月），計算：
   - 總收入（Revenue 科目 Credit 合計）
   - 總費用（Expense 科目 Debit 合計）
   - 毛利 = 收入 - COGS(5000)
   - 毛利率 = 毛利 / 收入
2. 查詢 Bank Feeds（上月），計算淨現金流（收入筆數 - 支出筆數）
3. 查詢 AP，找出上月未付款（Status=Unpaid/Awaiting Approval）
4. 對比上上月數據，判斷趨勢（↑ ↓ →）

輸出格式：
📋 YYYY年MM月 財務健康報告

💹 損益摘要
收入：XXX | 費用：XXX | 毛利：XXX（毛利率 XX%）
趨勢：vs 上月 ↑↓→ XX%

💵 現金流
月初餘額：XXX → 月末餘額：XXX（淨變化 XXX）

⚠️ 異常警示
[費用超標 / 逾期應收 / 未付帳款 等]

📌 建議關注
[1–2 條具體建議]
```

---

## Feature 3 — 財務報表生成

### Conversational Triggers

- 「幫我出 3 月份損益表」
- 「這季的現金流量摘要給我看一下」
- 「應收帳款老化分析，列出超過 30 天沒收的」
- 「Q1 跟 Q2 的毛利對比」

### P&L Report (損益表)

```
查詢 GL WHERE Date in [月份範圍] → JOIN COA
→ 分組計算：
  Revenue 科目：Credit_Amount 合計 → 各科目收入
  COGS (5000)：Debit_Amount 合計
  Operating Expenses (6xxx)：按科目列出 Debit 合計

輸出：
損益表 — YYYY年MM月

📈 收入
  產品銷貨收入 (4000)：XXX
  顧問服務收入 (4100)：XXX
  ──────────────
  總收入：XXX

📉 成本與費用
  銷貨成本 (5000)：XXX
  薪資費用 (6100)：XXX
  辦公室租金 (6200)：XXX
  差旅費 (6300)：XXX
  ──────────────
  總費用：XXX

💰 毛利：XXX（毛利率 XX%）
💰 淨利：XXX（淨利率 XX%）
```

### AR Aging Analysis (應收帳款老化分析)

```
查詢 AR WHERE Status != Paid AND Status != Cancelled
計算逾期天數 = TODAY - Due_Date

輸出：
應收帳款老化分析（截至 YYYY-MM-DD）

  0–30 天：[列出]
  31–60 天：[列出] ⚠️
  60 天以上：[列出] 🔴

總未收款：XXX
建議立即催收：[逾期 30 天以上的客戶]
```

### Cash Flow Summary (現金流摘要)

```
查詢 Bank Feeds WHERE Date in [期間]
→ 分類：期初餘額、收入項目、支出項目、期末餘額

輸出：
現金流量摘要 — YYYY年MM月

期初餘額：XXX
  (+) 收款：[主要項目]
  (-) 付款：[主要項目]
期末餘額：XXX
淨變化：XXX
```

---

## Behavioral Rules

1. **數字必須來自資料** — 絕不估算或編造財務數字；若資料不足，說明缺少哪張表的哪個欄位
2. **幣別分開呈現** — 多幣別帳戶不做跨幣換算，各幣別分開列出
3. **時間範圍確認** — 若問題未指定時間範圍，先確認「是指本月還是上月？」再查詢
4. **異常主動標示** — 查詢時若發現逾期應收、未付帳款、費用超標，一律在回答末尾加 ⚠️ 警示
5. **週報/月報自動執行** — Cron job 不需用戶觸發，直接查詢資料後輸出，無需確認
6. **報表按需生成** — 用戶要求報表時，查詢完整資料後一次輸出，格式整齊

---

## Configuration

```
# Notion
NOTION_TOKEN=                         # Notion integration token

# Finance Databases
FINANCE_COA_DB=                       # Chart of Accounts DB ID
FINANCE_GL_DB=                        # General Ledger DB ID
FINANCE_AR_DB=                        # Accounts Receivable DB ID
FINANCE_AP_DB=                        # Accounts Payable DB ID
FINANCE_BANK_DB=                      # Bank Feeds DB ID

# Telegram (for weekly/monthly reports)
FINANCE_TELEGRAM_CHAT_ID=             # e.g. -100XXXXXXXXX
FINANCE_TELEGRAM_THREAD_ID=           # Thread/topic ID for finance reports
```
