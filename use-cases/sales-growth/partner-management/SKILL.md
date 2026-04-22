# Partner Management — Skill

This is the agent's daily operating guide for Partner Management. Load this file at the start of every session. All behavioral rules, feature logic, and decision points are defined here.

---

## Configuration

At the start of each session, load the config saved in agent memory under **'Partner Management config'** and confirm all keys are present:

```
partnership_db, contacts_db, activities_db, content_library_db,
kb_partner_strategy_page, monthly_report_chat_id, monthly_report_thread_id
```

If any key is missing, stop and ask the user to run SETUP.md first.

---

## Core Behavioral Rules

These rules apply across all features. Never violate them.

1. **Monthly performance review runs automatically on the 1st of each month.** No user confirmation required. Push the report to `monthly_report_chat_id` (and `monthly_report_thread_id` if configured).

2. **Check-in reminders are proactive.** The agent initiates check-in reminders based on each partner's Last Check-in date. The owner just needs to act. Do not wait for the user to ask.

3. **Load Business Core partner strategy before qualifying.** Before setting Qualification Status on any partner candidate, load `kb_partner_strategy_page` to check the client's defined qualification criteria. If the page does not exist or is not accessible, stop and ask the user to define 3–5 qualification criteria first. Never qualify based on assumptions.

4. **Always ask about new leads after logging a partner interaction.** After every partner interaction is logged, ask: "Any new leads or deals they mentioned?" If yes, log them as Activities linked to the Partnership record.

5. **Tier changes require actual data, not promises.** Never upgrade or downgrade a partner's Tier (Gold/Silver/Bronze) without first reviewing their actual Activity and deal data. When a tier change is warranted, say: "Based on [N] referrals and [value] in deals this quarter, this partner qualifies for [new tier]. Shall I update their Tier?" Wait for confirmation.

6. **Never advance Onboarding Status without confirmation.** When an onboarding milestone is completed, confirm with the user before updating the status.

7. **Check Content Library before drafting new content.** When preparing materials for a partner, always search `content_library_db` first for existing partner-relevant content. Only draft new content if nothing suitable is found, and note what you searched for.

---

## Feature 1: Partner Source & Qualification

**Trigger:** User mentions a new partner candidate, reseller, or distributor.

**Steps:**
1. Ask for: partner name, type (Reseller/Referral/Strategic/Distributor), country/market, contact name, and how the lead came in.
2. Create a Partnership record with Status = Prospect and Qualification Status = Pending Review.
3. Create a Contact record linked to the Partnership.
4. Load `kb_partner_strategy_page` to retrieve qualification criteria. If the page doesn't exist, ask the user to define 3–5 criteria before continuing.
5. Evaluate the partner candidate against each criterion. For missing data, ask the user.
6. Score and set Qualification Status: Qualified / Not Qualified. For borderline cases, present the scoring to the user and ask for their call.
7. If Qualified, flag for onboarding: "This partner is qualified — ready to move to onboarding. Shall I start the onboarding checklist?"

---

## Feature 2: Onboarding Tracking

**Trigger:** User confirms a partner has been signed / agreement is executed.

**Steps:**
1. Update Partnership Status to Active and Onboarding Status to In Progress.
2. Create a standard onboarding checklist as a sub-task list or page comment. Default checklist:
   - [ ] Onboarding call scheduled
   - [ ] Product/service training completed
   - [ ] System access granted (if applicable)
   - [ ] First deal or lead discussed
   - [ ] Co-marketing materials shared
3. Track each milestone as the user reports completion. Update Onboarding Status to Completed when all items are done.
4. For each completed milestone, ask: "Should I notify the partner?" If yes, draft a notification message and send (or prepare for the user to send).
5. When all milestones are complete, update Onboarding Status = Completed and log an Activity: "Onboarding completed."

---

## Feature 3: Partner Relationship Maintenance

**Schedule:** Check daily for partners whose Last Check-in was 30+ days ago (or as configured by the user). Push reminders proactively.

**Steps:**
1. Query Partnership where Status = Active and Last Check-in is 30+ days ago (or blank).
2. For each partner due for check-in, push a reminder to Telegram: "You haven't checked in with [Partner Name] in [N] days. Would you like me to draft a check-in message?"
3. When the user confirms or asks to draft: generate a personalized check-in message based on the partner's type, market, and last interaction.
4. When the user logs a check-in interaction:
   - Create an Activity record.
   - Ask: "Any open items or follow-ups from this check-in?" Log them as Next Action on the Activity.
   - Ask: "Any new leads or deals they mentioned?" Log if yes.
   - Update Last Check-in on the Partnership record.

---

## Feature 4: Performance Monitoring

**Schedule:** Automatically on the 1st of each month. No user confirmation required.

**Steps:**
1. Query all Active partnerships.
2. For each partner, retrieve from Activities:
   - Number of interactions logged this month
   - Referral count (Activities where Type = Referral or a deal was mentioned)
   - Total deal value attributed to this partner
3. Compare against Monthly Target on the Partnership record.
4. Calculate trend vs. prior month.
5. Flag partners where deal value < 50% of Monthly Target or where Activity count = 0 as at-risk.
6. Push report to Telegram: `monthly_report_chat_id` / `monthly_report_thread_id`.

**Format:**
```
Partner Performance Report — [Month Year]

PARTNER SUMMARY

1. [Partner Name] — [Tier] | [Country/Market]
   Referrals this month: N
   Deal value: [value] vs. target [target] ([% of target])
   Trend vs. last month: ▲ / ▼ / —
   Last check-in: [date]

⚠️ AT-RISK PARTNERS
[Partners flagged for underperformance or no activity]

RECOMMENDED ACTIONS
[One action per at-risk partner]
```

---

## Feature 5: Partner Activity Tracking & Content Matching

**Trigger:** User asks to send materials to a partner, or agent proactively identifies a content opportunity.

**Steps:**
1. Identify the partner(s) and their market/industry from the Partnership record.
2. Search `content_library_db` for content where:
   - Type includes partner-relevant categories (Onboarding Guide, Case Study, Co-marketing Kit)
   - Industry Target matches the partner's market
   - Status = Active
3. Present found content to the user: "I found [N] relevant items in the Content Library for [Partner Name]: [list]. Shall I send these?"
4. If no suitable content is found, note: "No matching content found in the library for [criteria]. Would you like me to draft something new?"
5. After sending, log an Activity: Type = Email (or as appropriate), Summary = "Content shared — [content names]", linked to the Partnership record.

---

## Error Handling

- If a database query fails, tell the user which database failed and ask them to check the Notion connection.
- If `kb_partner_strategy_page` is not accessible, note it and ask the user to provide qualification criteria directly for this session.
- If Telegram delivery fails for the monthly report, retry once. If it fails again, log the error and notify the user at the start of the next conversation.
- Never silently skip a step. If something cannot be completed, tell the user why and what they need to do.
