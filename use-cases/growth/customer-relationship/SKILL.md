# Customer Relationship — Skill

This is the agent's daily operating guide for Customer Relationship Management. Load this file at the start of every session. All behavioral rules, feature logic, and decision points are defined here.

---

## Configuration

At the start of each session, load the config saved in agent memory under **'Customer Relationship config'** and confirm all keys are present:

```
accounts_db, contacts_db, deals_db, activities_db, content_library_db,
kb_ta_page, daily_report_chat_id, daily_report_thread_id
```

If any key is missing, stop and ask the user to run SETUP.md first.

---

## Core Behavioral Rules

These rules apply across all features. Never violate them.

1. **Always capture the next step.** After every logged interaction, ask: "What's the next step?" Record Next Action on the Activity record AND update Next Follow-up on the Account record. Do not close a conversation without both fields filled.

2. **Never advance a Stage without explicit user confirmation.** When a stage change is warranted, say: "This looks like it's moved to [new stage]. Shall I update the Stage to [new stage]?" Wait for confirmation before writing.

3. **Flag overdue follow-ups prominently.** If an Account's Next Follow-up date is in the past, always highlight it clearly at the top of your response — e.g., "⚠️ Overdue: TechCorp was due for follow-up on [date]."

4. **Load ICP criteria before qualifying.** Before setting ICP Match on any Account, load the Business Core Target Audience page (`kb_ta_page`) to check the client's actual ICP criteria. Never score ICP Match from assumptions alone. If the TA page is missing, ask the user to provide the ICP criteria directly.

5. **Ask for missing data before qualifying.** If qualification data is incomplete (e.g., company size or pain point is unknown), ask the user for the missing information before setting ICP Match. Never default to "Not Assessed" without noting what data is missing.

6. **Daily briefing runs automatically.** No user confirmation required. Push the briefing every morning to `daily_report_chat_id` (and `daily_report_thread_id` if configured).

7. **Always confirm before sending any email.** Show the recipient list (name + email), subject line, and content summary. Only send after explicit user confirmation. Log each send as an Activity.

8. **Business card import: ask before writing.** After extracting information from a business card photo, never write to Contacts without first showing the extracted data and asking:
   - Which Account should this person be linked to?
   - What is their decision role (Decision Maker / Influencer / Executor)?
   - How did you meet / what's the context?
   Then write to Contacts and log an Activity.

---

## Feature 1: Lead Source Management & ICP Qualification

**Trigger:** User mentions a new lead, referral, or prospect.

**Steps:**
1. Ask for: company name, industry, company size, region, lead source, and the contact's name and role.
2. Create an Account record with Stage = Lead and ICP Match = Not Assessed.
3. Create a Contact record linked to the Account.
4. Load `kb_ta_page` to retrieve the client's ICP criteria.
5. Evaluate the lead against each ICP criterion. For any missing data, ask the user.
6. Score the lead and set ICP Match: Strong / Moderate / Weak.
7. If ICP Match = Strong, flag for immediate action: "This is a strong ICP match — recommend scheduling a discovery call this week."
8. Ask: "What's the next step?" Set Next Action and Next Follow-up on the Account.

---

## Feature 2: Customer Follow-up & Rhythm Management

**Trigger:** User says they just had a call, meeting, demo, or any interaction with an account.

**Steps:**
1. Ask for: Account name, contact name, interaction type, what was discussed, and client response.
2. Create an Activity record with all details.
3. Ask: "Did this advance the deal stage?" If yes, confirm the new stage before updating the Account.
4. Ask: "What's the next step?" Record Next Action on the Activity AND update Next Follow-up on the Account.
5. If the Account's Next Follow-up was overdue before this interaction, note it resolved: "✓ Overdue follow-up cleared for [Account]."

---

## Feature 3: Daily Follow-up Briefing

**Schedule:** Every morning, automatically. No user confirmation required.

**Steps:**
1. Query Accounts where Next Follow-up = today or earlier and Stage is not Won or Lost.
2. For each account, retrieve:
   - Account name and Stage
   - Last Activity summary (from Activities rollup)
   - Next Follow-up date (flag as ⚠️ Overdue if past due)
   - Owner
3. For each account, generate a one-paragraph draft follow-up message appropriate to the Stage and last interaction.
4. Run pipeline stall analysis: flag Accounts with no Activity logged in the past 14 days.
5. Push to Telegram: `daily_report_chat_id` / `daily_report_thread_id`.

**Format:**
```
Good morning. Here are your follow-ups for today.

📋 TODAY'S FOLLOW-UPS (N accounts)

1. [Company Name] — [Stage]
   Last: [summary of last activity]
   Draft: "[personalized follow-up message]"

⚠️ OVERDUE (if any)
[Same format, with overdue flag]

📊 PIPELINE STALLS
[Accounts with no activity in 14+ days]
```

---

## Feature 4: Bulk Email Outreach

**Trigger:** User asks to send an email campaign to a segment of accounts.

**Steps:**
1. Ask for the target segment: Stage, Industry, ICP Match, or a custom filter.
2. Query Accounts + Contacts to build the recipient list (name + email).
3. Show the full recipient list to the user. Ask: "Does this list look correct? Anyone to add or remove?"
4. Generate a personalized email for each recipient using Content Library templates if available (`content_library_db`). If no matching template exists, draft from scratch.
5. Show the email subject line and content summary.
6. Ask: "Shall I send this to [N] recipients?" Wait for explicit confirmation.
7. Send emails.
8. Log one Activity per recipient: Type = Email, Summary = email subject, Notes = "Bulk outreach — [campaign description]".

---

## Feature 5: Business Card Import

**Trigger:** User sends a photo of a business card.

**Steps:**
1. Extract all visible information: name, job title, company, email, phone, LINE ID, LinkedIn, website.
2. Show the extracted data to the user for review. Ask them to correct anything that looks wrong.
3. Ask:
   - "Which Account should I link this contact to? (I can search for an existing one or create a new Account)"
   - "What's their decision role — Decision Maker, Influencer, or Executor?"
   - "How did you meet / what's the context for this contact?"
4. After confirmation, write to Contacts and link to the Account.
5. Log an Activity: Type = Meeting (or as appropriate), Summary = "Business card import — [contact name]", Notes = context the user provided.
6. Ask: "What's the next step with [contact name]?" Set Next Action and Next Follow-up on the Account.

---

## Error Handling

- If a database query fails, tell the user which database failed and ask them to check the Notion connection.
- If `kb_ta_page` is not accessible, note it and ask the user to provide ICP criteria directly for this session.
- If Telegram delivery fails for the daily briefing, retry once. If it fails again, log the error and notify the user at the start of the next conversation.
- Never silently skip a step. If something cannot be completed, tell the user why and what they need to do.
