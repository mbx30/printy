# Notion API Call Examples

This guide shows exactly how to structure Notion MCP tool calls when creating print job entries.

> **Important**: These examples target the **Notion MCP** `notion-create-pages` tool. This is **not** the raw Notion REST API. The MCP tool uses a flat property format (scalar values), not the REST API's nested objects (`{"title": [{"text": {"content": ...}}]}`). Do not mix the two.

## Always Fetch the Schema First

Before writing, call `notion-fetch` on the Print Jobs database to read its live **data source schema**. The fetch result shows:
- The exact property names and types (the `<database>` SQLite definition)
- The `data_source_id` (a `collection://<id>` URL) you must use as the parent when a database has one or more data sources

```json
// notion-fetch
{
  "id": "32c9cb079ddb807eba29dd54fee53aac"
}
```

Use the property names and formats from that result as the authoritative spec — they supersede anything written here.

## Tool Call Structure

`notion-create-pages` takes a **top-level** `parent` (shared by every page in the call) and a `pages` array. Each item in `pages` has `properties` only — **do not put `parent` inside a page object** (it will be rejected).

```json
// notion-create-pages
{
  "parent": {
    "type": "data_source_id",
    "data_source_id": "DATA_SOURCE_ID_FROM_FETCH"
  },
  "pages": [
    {
      "properties": {
        "Job": "Laminated Menus",
        "Client": "Loma Club",
        "Size": "8.5x14",
        "Quantity": 35,
        "Paper Type": "Cardstock",
        "Rcvd": "2026-06-18",
        "Promised": "2026-06-19",
        "Done": "Not started"
      }
    }
  ]
}
```

Properties are a **flat map of property name → scalar value** (string, number, or null). There is no nested `{"select": {...}}` or `{"title": [...]}` wrapping.

## Required Structure

| Element | Where | Notes |
|---------|-------|-------|
| `pages` | top-level array | Required. Even one page must be wrapped in `[ ]`. |
| `parent` | top-level object | Shared by all pages. Has `type` + matching id field. |
| `parent.type` | inside `parent` | `"data_source_id"`, `"database_id"`, or `"page_id"`. |
| `properties` | inside each page | Flat map: property name → string / number / null. |

Use `data_source_id` (from a `collection://` URL in the fetch result) when the database has one or more data sources. Use `database_id` only for a single-data-source database. `page_id` is for non-database pages.

## Field Formatting (flat values)

The exact accepted value for each property comes from the fetched `<database>` schema. As a general guide for this tracker:

| Property | Type | Example value |
|----------|------|---------------|
| Job | Title | `"Laminated Menus"` (plain string) |
| Client | Relation | Confirm format against the live schema (usually the related page title or ID as a string) |
| Size | Select | `"8.5x14"` (string, must match a live option exactly) |
| Quantity | Number | `35` (plain number, not a string) |
| Paper Type | Multi-select | Confirm format against the live schema (often the option name as a string) |
| Rcvd | Date | `"2026-06-18"` (ISO date string) |
| Promised | Date | `"2026-06-19"` (ISO date string; include time only if the schema accepts it) |
| Done | Status | `"Not started"` (string, must match a live option exactly) |

> Relation and multi-select formatting under the MCP's flat-value convention is not guaranteed identical across schemas — always confirm against the live `notion-fetch` result before writing, rather than guessing.

## Common Mistakes

❌ **WRONG** — `pages` missing entirely (this is the error from issue #11):
```json
{
  "parent": { "database_id": "..." },
  "properties": { "Job": "Laminated Menus" }
}
```

✅ **RIGHT** — `pages` is a top-level array:
```json
{
  "parent": { "type": "data_source_id", "data_source_id": "..." },
  "pages": [
    { "properties": { "Job": "Laminated Menus" } }
  ]
}
```

❌ **WRONG** — `parent` placed inside a page object (rejected; page items don't allow extra fields):
```json
{
  "pages": [
    {
      "parent": { "database_id": "..." },
      "properties": { "Job": "Laminated Menus" }
    }
  ]
}
```

✅ **RIGHT** — `parent` is top-level, shared by all pages:
```json
{
  "parent": { "type": "data_source_id", "data_source_id": "..." },
  "pages": [
    { "properties": { "Job": "Laminated Menus" } }
  ]
}
```

❌ **WRONG** — REST API nested property objects (this MCP does not accept them):
```json
{
  "properties": {
    "Job": { "title": [{ "text": { "content": "Laminated Menus" } }] },
    "Size": { "select": { "name": "8.5x14" } },
    "Quantity": { "number": 35 }
  }
}
```

✅ **RIGHT** — flat scalar values:
```json
{
  "properties": {
    "Job": "Laminated Menus",
    "Size": "8.5x14",
    "Quantity": 35
  }
}
```

## Batch Creation

All pages in one call share the same top-level `parent`. Add more objects to the `pages` array:

```json
{
  "parent": { "type": "data_source_id", "data_source_id": "..." },
  "pages": [
    { "properties": { "Job": "Laminated Menus", "Quantity": 35 } },
    { "properties": { "Job": "Venue Posters", "Quantity": 200 } }
  ]
}
```

## Before Calling the Tool

1. ✅ Fetch the Print Jobs database with `notion-fetch` to get the live schema and `data_source_id`
2. ✅ Verify all required fields are non-empty (Job, Client, Size, Quantity, Paper Type, Promised, Done)
3. ✅ Verify Size, Paper Type, and Done values exist in the live schema option sets
4. ✅ Verify each property is a flat scalar value (string/number/null), not a nested object
5. ✅ Verify `parent` is top-level with a `type`, and `pages` is a non-empty array with no per-page `parent`

If any check fails, ask the user for clarification before calling the tool.
