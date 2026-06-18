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
  version: "1.0.0"
  author: "Go Postal"
---

# Gmail Intake Routine

API-based morning intake for Go Postal. Identical workflow logic to `morning-intake` but uses Gmail MCP tools instead of screen control. Designed for handoff — anyone with Gmail MCP and Notion MCP connected can run this skill without needing computer use access.

**Tools required**: Gmail MCP · Notion MCP

Load `references/gmail-mcp.md` for the specific tool names and parameters used in each step. If the Gmail provider or MCP server changes, only that file needs updating.

**Current reply mode: DRAFT** — create drafts only. Do not send. Switch to auto-send only after Michael confirms accuracy.

## Execution Flow

### Step 1 — Verify Gmail Account

Load `references/gmail-mcp.md` and use the search tool to run a quick test query. Confirm responses are coming from `gopostalsd@gmail.com`. If credentials point to a different account, stop and alert the user before proceeding.

### Step 2 — Search for New Print Job Emails

Use the Gmail MCP search tool (see `references/gmail-mcp.md` for the exact call) with this query:

```
is:unread (subject:print OR subject:menu OR subject:menus OR subject:poster OR subject:flyer OR subject:flier OR subject:bulletin OR subject:card OR subject:sign OR subject:signage OR subject:bookmark OR subject:order OR subject:flyers OR subject:fliers)
```

Also search for unread emails from known client domains — see `../morning-intake/references/email-detection-rules.md` for the full sender list.

Read thread subjects and senders from the tool response. Fetch the full body of each candidate thread.

### Step 3 — Evaluate Each Email

For each candidate, determine if it is a **new print job request**. Load `../morning-intake/references/email-detection-rules.md` for full decision logic.

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

For each confirmed print job email, apply the STARRED label using the Gmail MCP label tool (see `references/gmail-mcp.md`).

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

### Step 6 — Check Workload for Estimated Completion Date

Use the Notion MCP to query the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`) and count jobs with status "Not started" or "In progress".

Load `../morning-intake/references/completion-date-logic.md` to translate queue depth into an estimated completion date for the client reply.

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

Show the user the extracted entry and wait for a go-ahead. When processing multiple emails, batch them into one message:

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

Use the Gmail MCP draft tool (see `references/gmail-mcp.md`) to create a reply draft on the original thread. Do not send.

Draft content — load `../morning-intake/references/reply-template.md` for format and tone:
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

- `references/gmail-mcp.md` — Gmail MCP tool names, parameters, and query syntax **(optional — swap this file if the email provider or MCP server changes)**
- `../morning-intake/references/email-detection-rules.md` — known senders, keyword logic, negative signals
- `../morning-intake/references/print-job-schema.md` — full data model, job title rules, paper type decision tree, normalization
- `../morning-intake/references/completion-date-logic.md` — how to estimate completion date from queue depth
- `../morning-intake/references/reply-template.md` — reply voice, format, and examples
