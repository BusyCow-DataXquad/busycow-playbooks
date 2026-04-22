# Lead Nurturing CRM — Setup Instructions

> You are setting up the Lead Nurturing CRM use case for a client. Run through all four phases in order. Do not skip ahead. Report findings to the user before making any changes.

---

## What needs to be set up

This use case requires up to six Notion databases: Accounts, Contacts, Deals, Partnership, Activities, and Content Library. Some clients may already have versions of these. Your job is to find what exists, adapt it where possible, and only create what is genuinely missing.

---

## Phase 1 — Discovery

Search the client's Notion workspace for existing databases that may match each required table.

### Search keywords to use

Use the Notion API `POST /search` endpoint with the following queries, one at a time:

| Target database | Search terms to try |
|-----------------|---------------------|
| Accounts | "Accounts", "Companies", "Clients", "Organizations", "CRM" |
| Contacts | "Contacts", "People", "Leads", "Persons" |
| Deals | "Deals", "Pipeline", "Opportunities", "Sales" |
| Partnership | "Partners", "Partnership", "Resellers", "Channel" |
| Activities | "Activities", "Interactions", "Log", "Meeting notes", "CRM log" |
| Content Library | "Content Library", "Templates", "Drafts", "Content" |

### How to identify a match

For each search result that is a database (not a page), check its properties. A database is likely a match if:

- **Accounts**: has a title field for company name, plus at least 2 of: industry, country/region, website, lead source, owner
- **Contacts**: has a title field for person name, plus at least 2 of: email, phone, job title, relation to companies
- **Deals**: has a title field for deal/opportunity name, plus a status or stage field with pipeline-style values
- **Partnership**: has a title field for partner name, plus a type or category field (reseller / referral / strategic)
- **Activities**: has a title field for interaction summary, plus a date field and at least one relation to Accounts or Contacts
- **Content Library**: has a title field for content name, plus a type/format field and a status field

### Report before proceeding

After searching, report to the user:

- Which databases were found (name, Notion ID, short description of what it contains)
- Which databases were not found
- Any ambiguous matches (e.g. two databases that could both be "Contacts")

Ask the user to confirm matches before moving to Phase 2. If the user says a found database is not the right one, treat it as not found.

---

## Phase 2 — Onboard

For each database that was not found (or that the user rejected), do the following:

1. Present the recommended schema (see Recommended Schema section below) for that database
2. Ask: "Should I create this database via the Notion API, or would you prefer to create it manually and share it with me?"
3. If the user says to create it via API: use `POST /databases` with the recommended schema as the starting point
4. After creation, tell the user: "Please open Notion, find the new database, and share it with your Hermes integration — otherwise I won't be able to read or write to it."
5. Wait for the user to confirm before continuing

Repeat for each missing database.

---

## Phase 3 — Adapt

For each database that was found (and confirmed by the user):

1. Fetch the full schema using `GET /databases/{id}`
2. Compare its properties to the recommended schema fields for that database
3. Identify fields that are in the recommended schema but missing from the client's database
4. Present the gaps to the user:

```
I found your [Database Name] database. Here are the recommended fields that are missing:

- [Field name] ([type]) — [what it's used for]
- [Field name] ([type]) — [what it's used for]

Would you like me to add these fields? I will not change or remove any of your existing fields.
```

5. If the user agrees: use `PATCH /databases/{id}` to add only the missing fields
6. Do NOT rename, delete, or modify any existing fields — only add new ones
7. Confirm each addition: "Added [field name] to [database name]."

---

## Phase 4 — Configure

Once all databases are found or created:

1. Use the memory tool to save all found/created IDs under a clearly labeled memory entry. Save the following structure:

```
Lead Nurturing CRM config:
  accounts_db: <id>
  contacts_db: <id>
  deals_db: <id>
  partnership_db: <id>
  activities_db: <id>
  content_library_db: <id>
  daily_report_chat_id: <id>
  daily_report_thread_id: <id>
  report_language: zh-TW
  report_schedule: 0 1 * * *
```

2. If any Telegram or email credentials are not yet in `~/.hermes/.env`, ask the user to provide them now
3. Ask the user to confirm the report language and schedule if not already provided
4. The agent will read all database IDs and delivery config from memory whenever this use case is active
5. Confirm setup is complete with a summary:

```
Lead Nurturing CRM setup complete.

Databases configured:
- Accounts: [ID]
- Contacts: [ID]
- Deals: [ID]
- Partnership: [ID or "not configured"]
- Activities: [ID]
- Content Library: [ID]

Daily report: [chat ID] / thread [thread ID]
Report language: [language]
Report schedule: [cron expression]

Telegram: [configured / not configured]
Email: [configured / not configured]

All config saved to agent memory. You are ready to start logging interactions.
```

---

## Recommended Schema

> For reference — adapt to what the client has. These are suggestions, not requirements. If the client already has fields that serve the same purpose under different names, use those instead.

### Accounts (Fact Layer — Companies)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Company name |
| Industry | Select | Client-defined options |
| Country | Select | Client-defined options |
| Company Size | Select | e.g. 1–10 / 11–50 / 51–200 / 200+ |
| Source | Select | Referral / Outbound / Event / Inbound / Ad |
| Owner | People | Assigned salesperson |
| Website | URL | |
| Notes | Rich Text | |

Recommended rollups from Activities (configure after relations are set up):
- Last Activity Date (function: latest_date)
- Activity Count (function: count)

### Contacts (Fact Layer — People)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Person's full name |
| Job Title | Rich Text | |
| Company | Relation → Accounts | Which company |
| Email | Email | |
| Phone | Phone Number | Primary contact number |
| WhatsApp | Phone Number | If different from above |
| LINE ID | Rich Text | |
| LinkedIn | URL | |
| Decision Role | Select | Decision Maker / Influencer / Executor |
| Preferred Channel | Select | Email / WhatsApp / LINE / Phone |
| Notes | Rich Text | How we met, referral info, etc. |

### Deals (Relationship Layer — Pipeline)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Deal name or opportunity description |
| Account | Relation → Accounts | Which company |
| Stage | Status | Lead → Qualified → Closing → Won → Lost |
| Est. Value | Number | Expected contract value |
| Source | Select | Referral / Outbound / Event / Inbound / Ad |
| Next Action | Rich Text | What to do next |
| Next Follow-up | Date | When to follow up |
| Owner | People | Assigned salesperson |
| Notes | Rich Text | |

### Partnership (Relationship Layer — Partners & Resellers)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Partner or organization name |
| Type | Select | Reseller / Referral / Strategic / Distributor |
| Status | Status | Active / Inactive / Prospect |
| Country | Multi-select | Markets they cover |
| Email | Email | Primary contact email |
| Phone | Phone Number | |
| Related to Accounts | Relation → Accounts | If also an Account |
| Related to Contacts | Relation → Contacts | Key contact person |
| Remarks | Rich Text | Notes about the partnership |

### Activities (Record Layer — Interaction Log)

| Field | Type | Notes |
|-------|------|-------|
| Summary | Title | One-line description of the interaction |
| Date | Date | When it happened |
| Type | Select | Call / Meeting / Demo / Email / Proposal / Contract / Other |
| Account | Relation → Accounts | |
| Contact | Relation → Contacts | |
| Client Response | Rich Text | What the client said |
| Next Action | Rich Text | What to do next |
| Stage Advanced? | Checkbox | Did this move the deal forward? |
| Owner | People | Who handled this interaction |
| Created | Created Time | Auto-filled by Notion |

### Content Library (Standalone — Templates & Drafts)

| Field | Type | Notes |
|-------|------|-------|
| Name | Title | Template or draft title |
| Type | Select | Email Template / WhatsApp Message / Follow-up Draft / Other |
| Stage Target | Multi-select | Which pipeline stages this applies to |
| Industry Target | Multi-select | Which industries this applies to |
| Channel | Multi-select | Email / WhatsApp / LINE / Phone Script |
| Status | Select | Draft / Ready / Archived |

Note: Content body is stored in Notion page blocks, not in table fields.
