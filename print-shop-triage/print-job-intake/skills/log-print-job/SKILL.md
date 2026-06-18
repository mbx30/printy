---
name: log-print-job
description: >
  This skill should be used when the user says "log this job", "add this to the
  tracker", "create a print job", "add a job", pastes email text and asks to
  log it, or says "new print job from [client]". It extracts job details from
  pasted email text or spoken specs, validates every field, and creates one
  correct row in the Notion Print Jobs tracker. No screen control needed —
  the user provides the email content directly.
metadata:
  version: "0.3.0"
  author: "Print Job Intake Plugin"
---

# Log Print Job

## Before You Begin — Identity

Check memory for the user's name and their company name. Substitute these throughout this skill wherever `[user's name]` and `[company name]` appear.

- If the user's name is not in memory → ask.
- If the company name is not in memory → ask.
- Ask both in one message if neither is available.

---

Create a single new entry in the Notion Print Jobs tracker from pasted email content or spoken specs. This is the manual intake path — the user provides the email text (or dictates the job details), and this skill handles extraction, validation, and writing.

> **Compatibility**: This skill works with any AI assistant. Steps 1–4 are pure reasoning — no tools required. Step 6 (Write) has two paths: API path (Notion MCP or equivalent) and Manual path (formatted output for the user to paste). Use whichever applies.

## When to Use This

- the user pastes an email body and says "log this"
- the user dictates: "Add a job for Loma Club — 35 laminated menus, 8.5x14, cardstock"
- the user wants to log a job without running the full morning intake routine

## Execution Steps

### 1. Extract Fields

From the provided text, extract all available fields using the confidence model:
- **Certain** — explicitly stated → use it
- **Probable** — strongly implied → confirm before writing
- **Unknown** — not present → apply default or ask

**Extraction checklist:**
- **Client**: name in signature, greeting, or "From:" line → match to Notion Clients database
- **Job title**: item type + premium add-ons only. No size, quantity, or client name.
- **Quantity**: look for numbers, "qty", "copies", "pcs"
- **Size**: dimension patterns; normalize to live option strings
- **Paper Type**: explicit material, or resolve via client defaults
- **Promised**: stated date/time, or default to next business day at 4:00 PM (America/Los_Angeles)
- **Rcvd**: default to today (America/Los_Angeles)
- **Done**: always "Not started"
- **Cost per**: only if explicitly stated

Load the full schema, rules, and normalization logic from the morning-intake skill's reference file at:
`skills/morning-intake/references/print-job-schema.md`

### 2. Ask for Missing Required Fields

If any required field (Job, Client, Size, Quantity, Paper Type, Promised, Done) is missing and no default applies, ask the user in one message as a short checklist:

- "Confirm size — I see 8.5x14, correct?"
- "What paper type? (or should I use their default?)"
- "When is this promised for? (I'll default to tomorrow at 4 PM)"

Ask the fewest questions possible. Group all clarifications into one message.

### 3. Calculate Promised Date

Query the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`) via Notion MCP and count jobs with status "Not started" or "In progress". Load `skills/morning-intake/references/completion-date-logic.md` and apply the depth → business days table to set Promised.

If the user stated a specific due date, use that instead. If queue depth is 20+, flag to the user before proceeding.

### 4. Confirm Before Write

Before touching the tracker, show the user a structured summary and wait for a go-ahead:

```
Ready to log — confirm?

  Client:   [Client]
  Job:      [Job title]
  Qty:      [Quantity] × [Size]
  Paper:    [Paper Type]
  Rcvd:     [date]
  Promised: [date] at 4:00 PM
  Status:   Not started

Reply ✅ to log, or correct any field: "[field] = [value]"
```

Do not write anything until the user confirms. If the user corrects a field, update it and re-show the summary.

### 5. Run Pre-Flight Gate

Before creating the row:

- [ ] Target is the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`)
- [ ] Job, Client, Size, Quantity, Paper Type, Promised, Done are all non-empty
- [ ] Size, Paper Type, Done values exist in the live schema option sets
- [ ] Job title contains billing add-ons and excludes Size/Quantity/Client name
- [ ] Rcvd and Promised set correctly; Promised is not before Rcvd
- [ ] Duplicate check done (same Client + Size + Quantity + Paper Type + Promised in last 30 days)
- [ ] All Probable values confirmed

If any box fails: stop and ask. Do not create a partial or guessed row.

### 6. Write to the Tracker

**Path A — API access available** (Notion MCP or equivalent Notion integration):

Use the `notion-create-pages` tool to create a new page. Load the detailed format guide at `references/notion-api-examples.md` for the exact JSON structure required.

**Critical**: First `notion-fetch` the Print Jobs database to get its live schema and `data_source_id`. Then call `notion-create-pages` with a **top-level** `parent` and a top-level `pages` **array**. Properties are flat scalar values (string/number/null), not nested REST objects.

Quick checklist before calling the tool:
- [ ] Fetched the live schema + `data_source_id` via `notion-fetch`
- [ ] All required fields populated (Job, Client, Size, Quantity, Paper Type, Promised, Done)
- [ ] Client resolved against the Clients database (`2ee9cb079ddb809d81f2fa9a9c2a35d3`)
- [ ] Size matches live option exactly (e.g., `8.5x11` or `5 x 8` with correct spacing)
- [ ] Paper Type matches a live option
- [ ] Done = "Not started"
- [ ] Promised date is YYYY-MM-DD format, not before Rcvd
- [ ] `parent` is top-level (with `type`); `pages` is a non-empty array; no per-page `parent`
- [ ] Each property is a flat scalar value, not a nested object

After creation, verify the returned page and properties match what you sent. Fix any mismatch immediately.

**Path B — No API access** (plain chat, no tools):
Output this block for the user to paste directly into Notion:

```
NEW PRINT JOB
─────────────────────────────
Job:        [Job title]
Client:     [Client]
Size:       [Size]
Qty:        [Quantity]
Paper Type: [Paper Type]
Rcvd:       [date]
Promised:   [date] 4:00 PM
Done:       Not started
Cost per:   [value or blank]
─────────────────────────────
```

### 7. Confirm to the user

Return a short confirmation:

```
✅ Job logged!

Client: [Client]
Job: [Job title]
Qty: [Quantity] × [Size]
Paper: [Paper Type]
Rcvd: [date]
Promised: [date + time]
Status: Not started

→ [Notion link to the new job page, if available]
```

If a new client was created, add: "New client page created for [Client] — remember to add a logo and cover image."

## Key Rules

- Never create the row while any required field is unknown and unconfirmed.
- Never put Size, Quantity, or Client name in the Job title.
- Never set same-day Promised by default.
- If client doesn't exist in Notion, create their Client page immediately.
- "Same as last time" with no stated specs = Unknown. Ask, don't copy.
- For Music Box jobs, apply the Music Box naming convention (see print-job-schema.md).

## Reference Files

- `references/notion-api-examples.md` — Exact JSON format for `notion-create-pages` calls (required before writing)
- `skills/morning-intake/references/print-job-schema.md` — Full data model, job title rules, paper type decision tree, size normalization, and client page layout
