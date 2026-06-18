# Notion API Call Format (Morning Intake)

Reference for structuring Notion MCP `notion-create-pages` calls in the morning intake workflow.

## Quick Reference

The `notion-create-pages` tool requires a `pages` array with this structure:

```json
{
  "pages": [
    {
      "parent": { "database_id": "32c9cb079ddb807eba29dd54fee53aac" },
      "properties": {
        "Job": { "title": [{ "text": { "content": "JOB_TITLE" } }] },
        "Client": { "relation": [{ "id": "CLIENT_PAGE_ID" }] },
        "Size": { "select": { "name": "SIZE" } },
        "Quantity": { "number": QTY },
        "Paper Type": { "multi_select": [{ "name": "PAPER" }] },
        "Rcvd": { "date": { "start": "YYYY-MM-DD" } },
        "Promised": { "date": { "start": "YYYY-MM-DD" } },
        "Done": { "status": { "name": "Not started" } }
      }
    }
  ]
}
```

## Key Points for Morning Intake

1. **`pages` MUST be an array** — even for a single job, wrap it in `[ ]`
2. **Every required field must be present** — do not omit any of the properties listed above
3. **Client page ID** must be looked up from the Clients database (`2ee9cb079ddb809d81f2fa9a9c2a35d3`) first
4. **Size, Paper Type, Done values must match live options exactly** — fetch the schema before writing

## Step-by-Step Before Calling the Tool

1. Confirm all extracted fields are non-empty (Job, Client, Size, Quantity, Paper Type, Promised, Done)
2. Look up the Client page ID by querying the Clients database
3. Verify Size value matches one of the live options
4. Verify Paper Type value matches one of the live options
5. Verify Done = "Not started" (the only option for new jobs)
6. Verify Promised date is formatted as YYYY-MM-DD
7. Construct the `pages` array using the structure above
8. Call `notion-create-pages` with the `pages` parameter
9. Verify the returned page ID and properties match what you sent

## Client Lookup Example

If the client name is "Loma Club", first query:

```json
{
  "tool_call": "notion-fetch",
  "parameters": {
    "id": "2ee9cb079ddb809d81f2fa9a9c2a35d3",
    "query": {
      "filter": {
        "property": "Name",
        "text": {
          "contains": "Loma Club"
        }
      }
    }
  }
}
```

Extract the page ID from the result, then use it in the Client relation.

## Full Morning Intake Example

After extracting and confirming all fields, construct and call:

```json
{
  "pages": [
    {
      "parent": {
        "database_id": "32c9cb079ddb807eba29dd54fee53aac"
      },
      "properties": {
        "Job": {
          "title": [
            {
              "text": {
                "content": "Laminated Menus"
              }
            }
          ]
        },
        "Client": {
          "relation": [
            {
              "id": "abc123def456"
            }
          ]
        },
        "Size": {
          "select": {
            "name": "8.5x14"
          }
        },
        "Quantity": {
          "number": 35
        },
        "Paper Type": {
          "multi_select": [
            {
              "name": "Cardstock"
            }
          ]
        },
        "Rcvd": {
          "date": {
            "start": "2026-06-18"
          }
        },
        "Promised": {
          "date": {
            "start": "2026-06-19"
          }
        },
        "Done": {
          "status": {
            "name": "Not started"
          }
        }
      }
    }
  ]
}
```

## Error Debugging

| Error | Cause | Fix |
|-------|-------|-----|
| "pages" is undefined | Tool called without `pages` parameter | Wrap all properties in `{ "pages": [...] }` |
| Invalid property name | Typo in property name | Use exact live schema names: Job, Client, Size, Quantity, Paper Type, Rcvd, Promised, Done |
| No matching option | Value not in live select/status options | Fetch database schema first to get current options |
| Relation ID not found | Invalid Client page ID | Verify page exists in Clients database |
| Invalid date format | Date string is not YYYY-MM-DD | Use ISO 8601 format: "2026-06-18" |

## Batch Processing

To create multiple jobs from a morning intake session:

```json
{
  "pages": [
    {
      "parent": { "database_id": "32c9cb079ddb807eba29dd54fee53aac" },
      "properties": { ... job 1 properties ... }
    },
    {
      "parent": { "database_id": "32c9cb079ddb807eba29dd54fee53aac" },
      "properties": { ... job 2 properties ... }
    },
    {
      "parent": { "database_id": "32c9cb079ddb807eba29dd54fee53aac" },
      "properties": { ... job 3 properties ... }
    }
  ]
}
```

Call the tool once with all jobs in the array.
