# Notion Schema — Litner Chain CIM

## Overview

This playbook requires 4 Notion databases. Create them in this order (relations depend on it):

1. Accounts
2. Contacts (relates to Accounts)
3. Activities (relates to Accounts + Contacts)
4. Content Library (standalone)

---

## 1. Accounts

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Company name |
| Stage | Status or Select | Lead / Qualified / Closing / Customer / Lost — customize to client |
| Industry | Select | Client-defined |
| Country / Region | Select | Client-defined |
| Company Size | Select | 1–10 / 11–50 / 51–200 / 200+ |
| Source | Select | Referral / Outbound / Event / Inbound / Ad |
| Est. Value | Number | Expected contract value |
| Next Follow-up | Date | |
| Owner | People | Assigned salesperson |
| Website | URL | |
| Notes | Rich Text | |

After creating Activities, add rollups:
- Last Activity Date → rollup from Activities.Date (function: latest_date)
- Activity Count → rollup from Activities (function: count)

---

## 2. Contacts

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Contact person's name |
| Job Title | Rich Text | |
| Account | Relation → Accounts | Link to company (enable dual property) |
| Email | Email | |
| Phone | Phone Number | |
| Decision Role | Select | Decision Maker / Influencer / Executor |
| Preferred Channel | Select | Email / WhatsApp / LINE / Phone |
| Notes | Rich Text | How we met, referral info |

---

## 3. Activities

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Summary | Title | One-line description |
| Date | Date | |
| Type | Select | Call / Meeting / Demo / Email / Proposal / Contract / Other |
| Account | Relation → Accounts | (enable dual property) |
| Contact | Relation → Contacts | (enable dual property) |
| Client Response | Rich Text | What the client said |
| Next Action | Rich Text | What to do next |
| Stage Advanced? | Checkbox | Did this move the deal forward? |
| Owner | People | Who handled this |

---

## 4. Content Library

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Template or draft title |
| Type | Select | Email Template / WhatsApp Message / Follow-up Draft / Other |
| Stage Target | Multi-select | Which pipeline stages |
| Industry Target | Multi-select | Which industries |
| Channel | Multi-select | Email / WhatsApp / LINE / Phone Script |
| Status | Select | Draft / Ready / Archived |

> Full message body goes in the **page content (blocks)**, NOT in a table field.

---

## After Setup

1. Share all 4 databases with your Notion integration
2. Copy each database ID from the URL bar:
   `https://www.notion.so/YOUR-DATABASE-ID?v=...`
3. Add all IDs to `~/.hermes/.env`
4. Confirm relations are set to dual_property (both sides linked)
5. Add rollups to Accounts after dual relations are confirmed
