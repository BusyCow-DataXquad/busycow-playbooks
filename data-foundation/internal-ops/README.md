# Internal Ops Data Foundation

Shared database schema for all Internal Ops use cases.
Install only the tables required by the use cases you've selected.

---

## Dependency Map

| Use Case | Required Tables |
|----------|----------------|
| HR 人事管理 | 員工名冊, 出勤紀錄, 請假紀錄, 年假管理, 薪酬發放紀錄, 報帳請款紀錄 |

---

## Data Architecture — HR Module

```
┌─────────────────────────────────────────────────────┐
│  MASTER RECORD — Who exists                         │
│  員工名冊 (employee profiles, status, tenure)        │
└───┬───────────────┬──────────────┬──────────────────┘
    │               │              │
┌───▼───────┐ ┌────▼──────┐ ┌────▼──────────────────┐
│  出勤紀錄  │ │  請假紀錄  │ │  薪酬發放紀錄           │
│ (daily    │ │ (leave    │ │ (monthly payroll       │
│ clock-in  │ │ requests  │ │ breakdown + status)    │
│ overtime) │ │ approval) │ └────────────────────────┘
└───────────┘ └────▼──────┘
                   │
         ┌─────────▼────────┐  ┌────────────────────────┐
         │  年假管理          │  │  報帳請款紀錄            │
         │ (annual quota +  │  │ (expense claims +      │
         │  balance per yr) │  │  reimbursement flow)   │
         └──────────────────┘  └────────────────────────┘
```

**核心設計原則：**
- 審批狀態直接嵌入請假紀錄與報帳請款紀錄，不另設獨立審批表
- 員工名冊是其他 5 張表的 relation 錨點，最先建立
- 年假管理每年每人一筆，由 HR 手動設定額度，系統追蹤剩餘

---

## Tables

### 員工名冊 (Master Record)

The anchor table. All other tables relate back to this one.

| Field | Type | Notes |
|-------|------|-------|
| 員工姓名 | Title | Full name |
| 英文名 | Rich Text | |
| 員工編號 | Rich Text | Format: EMP-001 |
| 部門 | Select | 業務 / 工程 / 產品 / 設計 / 人資 / 營運 / 財務 / 其他 |
| 職稱 | Rich Text | |
| 到職日 | Date | |
| 離職日 | Date | Leave blank for active employees |
| 狀態 | Select | 在職 / 試用 / 留職停薪 / 離職 |
| 備註 | Rich Text | Free-form notes |

---

### 出勤紀錄

Daily clock-in/out and overtime records. One row per employee per day (for days with notable records).

| Field | Type | Notes |
|-------|------|-------|
| 出勤紀錄 | Title | Format: A-YYYY-MMDD-姓 |
| 員工 | Relation → 員工名冊 | Enable dual property |
| 上班時間 | Date (datetime) | |
| 下班時間 | Date (datetime) | |
| 累計加班時數 | Number | Unit: hours |
| 加班事由 | Rich Text | Required if overtime > 0 |

---

### 請假紀錄

Leave applications with embedded approval status. One row per leave request.

| Field | Type | Notes |
|-------|------|-------|
| 請假單 | Title | Format: L-YYYY-NNNN 姓名 假別 |
| 員工 | Relation → 員工名冊 | Enable dual property |
| 假別 | Select | 年假 / 特休 / 病假 / 事假 / 公假 / 補休 / 其他 |
| 起始日 | Date | |
| 結束日 | Date | |
| 天數 | Number | Supports 0.5 |
| 原因 | Rich Text | |
| 狀態 | Select | **待核准 / 已核准 / 已拒絕 / 已取消** |

> Approval happens by updating the 狀態 field — no separate approval table needed.

---

### 年假管理

Annual leave quota and balance per employee per year. One row per employee per year.

| Field | Type | Notes |
|-------|------|-------|
| 年假計算 | Title | Format: YYYY-姓名 |
| 員工 | Relation → 員工名冊 | |
| 年度 | Rich Text | e.g. "2026" |
| 年假總天數 | Number | Set by HR at year start |
| 已使用天數 | Number | Update when leave is approved |
| 剩餘天數 | Number | = 總天數 - 已使用天數 |
| 備註 | Rich Text | e.g. "留職停薪中" |

---

### 薪酬發放紀錄

Monthly payroll breakdown and disbursement status. One row per employee per month.

| Field | Type | Notes |
|-------|------|-------|
| 薪酬單 | Title | Format: P-YYYY-MM-姓名 |
| 員工 | Relation → 員工名冊 | |
| 月份 | Rich Text | Format: YYYY-MM |
| 底薪 | Number | |
| 加班費 | Number | |
| 津貼 | Number | |
| 扣款 | Number | |
| 實發金額 | Number | = 底薪 + 加班費 + 津貼 - 扣款 |
| 發放狀態 | Select | **已發放 / 待發放 / 待確認** |

---

### 報帳請款紀錄

Employee expense claims with full reimbursement workflow. One row per claim.

| Field | Type | Notes |
|-------|------|-------|
| 報帳項目 | Title | Descriptive name of the expense |
| 員工 | Relation → 員工名冊 | |
| 類別 | Select | 差旅 / 交通 / 餐飲 / 住宿 / 軟體/訂閱 / 辦公用品 / 其他 |
| 報帳日期 | Date | Date of expense |
| 金額 | Number | |
| 狀態 | Select | **待送出 / 待審核 / 審核中 / 已核准 / 已退回 / 已報銷** |
| 備註 | Rich Text | Attachment notes, rejection reasons, etc. |
| 成本歸類 | Select | OPEX（營業費用）/ COGS（銷貨成本）/ 無 |
| 計入雜支 | Checkbox | |

> Rejection reasons go in the 備註 field. The agent surfaces this when queried.

---

## Setup Order

Create tables in this order (relations depend on 員工名冊):

1. **員工名冊** — no dependencies, create first
2. **出勤紀錄** → relate to 員工名冊
3. **請假紀錄** → relate to 員工名冊
4. **年假管理** → relate to 員工名冊
5. **薪酬發放紀錄** → relate to 員工名冊
6. **報帳請款紀錄** → relate to 員工名冊

After setup:
- Share all 6 tables with your Notion integration
- Copy all database IDs to `~/.hermes/.env` (see `use-cases/internal-ops/hr-management/config-template/`)
- Confirm dual_property relations on 出勤紀錄 and 請假紀錄
