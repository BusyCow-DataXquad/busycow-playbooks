# Customer Relationship

Manage the full customer lifecycle from lead qualification to long-term retention — ICP scoring, rhythm-based follow-up, daily briefing, bulk outreach, and business card import.

---

## What It Does

Most SMEs have leads but no system to maintain a consistent follow-up rhythm. Deals stall not because the prospect said no, but because the owner forgot to follow up. This use case gives the agent a structured CRM layer and the behavioral rules to make sure no one falls through the cracks at any stage.

The agent handles:
- Logging new leads with source tagging and ICP qualification
- Tracking every interaction and always capturing the next step
- Sending a daily morning briefing with today's follow-ups and draft messages
- Running bulk email outreach campaigns by stage or industry segment
- Importing business cards by photo and writing to Contacts automatically

---

## Lifecycle

```
Lead Sources → ICP Qualification → Active Nurturing → Closing → Won → Long-term Retention
```

---

## Recommended Schema

### 1. Accounts (Customer List)
One row per company or organization.

| Field | Type | Notes |
|-------|------|-------|
| Company Name | title | — |
| Industry | select | — |
| Company Size | select | e.g. 1–10 / 11–50 / 51–200 / 201+ |
| Region | select | — |
| Website | url | — |
| Lead Source | select | Referral / Partner Referral / Outbound / Event / Inbound / Other |
| Owner | people or rich text | — |
| Stage | select | Lead / Qualified / Use Case Confirmed / Closing / Won / Lost |
| ICP Match | select | Strong / Moderate / Weak / Not Assessed |
| Qualification Notes | rich text | — |
| Next Follow-up | date | — |
| Last Activity Date | rollup | from Activities |
| Activity Count | rollup | from Activities |
| Notes | rich text | — |

### 2. Contacts
One row per individual.

| Field | Type | Notes |
|-------|------|-------|
| Name | title | — |
| Job Title | rich text | — |
| Company | relation | → Accounts |
| Email | email | — |
| Phone/WhatsApp | phone | — |
| LINE ID | rich text | — |
| LinkedIn | url | — |
| Decision Role | select | Decision Maker / Influencer / Executor |
| Preferred Channel | select | e.g. Email / WhatsApp / LINE / Call |
| Notes | rich text | — |

### 3. Deals
One row per sales opportunity.

| Field | Type | Notes |
|-------|------|-------|
| Deal Name | title | — |
| Account | relation | → Accounts |
| Stage | status | — |
| Est. Value | number | — |
| Source | rich text | — |
| Next Action | rich text | — |
| Next Follow-up | date | — |
| Owner | people or rich text | — |
| Notes | rich text | — |

### 4. Activities
One row per interaction.

| Field | Type | Notes |
|-------|------|-------|
| Summary | title | — |
| Date | date | — |
| Account | relation | → Accounts |
| Contact | relation | → Contacts |
| Type | select | Call / Meeting / Demo / Email / Proposal / Contract / Other |
| Client Response | rich text | — |
| Next Action | rich text | — |
| Stage Advanced? | checkbox | — |
| Owner | people or rich text | — |

### 5. Content Library
Follow-up templates and outreach materials. Body stored in page blocks.

| Field | Type | Notes |
|-------|------|-------|
| Name | title | — |
| Type | select | e.g. Follow-up Email / Case Study / Proposal Template |
| Stage Target | multi-select | Which pipeline stages this content applies to |
| Industry Target | multi-select | — |
| Channel | multi-select | Email / WhatsApp / LINE / etc. |
| Status | select | Draft / Active / Archived |

---

## Features

| # | Feature | Description |
|---|---------|-------------|
| 1 | Lead Source Management & ICP Qualification | Log and tag lead sources, evaluate new leads against Business Core ICP criteria, auto-score and categorize, flag high-priority leads for immediate action |
| 2 | Customer Follow-up & Rhythm Management | Log interactions via conversation, auto-update Stage and Next Follow-up, daily briefing with personalized follow-up drafts |
| 3 | Daily Follow-up Briefing | Every morning: push today's follow-up list with one-line context per account, auto-generate draft messages, pipeline stall analysis |
| 4 | Bulk Email Outreach | Segment by Stage/Industry, generate personalized emails, show recipient list before sending, log Activities after send |
| 5 | Business Card Import | Photo → AI extraction → confirm context → write to Contacts + link to Account + log Activity |

---

## Example Interactions

**Logging a new lead:**
> "Just got a referral — TechCorp, 50-person SaaS company in Bangkok, spoke to their Head of Sales."
> Agent: logs Account + Contact, tags Lead Source as Referral, runs ICP qualification, sets Next Follow-up.

**Daily briefing:**
> Agent pushes every morning: "You have 4 follow-ups today. Here's your list + draft messages for each."

**Business card import:**
> User sends photo of business card.
> Agent extracts name, title, email, phone, company — asks "Which account should I link this to? What's their decision role? How did you meet?" — then writes to Contacts and logs an Activity.

**Bulk outreach:**
> "Send a re-engagement email to all Qualified accounts in the retail industry."
> Agent shows recipient list + subject + content summary for confirmation before sending.

---

## Dependencies

- Notion workspace with the 5 tables above (agent creates them during SETUP if they don't exist)
- Business Core > Target Audience page — used to load ICP criteria during qualification
- Gmail or SMTP access — required for bulk email outreach feature only
- Telegram bot — for daily briefing delivery
