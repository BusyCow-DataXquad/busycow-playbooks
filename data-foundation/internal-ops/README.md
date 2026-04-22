# Internal Ops Data Foundation

Shared data tables for Internal Ops use cases.

---

## Tables

### Internal Discussions

Logs every internal discussion, meeting, or ad-hoc conversation.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Short label for the discussion |
| Date | Date | When the discussion took place |
| Type | Select | e.g. 週會 / 電話 / 線上 / 臨時討論 |
| Status | Select | 待處理 / 處理中 / 已完成 |
| Output Type | Select | 策略更新 / 待辦萃取 / 純記錄 |
| Output Note | Rich Text | Summary of outcomes or key decisions |
| Participants | Rich Text | Who was involved |
| Follow-up Date | Date | When to revisit if needed |
| Related Tasks | Relation → Task Tracker | Tasks extracted from this discussion |

**Used by:** Discussion & Todo Tracker

---

### Task Tracker

Tracks all actionable items and unresolved issues extracted from discussions.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | What needs to be done or discussed |
| Type | Select | 任務 / 討論待續 |
| Status | Select | 待處理 / 進行中 / 已完成 / 取消 |
| Priority | Select | 高 / 中 / 低 |
| Owner | Rich Text | Who is responsible |
| Due Date | Date | Deadline |
| Source | Select | Discussion / Business Core / 自主 / 其他 |
| Note | Rich Text | Extra context |
| Related Discussion | Relation → Internal Discussions | Source discussion (auto back-reference) |

**Used by:** Discussion & Todo Tracker

---

## Setup Order

1. Create **Internal Discussions** database in Notion
2. Create **Task Tracker** database in Notion
3. Add a **Relation** from Internal Discussions → Task Tracker (dual property)
4. Share both databases with your Hermes integration
5. Add both database IDs to `~/.hermes/.env`
