# Partner Management — Setup

Run this guide once per client to connect the agent to the client's Notion workspace and configure all required settings.

---

## Overview

This setup guide walks through:
1. Discovering existing databases in the client's Notion workspace
2. Creating any missing databases (checking for shared tables if Customer Relationship is also deployed)
3. Verifying relations between databases
4. Collecting all config values and saving to agent memory

Run setup once. Repeat only if the client restructures their workspace.

---

## Step 1: Discovery

Search the client's Notion workspace for existing databases that match this use case. Use the following search terms:

```
partner, reseller, distributor, referral, channel, alliance
```

Also check whether the Customer Relationship use case has already been deployed. If it has, the Contacts, Activities, and Content Library tables may already exist and should be reused rather than recreated.

---

## Step 2: Database Setup

### 2.1 Partnership (always required — create if it doesn't exist)
Required fields: Partner Name (title), Type (select), Status (status), Tier (select), Qualification Status (select), Onboarding Status (select), Country/Market (multi-select), Email (email), Phone (phone), Monthly Target (number), Last Check-in (date), Remarks (rich text).

Type options: Reseller / Referral / Strategic / Distributor
Status options: Active / Inactive / Prospect
Tier options: Gold / Silver / Bronze / Prospect
Qualification Status options: Pending Review / Qualified / Not Qualified
Onboarding Status options: Not Started / In Progress / Completed

### 2.2 Contacts
Check first: if Customer Relationship is already deployed, use the existing Contacts database. If not, create a standalone Contacts table with the same schema (see Customer Relationship README.md for field definitions). Save the database ID as `contacts_db` regardless of whether it's shared or new.

### 2.3 Activities
Check first: if Customer Relationship is already deployed, use the existing Activities database. Partner activities are distinguished from customer activities via the Account/Partner relation field. If not, create a standalone Activities table with the same schema. Save the database ID as `activities_db`.

### 2.4 Content Library
Check first: if Customer Relationship is already deployed, use the existing Content Library database. Partner-specific content (onboarding guides, case studies, co-marketing kits) can be added to the shared library with appropriate Type and Stage Target tags. If not, create a standalone Content Library table. Save the database ID as `content_library_db`.

---

## Step 3: Business Core — Partner Strategy Page

This use case requires access to the client's Business Core > Partner Strategy page. The agent loads this page when evaluating qualification criteria for new partner candidates.

Ask the client: "Do you have a Partner Strategy or channel strategy page in your Notion workspace?"

- If yes: get the page ID or URL and save it as `kb_partner_strategy_page`
- If no: note that the agent will ask the user to define 3–5 qualification criteria before proceeding with any partner qualification

---

## Step 4: Monthly Report Delivery

The monthly partner performance report is pushed via Telegram on the 1st of each month. Collect:

- **Chat ID** — the Telegram chat or group where the report should be sent
- **Thread ID** — the message thread ID within the group (leave blank if not applicable)

To find the Chat ID: send a message to the bot and check `https://api.telegram.org/bot<TOKEN>/getUpdates`.

---

## Step 5: Configure — Save to Agent Memory

Once all databases are confirmed, save the following config to agent memory under the key **'Partner Management config'**:

```
partnership_db:              <Notion database ID for Partnership>
contacts_db:                 <Notion database ID for Contacts>
activities_db:               <Notion database ID for Activities>
content_library_db:          <Notion database ID for Content Library>
kb_partner_strategy_page:    <Notion page ID for Business Core Partner Strategy>
monthly_report_chat_id:      <Telegram chat ID for monthly performance report>
monthly_report_thread_id:    <Telegram thread ID — blank if not applicable>
```

Confirm each value with the client before saving. Read back the full config and ask: "Does this look correct?"

---

## Step 6: Verify Setup

Run a quick sanity check:

- [ ] Can query Partnership database and retrieve at least the schema
- [ ] Can query Contacts database and confirm it exists (shared or standalone)
- [ ] Can query Activities database and confirm it exists (shared or standalone)
- [ ] Can query Content Library database and confirm it exists (shared or standalone)
- [ ] Partner Strategy page is accessible (or noted as missing — agent will ask for criteria)
- [ ] Telegram bot can send a test message to `monthly_report_chat_id`

If any check fails, resolve before declaring setup complete.

---

## Done

Tell the client: "Setup is complete. The agent is now configured for Partner Management. Point me at SKILL.md for daily operations."
