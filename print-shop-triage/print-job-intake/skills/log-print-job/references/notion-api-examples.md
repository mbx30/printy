# Notion API Call Examples

This guide shows exactly how to structure Notion MCP tool calls when creating print job entries.

## Overview

The Notion MCP provides `notion-create-pages` tool. Use it to create new pages in the Print Jobs database.

## Tool Call Structure

```json
{
  "tool_call": "notion-create-pages",
  "parameters": {
    "pages": [
      {
        "parent": {
          "database_id": "32c9cb079ddb807eba29dd54fee53aac"
        },
        "properties": {
          "Job": { "title": [{ "text": { "content": "Job Title Here" } }] },
          "Client": { "relation": [{ "id": "CLIENT_PAGE_ID" }] },
          "Size": { "select": { "name": "8.5x11" } },
          "Quantity": { "number": 100 },
          "Paper Type": { "multi_select": [{ "name": "Cardstock" }] },
          "Rcvd": { "date": { "start": "2026-06-18" } },
          "Promised": { "date": { "start": "2026-06-19" } },
          "Done": { "status": { "name": "Not started" } }
        }
      }
    ]
  }
}
```

## Required Fields

All of these must be present:

| Field | Type | Example Value |
|-------|------|---------------|
| `pages` | array | `[{ parent: {...}, properties: {...} }]` |
| `parent.database_id` | string | `"32c9cb079ddb807eba29dd54fee53aac"` |
| `Job` (title) | object | `{ "title": [{ "text": { "content": "Laminated Menus" } }] }` |
| `Client` (relation) | object | `{ "relation": [{ "id": "PAGE_ID" }] }` |
| `Size` (select) | object | `{ "select": { "name": "8.5x11" } }` |
| `Quantity` (number) | object | `{ "number": 100 }` |
| `Paper Type` (multi_select) | object | `{ "multi_select": [{ "name": "Cardstock" }] }` |
| `Done` (status) | object | `{ "status": { "name": "Not started" } }` |

## Real Example

Print Job for Loma Club:

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
              "id": "loma_club_page_id_here"
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

## Common Mistakes

❌ **WRONG** — `pages` is undefined or missing:
```json
{
  "parent": { "database_id": "..." },
  "properties": { ... }
}
```

✅ **RIGHT** — `pages` is an array containing the object:
```json
{
  "pages": [
    {
      "parent": { "database_id": "..." },
      "properties": { ... }
    }
  ]
}
```

❌ **WRONG** — Title is a string:
```json
{
  "Job": "Laminated Menus"
}
```

✅ **RIGHT** — Title is a Notion title object:
```json
{
  "Job": {
    "title": [
      {
        "text": {
          "content": "Laminated Menus"
        }
      }
    ]
  }
}
```

❌ **WRONG** — Multi-select as a string:
```json
{
  "Paper Type": "Cardstock"
}
```

✅ **RIGHT** — Multi-select as an array of objects:
```json
{
  "Paper Type": {
    "multi_select": [
      {
        "name": "Cardstock"
      }
    ]
  }
}
```

## Batch Creation

To create multiple jobs at once, add multiple objects to the `pages` array:

```json
{
  "pages": [
    {
      "parent": { "database_id": "32c9cb079ddb807eba29dd54fee53aac" },
      "properties": { ... job 1 ... }
    },
    {
      "parent": { "database_id": "32c9cb079ddb807eba29dd54fee53aac" },
      "properties": { ... job 2 ... }
    }
  ]
}
```

## Field Type Reference

### Title (Job field)
```json
{
  "title": [
    {
      "text": {
        "content": "Your title text"
      }
    }
  ]
}
```

### Relation (Client field)
First, query or fetch the Client page ID, then:
```json
{
  "relation": [
    {
      "id": "the_client_page_id"
    }
  ]
}
```

### Select (Size field)
```json
{
  "select": {
    "name": "8.5x11"
  }
}
```

### Number (Quantity, Cost per fields)
```json
{
  "number": 100
}
```

### Multi-select (Paper Type field)
```json
{
  "multi_select": [
    {
      "name": "Cardstock"
    }
  ]
}
```

### Date (Rcvd, Promised fields)
```json
{
  "date": {
    "start": "2026-06-19"
  }
}
```

### Status (Done field)
```json
{
  "status": {
    "name": "Not started"
  }
}
```

## Before Calling the Tool

1. ✅ Verify all required fields are non-empty
2. ✅ Verify Size, Paper Type, and Done values exist in the live Notion schema
3. ✅ Verify Client page ID is correct (fetch from Clients database if needed)
4. ✅ Verify the pages array is not empty
5. ✅ Verify every property object follows the format in this guide

If any check fails, ask the user for clarification before calling the tool.
