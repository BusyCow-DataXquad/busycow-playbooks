# Customer Relationship — Setup

Run this guide once per client to connect the agent to the client's Notion workspace and configure all required settings.

---

## Overview

This setup guide walks through:
1. Discovering existing databases in the client's Notion workspace
2. Creating any missing databases
3. Verifying relations between databases
4. Collecting all config values and saving to agent memory

Run setup once. Repeat only if the client restructures their workspace.

---

## Step 1: Discovery

Search the client's Notion workspace for existing databases that match this use case. Use the following search terms:

```
CRM, accounts, customers, leads, contacts, pipeline, deals, activities
```

For each search result, check whether the database schema matches the recommended schema in README.md. Note any databases that can be reused vs. ones that need to be created fresh.

---

## Step 2: Database Setup

Onboard databases in this order — relations must be set up from parent to child:

### 2.1 Accounts (first — all other tables relate to this)
Required fields: Company Name (title), Industry (select), Company Size (select), Region (select), Website (url), Lead Source (select), Owner (people or rich text), Stage (select), ICP Match (select), Qualification Notes (rich text), Next Follow-up (date), Notes (rich text).

Rollup fields (set up after Activities is created): Last Activity Date, Activity Count.

Stage options: Lead / Qualified / Use Case Confirmed / Closing / Won / Lost
ICP Match options: Strong / Moderate / Weak / Not Assessed
Lead Source options: Referral / Partner Referral / Outbound / Event / Inbound / Other

### 2.2 Contacts (second — relates to Accounts)
Required fields: Name (title), Job Title, Company (relation → Accounts), Email, Phone/WhatsApp, LINE ID, LinkedIn, Decision Role (select), Preferred Channel (select), Notes.

Decision Role options: Decision Maker / Influencer / Executor

### 2.3 Deals (third — relates to Accounts)
Required fields: Deal Name (title), Account (relation → Accounts), Stage (status), Est. Value (number), Source, Next Action, Next Follow-up (date), Owner, Notes.

### 2.4 Activities (fourth — relates to Accounts + Contacts)
Required fields: Summary (title), Date, Account (relation → Accounts), Contact (relation → Contacts), Type (select), Client Response (rich text), Next Action (rich text), Stage Advanced? (checkbox), Owner.

Type options: Call / Meeting / Demo / Email / Proposal / Contract / Other

After creating Activities, return to Accounts and add the rollup fields: Last Activity Date and Activity Count (both rolling up from the Activities relation).

### 2.5 Content Library (standalone — can be created at any point)
Required fields: Name (title), Type (select), Stage Target (multi-select), Industry Target (multi-select), Channel (multi-select), Status (select).

Body content is stored in Notion page blocks, not in a field.

---

## Step 3: Business Core — Target Audience Page

This use case requires access to the client's Business Core > Target Audience analysis page. The agent loads this page when evaluating ICP Match for new leads.

Ask the client: "Do you have a Target Audience or ICP analysis page in your Notion workspace?"

- If yes: get the page ID or URL and save it as `kb_ta_page`
- If no: note that ICP qualification will require the user to manually provide ICP criteria each time until the page is created

---

## Step 4: Daily Briefing Delivery

The daily follow-up briefing is pushed via Telegram every morning. Collect:

- **Chat ID** — the Telegram chat or group where the briefing should be sent
- **Thread ID** — the message thread ID within the group (leave blank if not applicable)

To find the Chat ID: send a message to the bot and check `https://api.telegram.org/bot<TOKEN>/getUpdates`.

---

## Step 5: Configure — Save to Agent Memory

Once all databases are confirmed, save the following config to agent memory under the key **'Customer Relationship config'**:

```
accounts_db:            <Notion database ID for Accounts>
contacts_db:            <Notion database ID for Contacts>
deals_db:               <Notion database ID for Deals>
activities_db:          <Notion database ID for Activities>
content_library_db:     <Notion database ID for Content Library>
kb_ta_page:             <Notion page ID for Business Core Target Audience analysis>
daily_report_chat_id:   <Telegram chat ID for daily briefing>
daily_report_thread_id: <Telegram thread ID — blank if not applicable>
```

Confirm each value with the client before saving. Read back the full config and ask: "Does this look correct?"

---

## Step 6: Verify Setup

Run a quick sanity check:

- [ ] Can query Accounts database and retrieve at least the schema
- [ ] Can query Contacts database and confirm relation to Accounts exists
- [ ] Can query Deals database and confirm relation to Accounts exists
- [ ] Can query Activities database and confirm relations to Accounts and Contacts exist
- [ ] Rollup fields on Accounts (Last Activity Date, Activity Count) are working
- [ ] Content Library database is accessible
- [ ] Target Audience page is accessible (or noted as missing)
- [ ] Telegram bot can send a test message to `daily_report_chat_id`

If any check fails, resolve before declaring setup complete.

---

## Done

Tell the client: "Setup is complete. The agent is now configured for Customer Relationship Management. Point me at SKILL.md for daily operations."
