# Growth Data Foundation

Shared database schema for all Growth use cases.
Install only the tables required by the use cases you've selected.

---

## Dependency Map

| Use Case | Required Tables |
|----------|----------------|
| Lead Nurturing | Accounts, Contacts, Activities, Deals, Content Library |
| Partner Nurturing | Accounts, Contacts, Activities, Partnership |
| Content Intelligence | Content Library |

---

## Data Architecture — 3-Layer Model

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

## Tables

### Accounts (Fact Layer)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Company name |
| Industry | Select | Client-defined |
| Country | Select | Client-defined |
| Company Size | Select | 1–10 / 11–50 / 51–200 / 200+ |
| Source | Select | Referral / Outbound / Event / Inbound / Ad |
| Owner | People | Assigned salesperson |
| Website | URL | |
| Notes | Rich Text | |
| Last Activity Date | Rollup | From Activities.Date — latest_date |
| Activity Count | Rollup | From Activities — count |

---

### Contacts (Fact Layer)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Person's full name |
| Job Title | Rich Text | |
| Company | Relation → Accounts | Enable dual property |
| Email | Email | |
| Phone / WhatsApp | Phone Number | Primary contact number |
| WhatsApp | Phone Number | If different from above |
| LINE ID | Rich Text | |
| LinkedIn | URL | |
| Decision Role | Select | Decision Maker / Influencer / Executor |
| Preferred Channel | Select | Email / WhatsApp / LINE / Phone |
| Notes | Rich Text | |

---

### Activities (Record Layer)

| Field | Type | Notes |
|-------|------|-------|
| Summary | Title | One-line description |
| Date | Date | When it happened |
| Type | Select | Call / Meeting / Demo / Email / Proposal / Contract / Other |
| Account | Relation → Accounts | Enable dual property |
| Contact | Relation → Contacts | Enable dual property |
| Client Response | Rich Text | What the client said |
| Next Action | Rich Text | What to do next |
| Stage Advanced? | Checkbox | Did this move the deal forward? |
| Owner | People | Who handled this |
| Created | Created Time | Auto-filled |

---

### Deals (Relationship Layer)

One deal per opportunity. One company can have multiple deals.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Deal name / opportunity description |
| Account | Relation → Accounts | Which company |
| Stage | Status | Lead → Qualified → Closing → Won → Lost |
| Est. Value | Number | Expected contract value |
| Source | Select | Referral / Outbound / Event / Inbound / Ad |
| Next Action | Rich Text | What to do next |
| Next Follow-up | Date | When to follow up |
| Owner | People | Assigned salesperson |
| Notes | Rich Text | |

---

### Partnership (Relationship Layer)

Resellers, referral partners, and strategic partners.
A partner can simultaneously be an Account.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Partner / organization name |
| Type | Select | Reseller / Referral / Strategic / Distributor |
| Status | Status | Active / Inactive / Prospect |
| Country | Multi-select | Markets they cover |
| Email | Email | Primary contact email |
| Phone | Phone Number | |
| Related to Accounts | Relation → Accounts | If also an Account (dual role) |
| Related to Contacts | Relation → Contacts | Key contact person at partner |
| Remarks | Rich Text | |

---

### Content Library (Standalone)

Stores message templates and AI-generated follow-up drafts.
Shared across Lead Nurturing and Content Intelligence.

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Template or draft title |
| Type | Select | Email Template / WhatsApp Message / Follow-up Draft / Other |
| Stage Target | Multi-select | Which pipeline stages |
| Industry Target | Multi-select | Which industries |
| Channel | Multi-select | Email / WhatsApp / LINE / Phone Script |
| Status | Select | Draft / Ready / Archived |

> Full message body goes in the **page content (blocks)**, not in a table field.

---

## Setup Order

Create tables in this order (relations depend on it):

1. Accounts
2. Contacts → relate to Accounts
3. Deals → relate to Accounts
4. Partnership → relate to Accounts + Contacts
5. Activities → relate to Accounts + Contacts
6. Content Library (standalone, any time)

After setup:
- Share all tables with your Notion integration
- Confirm dual_property relations (both sides linked)
- Add rollups to Accounts after dual relations are confirmed
- Copy all database IDs to `~/.hermes/.env`
