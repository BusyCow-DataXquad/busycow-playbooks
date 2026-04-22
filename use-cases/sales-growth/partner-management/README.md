# Partner Management

Manage the full partner lifecycle from qualification to performance monitoring — onboarding tracking, monthly performance review, check-in reminders, and co-marketing content matching.

---

## What It Does

Signing a partner is easy — the real work is keeping the partnership active, monitoring their pipeline, and ensuring they have what they need to sell effectively. Most SMEs sign partners and then lose track of them. This use case gives the agent the structure and rules to manage every partner relationship systematically.

The agent handles:
- Evaluating new partner candidates against a defined Partner Profile
- Running a structured onboarding checklist for every new partner
- Proactively reminding the owner to check in with each partner on a monthly cadence
- Generating monthly performance summaries per partner with trend analysis
- Matching and delivering relevant co-marketing content to each partner

---

## Lifecycle

```
Partner Sources → Partner Qualification → Onboarding → Active Management → Performance Review → Tier Upgrade/Exit
```

---

## Recommended Schema

### 1. Partnership
One row per partner organization.

| Field | Type | Notes |
|-------|------|-------|
| Partner Name | title | — |
| Type | select | Reseller / Referral / Strategic / Distributor |
| Status | status | Active / Inactive / Prospect |
| Tier | select | Gold / Silver / Bronze / Prospect |
| Qualification Status | select | Pending Review / Qualified / Not Qualified |
| Onboarding Status | select | Not Started / In Progress / Completed |
| Country/Market | multi-select | — |
| Email | email | — |
| Phone | phone | — |
| Monthly Target | number | — |
| Last Check-in | date | — |
| Remarks | rich text | — |

### 2. Contacts
One row per individual. Shared with Customer Relationship if deployed together; otherwise standalone.

| Field | Type | Notes |
|-------|------|-------|
| Name | title | — |
| Job Title | rich text | — |
| Company | relation | → Partnership (or Accounts if shared) |
| Email | email | — |
| Phone/WhatsApp | phone | — |
| LINE ID | rich text | — |
| LinkedIn | url | — |
| Decision Role | select | Decision Maker / Influencer / Executor |
| Preferred Channel | select | — |
| Notes | rich text | — |

### 3. Activities
One row per interaction. Shared with Customer Relationship if deployed together; otherwise standalone. Use the Account/Partner relation field to distinguish customer vs. partner activities.

| Field | Type | Notes |
|-------|------|-------|
| Summary | title | — |
| Date | date | — |
| Account | relation | → Partnership (or Accounts if shared) |
| Contact | relation | → Contacts |
| Type | select | Call / Meeting / Demo / Email / Proposal / Contract / Other |
| Client Response | rich text | — |
| Next Action | rich text | — |
| Stage Advanced? | checkbox | — |
| Owner | people or rich text | — |

### 4. Content Library
Shared with Customer Relationship if deployed together; otherwise standalone. Partner-specific content includes onboarding guides, case studies for partner sharing, and co-marketing materials.

| Field | Type | Notes |
|-------|------|-------|
| Name | title | — |
| Type | select | e.g. Onboarding Guide / Case Study / Co-marketing Kit |
| Stage Target | multi-select | Which lifecycle stages this content applies to |
| Industry Target | multi-select | — |
| Channel | multi-select | Email / WhatsApp / LINE / etc. |
| Status | select | Draft / Active / Archived |

---

## Features

| # | Feature | Description |
|---|---------|-------------|
| 1 | Partner Source & Qualification | Log partner candidates, evaluate against Partner Profile criteria, score and qualify, flag high-potential partners |
| 2 | Onboarding Tracking | When a new partner is signed, auto-create onboarding todo checklist (training, system access, first deal discussion), track completion, notify partner on each milestone |
| 3 | Partner Relationship Maintenance | Scheduled check-in reminders (default: monthly), log all partner interactions in Activities, track open items from each check-in |
| 4 | Performance Monitoring | Monthly auto-summary of each partner's referral count, deal value vs. target, trend vs. prior month; flag underperforming or at-risk partners |
| 5 | Partner Activity Tracking | Track partner-specific outreach and co-marketing activities, match relevant content from Content Library to each partner's market and send |

---

## Example Interactions

**Logging a new partner candidate:**
> "We just met a distributor in Vietnam — Viet Distribution Co., covers Ho Chi Minh City, 200+ retail clients."
> Agent: logs to Partnership as Prospect, loads Business Core partner strategy, runs qualification scoring, sets next action.

**Onboarding a new partner:**
> "TechReseller just signed the agreement."
> Agent: updates Status to Active, creates onboarding checklist (training scheduled, system access granted, first deal discussion), tracks each step.

**Monthly performance review (automatic):**
> On the 1st of each month, agent pushes: "Monthly Partner Performance Report — here's a summary of each partner's referral count, deal value vs. target, and trend vs. last month. 2 partners flagged as at-risk."

**Content matching:**
> "Send relevant materials to our Silver partners in the retail sector."
> Agent: searches Content Library for retail-relevant partner content, shows what it found, confirms before sending.

---

## Dependencies

- Notion workspace with the Partnership table above (agent creates during SETUP if it doesn't exist)
- Contacts, Activities, and Content Library — may be shared with Customer Relationship use case if deployed together
- Business Core > Partner Strategy page — used to load qualification criteria (agent asks user to define if not present)
- Telegram bot — for monthly performance review delivery and check-in reminders
