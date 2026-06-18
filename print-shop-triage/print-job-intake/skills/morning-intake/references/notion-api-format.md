# Notion API Call Format (Morning Intake)

Reference for structuring Notion MCP `notion-create-pages` calls in the morning intake workflow.

> **Important**: This targets the **Notion MCP** tool, not the raw Notion REST API. Properties are **flat scalar values** (string / number / null), not nested REST objects like `{"select": {"name": ...}}`.

## Step 0 — Fetch the Schema First

Before any write, call `notion-fetch` on the Print Jobs database to read its live schema and obtain the `data_source_id` (a `collection://<id>` URL). The fetched `<database>` definition is the authoritative list of property names, types, and option sets.

```json
// notion-fetch  (only takes id / include_discussions / include_transcript — no filter object)
{
  "id": "32c9cb079ddb807eba29dd54fee53aac"
}
```

## Quick Reference

`notion-create-pages` takes a **top-level** `parent` (shared by all pages) and a `pages` array. Each page carries `properties` only — no per-page `parent`.

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
        "Job": "JOB_TITLE",
        "Client": "CLIENT_NAME_OR_ID",
        "Size": "SIZE",
        "Quantity": 0,
        "Paper Type": "PAPER",
        "Rcvd": "YYYY-MM-DD",
        "Promised": "YYYY-MM-DD",
        "Done": "Not started"
      }
    }
  ]
}
```

## Key Points for Morning Intake

1. **`pages` MUST be a top-level array** — even for a single job, wrap it in `[ ]`. (Omitting it is the exact failure from issue #11: "expected array, received undefined".)
2. **`parent` is top-level**, not inside each page. It needs a `type` plus the matching id field. Page array items reject any extra fields, so a per-page `parent` will be rejected.
3. **Properties are flat scalars** — `"Quantity": 35`, not `{"number": 35}`; `"Size": "8.5x14"`, not `{"select": {"name": "8.5x14"}}`.
4. **Use `data_source_id`** from the fetch result when the database has one or more data sources. Use `database_id` only for a single-data-source database.
5. **Match option sets exactly** — Size, Paper Type, and Done values must match the live schema fetched in Step 0.

## Step-by-Step Before Calling the Tool

1. Fetch the Print Jobs database to get the live schema and `data_source_id`
2. Confirm all required fields are non-empty (Job, Client, Size, Quantity, Paper Type, Promised, Done)
3. Resolve the Client (relate to an existing client; confirm the relation value format against the live schema)
4. Verify Size, Paper Type, and Done values match live options
5. Format each property as a flat scalar (string/number/null) — not a nested object
6. Format dates as ISO strings (`YYYY-MM-DD`)
7. Build the top-level `parent` and the `pages` array
8. Call `notion-create-pages`
9. Verify the returned page and properties match what you sent

## Full Morning Intake Example

After fetching the schema and confirming all fields:

```json
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

## Error Debugging

| Error | Cause | Fix |
|-------|-------|-----|
| `"pages": expected array, received undefined` | Tool called without a top-level `pages` array | Wrap pages in `{ "pages": [ ... ] }` |
| Unexpected/extra key rejected on a page | `parent` (or other field) placed inside a page object | Move `parent` to top level; pages hold `properties` only |
| Property type error | Property given as a nested REST object | Use a flat scalar value (string/number/null) |
| No matching option | Value not in the live select/status options | Fetch the schema first and use an exact option |
| Wrong parent | Used `database_id` for a multi-data-source database | Fetch first, then use the `data_source_id` |
| Invalid date | Date not ISO formatted | Use `YYYY-MM-DD` |

## Batch Processing

All jobs in one call share the same top-level `parent`:

```json
{
  "parent": {
    "type": "data_source_id",
    "data_source_id": "DATA_SOURCE_ID_FROM_FETCH"
  },
  "pages": [
    { "properties": { "Job": "Laminated Menus", "Quantity": 35, "Size": "8.5x14", "Done": "Not started" } },
    { "properties": { "Job": "Venue Posters", "Quantity": 200, "Size": "11x17", "Done": "Not started" } }
  ]
}
```

Call the tool once with all jobs in the array.
