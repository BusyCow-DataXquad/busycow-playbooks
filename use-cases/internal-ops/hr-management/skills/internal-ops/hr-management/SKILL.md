---
name: hr-management
description: >
  AI-powered HR assistant for Hermes Agent. Manages employee records,
  attendance and leave tracking, approval workflows, and payroll/expense
  monitoring — all through natural conversation. Grounded in 6 Notion databases.
version: 1.0.0
author: BusyCow
tags: [HR, Human Resources, Attendance, Payroll, Notion, Internal Ops]
---

# HR 人事管理 — AI 人事行政系統

## Overview

把人事行政從「人工追、人工填、人工通知」變成一句話就能查、一句話就能審。

四個核心功能：
1. **人員名冊管理** — 對話式查詢與修改員工資料
2. **出勤與請假查詢** — 查詢出勤紀錄、請假狀態、剩餘年假
3. **審批處理** — 核准或拒絕請假 / 報帳申請
4. **薪酬與報帳追蹤** — 查詢薪資發放狀態與報帳審核進度

---

## Notion Databases Required

6 張資料表，全部建在 Internal Ops 工作區下。

| 資料表 | 用途 | 審批狀態欄位 |
|--------|------|------------|
| 員工名冊 | 員工基本資料與職涯異動 | 無（純事實） |
| 出勤紀錄 | 每日上下班時間與加班時數 | 無 |
| 請假紀錄 | 各類假別申請與審核 | 待核准 / 已核准 / 已拒絕 / 已取消 |
| 年假管理 | 每位員工年度額度與剩餘天數 | 無 |
| 薪酬發放紀錄 | 每月薪資明細與發放狀態 | 已發放 / 待發放 / 待確認 |
| 報帳請款紀錄 | 員工報帳申請與審核流程 | 待送出 / 待審核 / 審核中 / 已核准 / 已退回 / 已報銷 |

> 審批狀態直接內嵌在請假紀錄與報帳請款紀錄中，不需要獨立審批表。

### 員工名冊 Schema

| 欄位 | 類型 | 說明 |
|------|------|------|
| 員工姓名 | Title | |
| 英文名 | Rich Text | |
| 員工編號 | Rich Text | 格式：EMP-001 |
| 部門 | Select | 業務 / 工程 / 產品 / 設計 / 人資 / 營運 |
| 職稱 | Rich Text | |
| 到職日 | Date | |
| 離職日 | Date | 在職員工留空 |
| 狀態 | Select | 在職 / 試用 / 留職停薪 / 離職 |
| 備註 | Rich Text | |

### 出勤紀錄 Schema

| 欄位 | 類型 | 說明 |
|------|------|------|
| 出勤紀錄 | Title | 格式：A-YYYY-MMDD-姓 |
| 員工 | Relation → 員工名冊 | |
| 上班時間 | Date (datetime) | |
| 下班時間 | Date (datetime) | |
| 累計加班時數 | Number | 單位：小時 |
| 加班事由 | Rich Text | |

### 請假紀錄 Schema

| 欄位 | 類型 | 說明 |
|------|------|------|
| 請假單 | Title | 格式：L-YYYY-NNNN 姓名 假別 |
| 員工 | Relation → 員工名冊 | |
| 假別 | Select | 年假 / 特休 / 病假 / 事假 / 公假 / 補休 / 其他 |
| 起始日 | Date | |
| 結束日 | Date | |
| 天數 | Number | 支援 0.5 天 |
| 原因 | Rich Text | |
| 狀態 | Select | 待核准 / 已核准 / 已拒絕 / 已取消 |

### 年假管理 Schema

| 欄位 | 類型 | 說明 |
|------|------|------|
| 年假計算 | Title | 格式：YYYY-姓名 |
| 員工 | Relation → 員工名冊 | |
| 年度 | Rich Text | |
| 年假總天數 | Number | |
| 已使用天數 | Number | |
| 剩餘天數 | Number | |
| 備註 | Rich Text | 如「留職停薪中」 |

### 薪酬發放紀錄 Schema

| 欄位 | 類型 | 說明 |
|------|------|------|
| 薪酬單 | Title | 格式：P-YYYY-MM-姓名 |
| 員工 | Relation → 員工名冊 | |
| 月份 | Rich Text | 格式：YYYY-MM |
| 底薪 | Number | |
| 加班費 | Number | |
| 津貼 | Number | |
| 扣款 | Number | |
| 實發金額 | Number | |
| 發放狀態 | Select | 已發放 / 待發放 / 待確認 |

### 報帳請款紀錄 Schema

| 欄位 | 類型 | 說明 |
|------|------|------|
| 報帳項目 | Title | 描述性名稱 |
| 員工 | Relation → 員工名冊 | |
| 類別 | Select | 差旅 / 交通 / 餐飲 / 住宿 / 軟體/訂閱 / 辦公用品 / 其他 |
| 報帳日期 | Date | |
| 金額 | Number | |
| 狀態 | Select | 待送出 / 待審核 / 審核中 / 已核准 / 已退回 / 已報銷 |
| 備註 | Rich Text | 附件說明、退回原因等 |
| 成本歸類 | Select | OPEX / COGS / 無 |
| 計入雜支 | Checkbox | |

---

## Feature 1 — 人員名冊管理

### Conversational Triggers

- 「林采恩什麼時候到職？現在年資多久了？」
- 「現在有誰是留職停薪狀態？」
- 「幫我把王柏凱的職稱更新成資深營運，異動日期今天」
- 「幫我新增一位員工：黃怡婷，業務部，到職日 5/1」

### Workflow

1. 查詢時：POST /databases/{員工名冊_DB}/query，依姓名、部門或狀態篩選
2. 修改時：PATCH /pages/{page_id}，確認異動內容後更新
3. 新增時：主動追問缺漏欄位（部門、職稱、到職日至少要有），確認後 POST /pages

回應格式（查詢）：
```
👤 王柏凱（EMP-005）
部門：營運 | 職稱：Operations Specialist
到職日：2022-11-21 | 年資：約 3 年 5 個月
狀態：留職停薪中
```

---

## Feature 2 — 出勤與請假查詢

### Conversational Triggers

- 「4 月 21 日有誰請假？」
- 「林采恩三月的加班時數總計多少？」
- 「張雅筑今年還剩幾天年假？」
- 「這個月有哪些補休還沒休完？」

### Workflow

**請假查詢：**
- POST /databases/{請假紀錄_DB}/query，filter by 起始日 / 假別 / 員工
- 查年假：POST /databases/{年假管理_DB}/query，filter by 員工

**出勤查詢：**
- POST /databases/{出勤紀錄_DB}/query，filter by 員工 + 日期範圍
- 加總累計加班時數

回應格式（請假查詢）：
```
📋 4 月 21 日請假名單：
• 張雅筑 — 事假 1 天（狀態：待核准）

年假剩餘（張雅筑）：7 天（2026 年度）
```

---

## Feature 3 — 審批處理

### Conversational Triggers

- 「現在有哪些請假待核准？」
- 「核准張雅筑的事假申請（L-2026-0005）」
- 「台北出差高鐵那筆報帳現在什麼狀態？」
- 「這個月哪些報帳還在待審核或審核中？」

### Workflow

**查詢待處理：**
- POST /databases/{請假紀錄_DB}/query，filter: 狀態 = 待核准
- POST /databases/{報帳請款紀錄_DB}/query，filter: 狀態 in [待審核, 審核中, 已退回]

**執行審批：**
1. 找到對應記錄的 page_id
2. 確認：「確定要核准 [申請人] 的 [假別/項目]（[日期/金額]）嗎？」
3. 收到確認後：PATCH /pages/{page_id} 更新狀態欄位

**審批狀態對照：**
- 請假：待核准 → 已核准 / 已拒絕
- 報帳：待審核 / 審核中 → 已核准 / 已退回 / 已報銷

> 重要：不能跳過確認直接審批，必須先讓 HR 或主管確認再更新。

---

## Feature 4 — 薪酬與報帳追蹤

### Conversational Triggers

- 「3 月份薪資有哪些還是待發放狀態？」
- 「差旅住宿（台中）那筆報帳被退回了，原因是什麼？」
- 「陳昱廷 3 月實發多少？加班費是多少？」
- 「這個月哪些報帳還在待審核或審核中？」

### Workflow

**薪資查詢：**
- POST /databases/{薪酬發放紀錄_DB}/query，filter by 月份 + 發放狀態
- 列出員工姓名、底薪、加班費、津貼、扣款、實發金額

**報帳查詢：**
- POST /databases/{報帳請款紀錄_DB}/query，filter by 員工 / 狀態 / 類別
- 退回原因在備註欄位中

回應格式（薪資）：
```
💰 2026-03 薪資發放狀況：
待發放：
• 林采恩 — 實發 NT$51,200（底薪 45,000 + 加班費 4,200 + 津貼 2,000）

已發放：4 筆（略）
```

---

## Behavioral Rules

1. **確認再改** — 所有修改（狀態更新、資料修改）都要先顯示確認訊息，收到確認後才執行
2. **追問缺欄** — 新增員工或記錄時，若缺少必要欄位（部門、到職日等），主動追問
3. **審批不跳過** — 審批操作必須列出申請人、項目、日期 / 金額，確認後才更新
4. **不猜員工** — 若姓名有歧義（如「張小姐」），先查詢確認是哪位員工再操作
5. **敏感資料謹慎** — 薪資明細僅在明確詢問時顯示，不在概覽查詢中主動展開

---

## Configuration

```
# Notion
NOTION_TOKEN=                         # Notion integration token

# HR Databases
HR_EMPLOYEES_DB=                      # 員工名冊 DB ID
HR_ATTENDANCE_DB=                     # 出勤紀錄 DB ID
HR_LEAVE_DB=                          # 請假紀錄 DB ID
HR_ANNUAL_LEAVE_DB=                   # 年假管理 DB ID
HR_PAYROLL_DB=                        # 薪酬發放紀錄 DB ID
HR_EXPENSE_DB=                        # 報帳請款紀錄 DB ID
```
