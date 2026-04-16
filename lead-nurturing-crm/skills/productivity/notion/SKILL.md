---
name: notion
description: Notion API for creating and managing pages, databases, and blocks via curl. Search, create, update, and query Notion workspaces directly from the terminal.
version: 1.0.0
author: community
license: MIT
metadata:
  hermes:
    tags: [Notion, Productivity, Notes, Database, API]
    homepage: https://developers.notion.com
prerequisites:
  env_vars: [NOTION_API_KEY]
---

# Notion API

Use the Notion API via curl to create, read, update pages, databases (data sources), and blocks. No extra tools needed — just curl and a Notion API key.

## Prerequisites

1. Create an integration at https://notion.so/my-integrations
2. Copy the API key (starts with `ntn_` or `secret_`)
3. Store it in `~/.hermes/.env`:
   ```
   NOTION_API_KEY=ntn_your_key_here
   ```
4. **Important:** Share target pages/databases with your integration in Notion (click "..." → "Connect to" → your integration name)

## API Basics

All requests use this pattern:

```bash
curl -s -X GET "https://api.notion.com/v1/..." \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json"
```

The `Notion-Version` header is required. This skill uses `2025-09-03` (latest). In this version, databases are called "data sources" in the API.

## Common Operations

### Search

```bash
curl -s -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"query": "page title"}'
```

### Get Page

```bash
curl -s "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03"
```

### Get Page Content (blocks)

```bash
curl -s "https://api.notion.com/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03"
```

### Create Page in a Database

```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "xxx"},
    "properties": {
      "Name": {"title": [{"text": {"content": "New Item"}}]},
      "Status": {"select": {"name": "Todo"}}
    }
  }'
```

### Query a Database

```bash
curl -s -X POST "https://api.notion.com/v1/data_sources/{data_source_id}/query" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {"property": "Status", "select": {"equals": "Active"}},
    "sorts": [{"property": "Date", "direction": "descending"}]
  }'
```

### Create a Database

```bash
curl -s -X POST "https://api.notion.com/v1/data_sources" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "xxx"},
    "title": [{"text": {"content": "My Database"}}],
    "properties": {
      "Name": {"title": {}},
      "Status": {"select": {"options": [{"name": "Todo"}, {"name": "Done"}]}},
      "Date": {"date": {}}
    }
  }'
```

### Update Page Properties

```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"properties": {"Status": {"select": {"name": "Done"}}}}'
```

### Add Content to a Page

```bash
curl -s -X PATCH "https://api.notion.com/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "children": [
      {"object": "block", "type": "paragraph", "paragraph": {"rich_text": [{"text": {"content": "Hello from Hermes!"}}]}}
    ]
  }'
```

## Property Types

Common property formats for database items:

- **Title:** `{"title": [{"text": {"content": "..."}}]}`
- **Rich text:** `{"rich_text": [{"text": {"content": "..."}}]}`
- **Select:** `{"select": {"name": "Option"}}`
- **Multi-select:** `{"multi_select": [{"name": "A"}, {"name": "B"}]}`
- **Date:** `{"date": {"start": "2026-01-15", "end": "2026-01-16"}}`
- **Checkbox:** `{"checkbox": true}`
- **Number:** `{"number": 42}`
- **URL:** `{"url": "https://..."}`
- **Email:** `{"email": "user@example.com"}`
- **Relation:** `{"relation": [{"id": "page_id"}]}`

## Key Differences in API Version 2025-09-03

- **Databases → Data Sources:** Use `/data_sources/` endpoints for queries and schema retrieval/update
- **Two IDs per database:** Each database has both a `database_id` and a `data_source_id`
  - Use `database_id` when creating pages (`parent: {"database_id": "..."}`)
  - Use `data_source_id` when querying (`POST /v1/data_sources/{id}/query`) and updating schema (`PATCH /v1/data_sources/{id}`)
  - The `data_source_id` is the `id` field returned in search results and GET responses
  - The `database_id` is found in the `parent.database_id` field of the data_source object
- **Search results:** Databases return as `"object": "data_source"` with their `data_source_id`
- **PATCH /databases/{id} returns 404** — always use `PATCH /v1/data_sources/{id}` to update schema
- **GET /databases/{id} returns 404** — always use `GET /v1/data_sources/{id}` to fetch schema

### Creating a Database (via /databases, not /data_sources)

Database creation still uses the old `/databases` endpoint with `parent.type` explicitly set:

```bash
curl -s -X POST "https://api.notion.com/v1/databases" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"type": "page_id", "page_id": "xxx"},
    "is_inline": true,
    "title": [{"text": {"content": "My Database"}}],
    "properties": {
      "Name": {"title": {}},
      "Status": {"select": {"options": [{"name": "Draft"}, {"name": "Live"}]}}
    }
  }'
```

Pitfalls:
- `parent.type` must be explicitly set — omitting it causes `body.parent.type should be defined` error
- Do NOT include a `"Title": {"title": {}}` property — the title property is always named `"Name"` and is auto-created; trying to add it returns `Cannot create new title property`
- After creation, the DB only has a `"Name"` property — add others via `PATCH /data_sources/{id}`
- The returned `id` is the `data_source_id`; find `database_id` in `parent.database_id` of the data_source object
- Use `database_id` (from `parent.database_id`) when creating pages with `parent: {"database_id": "..."}`

### Adding a Relation Property to a Database (Schema Update)

Use `data_source_id` for both the target database and the related database:

```bash
curl -s -X PATCH "https://api.notion.com/v1/data_sources/{data_source_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "RelatedTable": {
        "relation": {
          "data_source_id": "target_data_source_id",
          "type": "dual_property",
          "dual_property": {}
        }
      }
    }
  }'
```

Note: Use `"data_source_id"` (not `"database_id"`) inside the relation definition — the API will return a validation error otherwise.

### Converting single_property relation to dual_property

If a relation was created as `single_property`, convert it to `dual_property` (so both tables get the back-reference) by PATCHing the same property with `dual_property`:

```bash
curl -s -X PATCH "https://api.notion.com/v1/data_sources/{data_source_id}" \
  -d '{
    "properties": {
      "RelatedTable": {
        "relation": {
          "data_source_id": "target_data_source_id",
          "type": "dual_property",
          "dual_property": {}
        }
      }
    }
  }'
```

The response contains `dual_property.synced_property_name` — the auto-generated name of the back-reference in the other table (e.g. `"Related to Activities (Account)"`). This name is needed when setting up rollups.

### Adding a Rollup Property

Rollups require the relation to be `dual_property` first. Use the `synced_property_name` as `relation_property_name`:

```bash
curl -s -X PATCH "https://api.notion.com/v1/data_sources/{data_source_id}" \
  -d '{
    "properties": {
      "Last Activity Date": {
        "rollup": {
          "relation_property_name": "Related to Activities (Account)",
          "rollup_property_name": "Date",
          "function": "latest_date"
        }
      }
    }
  }'
```

Common rollup functions: `latest_date`, `earliest_date`, `count`, `sum`, `average`, `max`, `min`, `checked`, `unchecked`, `percent_checked`.

### Block annotations in page body

When adding blocks via `children`, annotations belong in `rich_text[].annotations`, NOT nested inside `rich_text[].text`. Wrong format causes validation error:

```json
// WRONG — causes "annotations should be not present" error
{"text": {"content": "hello", "annotations": {"italic": true}}}

// CORRECT
{"text": {"content": "hello"}, "annotations": {"italic": true}}
```

## Passing JSON bodies via terminal()

When calling the Notion API from `terminal()` (execute_code), **never inline the JSON body directly in the curl command string**. Shell escaping of special characters (parentheses, Unicode, emoji, em-dashes) causes syntax errors.

Always write the payload to a temp file first, then use `-d @file`:

```python
import json, tempfile
payload = {"children": [...]}
with open("/tmp/notion_payload.json", "w") as f:
    json.dump(payload, f)
result = terminal('curl -s -X PATCH "https://api.notion.com/v1/blocks/{id}/children" '
                  '-H "Authorization: Bearer $NOTION_API_KEY" '
                  '-H "Notion-Version: 2025-09-03" '
                  '-H "Content-Type: application/json" '
                  '-d @/tmp/notion_payload.json | jq .object')
```

This avoids all quoting/escaping issues regardless of content.

## Notes

- Page/database IDs are UUIDs (with or without dashes)
- Rate limit: ~3 requests/second average
- The API cannot set database view filters — that's UI-only
- Use `is_inline: true` when creating data sources to embed them in pages
- Add `-s` flag to curl to suppress progress bars (cleaner output for Hermes)
- Pipe output through `jq` for readable JSON: `... | jq '.results[0].properties'`
