# Setup Guide: Project Management

This guide walks through connecting the Project Management use case to your Notion workspace.

## Step 1: Discover Existing Databases

Search your Notion workspace for existing databases before creating anything new. Look for pages or databases with names containing:

- `project`
- `task`
- `milestone`
- `meeting notes`
- `action items`
- `PM`

If you find databases that match the schema in README.md (even partially), use those. The Agent can work with existing structures — you don't need to create from scratch.

## Step 2: Onboard in Order

Connect the databases in this order — each one depends on the one before it:

### 2a. Project List (anchor — connect first)
This is the central table. Every todo and discussion will link back to it.

- Locate or create a database with a title field for project names
- Confirm it has (or you can add): Project Type, Status, Start Date, Target Completion Date, Last Updated Date, Current Milestone, Next Action
- Save the database ID as `project_list_db`

### 2b. Project Todos (connect second)
This table must have a Relation field pointing to Project List.

- Locate or create a database for tasks, todos, or action items
- Confirm it has a Relation field that links back to the Project List database
- Confirm it has Item Type, Priority, Owner, Due Date, Status
- Save the database ID as `project_todos_db`

### 2c. Project Discussions (connect third)
This table also must have a Relation field pointing to Project List.

- Locate or create a database for meeting notes, call logs, or discussion records
- Confirm it has a Relation field that links back to the Project List database
- Confirm it has Date, Channel, Participants, Key Decisions, Pending Items, Pending Items Status
- Save the database ID as `project_discussions_db`

## Step 3: Set Up the Project Templates Page

The templates page is a standalone Notion page (not a database). It contains the pre-built todo lists for each project type.

Search for an existing page with names like:
- `templates`
- `project templates`
- `todo templates`
- `standard procedures`

If a suitable page exists, review whether it already has structured todo lists by project type. If yes, save its page ID as `project_templates_page`.

If no suitable page exists, offer to create it using the 3-group structure defined in README.md:
- Construction / Installation: 3 stages, 14 todos
- Service Deployment: 3 stages, 11 todos
- General: 6 starter todos

## Step 4: Configure Daily Report Schedule

Ask the user when they want the daily project report delivered. Default: 9:00 AM local time.

Save the schedule as `daily_report_schedule` in the config (e.g., `"09:00"`).

## Step 5: Save Config to Agent Memory

Save all settings to agent memory under the key **"Project Management config"**:

```
project_list_db:          <Notion database ID>
project_todos_db:         <Notion database ID>
project_discussions_db:   <Notion database ID>
project_templates_page:   <Notion page ID>
daily_report_schedule:    <time string, e.g. "09:00">
```

## Step 6: Run a Quick Smoke Test

Once connected, verify the setup by asking the Agent:

1. "Show me all current projects" — should return Project List entries
2. "Open a test project" — Agent should ask for project type before anything else
3. "What time will my daily report arrive?" — Agent should confirm the schedule

If any step fails, re-check the database IDs and Relation field connections.

## Troubleshooting

**Agent can't find projects**: Confirm the Notion integration has been shared with the database pages. In Notion, go to each database → Share → Invite your integration.

**Todos not linking to projects**: Check that the Project Todos database has a Relation field that points specifically to the Project List database (not just any database).

**Template not applying**: Confirm the project_templates_page ID is a page (not a database), and that the page content is structured with clear section headers matching the 3 template groups.

**Daily report not arriving**: Check that `daily_report_schedule` is saved and that the Telegram bot or messaging channel is configured in the environment.
