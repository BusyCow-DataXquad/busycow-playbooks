---
name: notion-api-v2025
description: Quirks and patterns for Notion API version 2025-09-03 — data_sources vs databases, rollups, dual relations, page body content.
version: 1.0.0
tags: [notion, api, crm, database]
---

# Notion API 2025-09-03 — Key Quirks

## Two IDs per database

Every Notion database has both a `database_id` and a `data_source_id`. They are different UUIDs.

- Use `database_id` when **creating pages**: `"parent": {"database_id": "..."}`
- Use `data_source_id` for **querying and schema changes**: `POST /v1/data_sources/{id}/query` and `PATCH /v1/data_sources/{id}`
- Search results return `"object": "data_source"` with the `data_source_id` in the `id` field
- To get `database_id`, call `GET /v1/data_sources/{data_source_id}` and read `parent.database_id`

## Schema changes (add/edit properties)

```bash
curl -s -X PATCH "https://api.notion.com/v1/data_sources/{data_source_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"properties": {"New Field": {"rich_text": {}}}}'
```

- Do NOT use `/v1/databases/{id}` — returns 404
- Cannot create a new title property (one already exists, usually named "Name")
- Title property name in new databases defaults to "Name", not "Title"

## Querying

```bash
curl -s -X POST "https://api.notion.com/v1/data_sources/{data_source_id}/query" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Dual-property relations (for rollups)

To add rollups, the source relation must be `dual_property`. Convert single to dual:

```json
{
  "properties": {
    "Account": {
      "relation": {
        "data_source_id": "{target_ds_id}",
        "type": "dual_property",
        "dual_property": {}
      }
    }
  }
}
```

Notion auto-generates a synced property on the target table named e.g. `"Related to Activities (Account)"`. Use that name in rollup definitions.

## Adding rollups

```json
{
  "properties": {
    "Last Activity Date": {
      "rollup": {
        "relation_property_name": "Related to Activities (Account)",
        "rollup_property_name": "Date",
        "function": "latest_date"
      }
    }
  }
}
```

Common functions: `latest_date`, `earliest_date`, `count`, `sum`, `average`, `percent_checked`

## Status property options

Status options and groups must be updated together. Groups reference option IDs, but passing empty `option_ids: []` works — Notion auto-assigns. Options update replaces existing ones, so include all desired options in one call.

## Creating pages with body content

```python
page_payload = {
    "parent": {"database_id": "..."},
    "properties": {
        "Name": {"title": [{"text": {"content": "Page title"}}]},
        "Type": {"select": {"name": "Success Story"}},
        ...
    },
    "children": [
        {"object": "block", "type": "heading_1", "heading_1": {"rich_text": [{"text": {"content": "H1 text"}}]}},
        {"object": "block", "type": "heading_2", "heading_2": {"rich_text": [{"text": {"content": "H2 text"}}]}},
        {"object": "block", "type": "paragraph", "paragraph": {"rich_text": [{"text": {"content": "Body text"}}]}},
        {"object": "block", "type": "bulleted_list_item", "bulleted_list_item": {"rich_text": [{"text": {"content": "Bullet"}}]}},
        {"object": "block", "type": "divider", "divider": {}}
    ]
}
```

**Pitfall**: Do NOT use `"annotations"` inside `rich_text[].text` — only valid inside `rich_text[]` directly (sibling to `text`). This causes a 400 validation error.

Correct:
```json
{"text": {"content": "hello"}, "annotations": {"bold": true}}
```

Wrong (causes error):
```json
{"text": {"content": "hello", "annotations": {"bold": true}}}
```

## Passing JSON payloads via curl

Use `json.dumps(payload)` in Python and pass as `{json.dumps(payload)}` in the curl `-d` argument. This properly escapes nested quotes.

## Pitfalls

- `/v1/databases/{id}` endpoint returns 404 for read AND write in this API version — always use `/v1/data_sources/`
- **`POST /v1/databases/{id}/query` silently returns empty results** (no error, `object: "list"`, but `results: []`) even when the database has data — this is a silent failure. Always use `POST /v1/data_sources/{data_source_id}/query` for querying.
- When creating a new database via `POST /v1/databases`, properties are NOT saved — the DB is created with only "Name". Add properties separately with PATCH /data_sources after creation.
- `"parent": {"page_id": "..."}` in database creation also needs `"type": "page_id"` field or it fails validation
- **Notion-Version header must be `2025-09-03`** — older versions like `2022-06-28` return `{"code": "invalid_request_url"}` for `/v1/data_sources/` endpoints
- **`database_id` ≠ `data_source_id`** — A given database_id may not be directly accessible as a data_source. If `GET /v1/data_sources/{id}` returns 404, use `POST /v1/search` to list all accessible objects, then call `POST /v1/data_sources/{found_ds_id}/query` and read `results[0].parent.database_id` to get the database_id for page creation
- **Always inspect property types before creating pages** — property types can differ from what the schema name implies. `Stage Target` may be `multi_select` while looking like a `select`, and `Status` may be `select` not `status`. Query with `page_size: 1` first and print all property `type` values to avoid validation errors
- **Rollup `Last Activity Date` returns `null` when no activities exist** — for accounts with zero activity records, fall back to account `created_time` to estimate days-since-contact
