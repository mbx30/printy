---
name: gmail-intake
description: >
  API-based alternative to morning-intake. Use this skill when Gmail MCP tools
  are available and screen control is not — for example, when a team member runs
  intake from a chat interface without computer use. Triggers on the same phrases
  as morning-intake: "run morning intake", "collect print jobs", "check for new
  orders". Reads Gmail via API, logs jobs to Notion, and creates reply drafts —
  all without touching the screen.
metadata:
  version: "1.1.0"
  author: "Go Postal"
---

# Gmail Intake Routine

API-based morning intake for Go Postal. Identical workflow logic to `morning-intake` but driven entirely by Gmail MCP tools instead of screen control. Anyone with Gmail MCP and Notion MCP connected can run this skill without computer use.

**Tools required**: Gmail MCP (`mcp__Gmail__*`) · Notion MCP (`mcp__Notion__*`)

**Current reply mode: DRAFT** — create drafts only. Do not send.

## Execution Flow

### Step 1 — Verify Gmail Account

Call `mcp__Gmail__search_threads` with `query: "label:inbox"` as a test. Confirm responses are coming from `gopostalsd@gmail.com`. If credentials point to a different account, stop and alert the user before proceeding.

### Step 2 — Search for New Print Job Emails

Call `mcp__Gmail__search_threads` with:

```
query: "is:unread (subject:print OR subject:menu OR subject:menus OR subject:poster OR subject:flyer OR subject:flier OR subject:bulletin OR subject:card OR subject:sign OR subject:signage OR subject:bookmark OR subject:order OR subject:flyers OR subject:fliers)"
```

Also run separate queries for known client domains from `../morning-intake/references/email-detection-rules.md`, e.g.:

```
query: "is:unread from:lomaclubreservations.com"
```

Returns: list of thread objects with `id`, `snippet`, and message metadata.

### Step 3 — Read and Evaluate Each Email

For each candidate thread, call `mcp__Gmail__get_thread`:

```
threadId: [id from search results]
```

Returns: full message bodies, sender, subject, date.

Load `../morning-intake/references/email-detection-rules.md` for full decision logic. Quick filter:

It IS a new job if the body contains:
- Specific quantities, sizes, or materials
- Request language: "could we please", "we need", "I need", "please print", "can you print"
- An attachment (file to print)

It is NOT a new job if:
- Body is only a thank-you or pickup confirmation
- Body is about billing, charges, claims, or payment
- Body says "scratch", "cancel", or "never mind"
- It is a forwarded email with no new client content

### Step 4 — Star the Email

For each confirmed print job, call `mcp__Gmail__label_message`:

```
messageId: [id of the first/most relevant message in the thread]
labelIds:  ["STARRED"]
```

### Step 5 — Extract Job Details

From the email body, extract every available field. Apply the confidence model:
- **Certain** — explicitly stated → use it
- **Probable** — strongly implied → confirm before writing
- **Unknown** — not present → apply default rule or flag for user

Fields to extract (see `../morning-intake/references/print-job-schema.md` for full schema and rules):
- **Job** (title): item type + premium add-ons only. No size, quantity, or client name.
- **Client**: match to the Notion Clients database. If not found, create a new client page.
- **Quantity**: plain integer
- **Size**: must match an existing Notion option exactly
- **Paper Type**: resolve via decision tree in `print-job-schema.md`
- **Rcvd**: today (America/Los_Angeles), date only
- **Promised**: next business day at 4:00 PM (America/Los_Angeles) unless email states otherwise
- **Done**: always "Not started" for new jobs
- **Cost per**: only if explicitly stated

### Step 6 — Calculate Promised Date

Use the Notion MCP to query the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`) and count jobs with status "Not started" or "In progress".

Load `../morning-intake/references/completion-date-logic.md` and apply the depth → business days table. The result sets the **Promised** date written to Notion and is also used as the completion date in the client reply draft.

If the email specified a due date, use that instead. If queue depth is 20+, flag to Michael before proceeding.

### Step 7 — Run Pre-Flight Gate

Before creating any Notion row, verify every item passes. Do not create a partial row.

- [ ] Target is the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`)
- [ ] Job, Client, Size, Quantity, Paper Type, Promised, Done are all non-empty
- [ ] Size, Paper Type, and Done values exist in the live schema
- [ ] Job title contains billing add-ons and excludes Size/Quantity/Client name
- [ ] Rcvd = today; Promised = next business day 4 PM (or confirmed otherwise); Promised is not before Rcvd
- [ ] No likely duplicate (same Client + Size + Quantity + Paper Type + Promised in last 30 days)
- [ ] All Probable values confirmed

If any item fails: stop, flag the issue, and move to the next email.

### Step 7b — Confirm Before Write

Show the user the extracted entry and wait for a go-ahead. Batch multiple emails into one message:

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

Do not write to Notion until confirmed.

### Step 8 — Create Notion Entry

Use the Notion MCP to create a new page in the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`) with the confirmed fields. Verify the returned values match. Fix any mismatch immediately.

### Step 9 — Create Reply Draft in Gmail

Call `mcp__Gmail__create_draft`:

```
to:       [client email address]
subject:  "Re: [original subject]"
body:     [reply text — see reply-template.md for format and tone]
threadId: [original thread id]
```

Do not set `send: true`. The draft sits in Gmail for the user to review and send manually.

Draft content rules (load `../morning-intake/references/reply-template.md`):
- Address client by first name
- Confirm the order briefly (item + quantity)
- Give the estimated completion date/time from Step 6
- Sign as Michael

### Step 10 — Summary Report

After processing all emails, report to the user:

```
Gmail Intake Complete — [date]

✅ Jobs logged: [N]
⚠️  Flagged for review: [N] (list any ambiguous emails)
❌ Skipped (not print jobs): [N]

Jobs created:
- [Client] — [Job title] — Promised [date]
- ...

Drafts created in Gmail: [N]
```

## Important Rules

- Never create a Notion row while any required field is unknown and unconfirmed.
- Never put Size, Quantity, or Client name in the Job title.
- Never set a same-day Promised date by default.
- Never send a reply — always create a draft.
- If an email is ambiguous, flag it in the summary rather than guessing.
- If a client doesn't exist in Notion, create their Client page immediately (then remind the user to add logo + cover image).

## Reference Files

- `../morning-intake/references/email-detection-rules.md` — known senders, keyword logic, negative signals
- `../morning-intake/references/print-job-schema.md` — full data model, job title rules, paper type decision tree, normalization
- `../morning-intake/references/completion-date-logic.md` — how to estimate completion date from queue depth
- `../morning-intake/references/reply-template.md` — reply voice, format, and examples
