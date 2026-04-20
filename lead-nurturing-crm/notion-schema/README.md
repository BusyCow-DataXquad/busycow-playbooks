# Notion Schema — Lead Nurturing CRM

## Architecture: 3-Layer Data Model

This playbook uses a **3-layer architecture** across 5 databases + 1 content store.

```
┌─────────────────────────────────────────────────────┐
│  FACT LAYER — Who exists                            │
│  Accounts (companies)   Contacts (people)           │
└───────────────┬────────────────────┬────────────────┘
                │                    │
┌───────────────▼────────────────────▼────────────────┐
│  RELATIONSHIP LAYER — How they connect to us        │
│  Deals (pipeline)       Partnership (resellers)     │
└───────────────┬────────────────────┬────────────────┘
                │                    │
┌───────────────▼────────────────────▼────────────────┐
│  RECORD LAYER — What happened                       │
│  Activities (every interaction logged here)         │
└─────────────────────────────────────────────────────┘

  Content Library (standalone — templates & drafts)
```

---

## Setup Order

Create databases in this order (relations depend on it):

1. Accounts
2. Contacts (relates to Accounts)
3. Deals (relates to Accounts)
4. Partnership (relates to Accounts + Contacts)
5. Activities (relates to Accounts + Contacts)
6. Content Library (standalone)

---

## FACT LAYER

### 1. Accounts

Tracks companies and organizations — the core entity everything else connects to.

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Company name |
| Industry | Select | Client-defined |
| Country | Select | Client-defined |
| Company Size | Select | 1–10 / 11–50 / 51–200 / 200+ |
| Source | Select | Referral / Outbound / Event / Inbound / Ad |
| Owner | People | Assigned salesperson |
| Website | URL | |
| Notes | Rich Text | |

After creating Activities, add rollups:
- **Last Activity Date** → rollup from Activities.Date (function: latest_date)
- **Activity Count** → rollup from Activities (function: count)

---

### 2. Contacts

Tracks individual people. Always linked to an Account.

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Person's full name |
| Job Title | Rich Text | |
| Company | Relation → Accounts | Link to company (enable dual property) |
| Email | Email | |
| Phone / WhatsApp | Phone Number | Primary contact number |
| WhatsApp | Phone Number | If different from above |
| LINE ID | Rich Text | |
| LinkedIn | URL | |
| Decision Role | Select | Decision Maker / Influencer / Executor |
| Preferred Channel | Select | Email / WhatsApp / LINE / Phone |
| Notes | Rich Text | How we met, referral info |

---

## RELATIONSHIP LAYER

### 3. Deals

Tracks sales pipeline — one deal per opportunity. Separate from the Account so one company can have multiple deals.

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Deal name / opportunity description |
| Account | Relation → Accounts | Which company |
| Stage | Status | e.g. Lead → Qualified → Closing → Won → Lost |
| Est. Value | Number | Expected contract value |
| Source | Select | Referral / Outbound / Event / Inbound / Ad |
| Next Action | Rich Text | What to do next |
| Next Follow-up | Date | When to follow up |
| Owner | People | Assigned salesperson |
| Notes | Rich Text | |

---

### 4. Partnership

Tracks resellers, referral partners, and strategic partners. Partners can also be Accounts (dual role).

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Partner / organization name |
| Type | Select | Reseller / Referral / Strategic / Distributor |
| Status | Status | Active / Inactive / Prospect |
| Country | Multi-select | Markets they cover |
| Email | Email | Primary contact email |
| Phone | Phone Number | |
| Related to Accounts | Relation → Accounts | If also an Account (dual role) |
| Related to Contacts | Relation → Contacts | Key contact person at the partner |
| Remarks | Rich Text | Notes about the partnership |

---

## RECORD LAYER

### 5. Activities

Every customer interaction is logged here. This is the single source of truth for what happened.

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Summary | Title | One-line description of the interaction |
| Date | Date | When it happened |
| Type | Select | Call / Meeting / Demo / Email / Proposal / Contract / Other |
| Account | Relation → Accounts | Which company (enable dual property) |
| Contact | Relation → Contacts | Who you spoke with (enable dual property) |
| Client Response | Rich Text | What the client said |
| Next Action | Rich Text | What to do next |
| Stage Advanced? | Checkbox | Did this move the deal forward? |
| Owner | People | Who handled this |
| Created | Created Time | Auto-filled |

---

## Content Library

Standalone database for message templates and follow-up drafts. Not part of the 3-layer model — used by the AI to store and retrieve outreach content.

| Field | Type | Options / Notes |
|-------|------|-----------------|
| Name | Title | Template or draft title |
| Type | Select | Email Template / WhatsApp Message / Follow-up Draft / Other |
| Stage Target | Multi-select | Which pipeline stages this applies to |
| Industry Target | Multi-select | Which industries this applies to |
| Channel | Multi-select | Email / WhatsApp / LINE / Phone Script |
| Status | Select | Draft / Ready / Archived |

> Full message body goes in the **page content (blocks)**, NOT in a table field.

---

## After Setup

1. Share all 6 databases with your Notion integration
2. Copy each database ID from the URL bar:
   `https://www.notion.so/YOUR-DATABASE-ID?v=...`
3. Add all IDs to `~/.hermes/.env`
4. Confirm dual_property relations are set (both sides linked):
   - Contacts ↔ Accounts (via Company field)
   - Activities ↔ Accounts
   - Activities ↔ Contacts
   - Partnership ↔ Accounts
   - Partnership ↔ Contacts
5. Add rollups to Accounts after dual relations are confirmed
