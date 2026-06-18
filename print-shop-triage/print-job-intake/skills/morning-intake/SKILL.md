---
name: morning-intake
description: >
  This skill should be used when Michael says "run morning intake", "start my
  morning routine", "collect print jobs", "check for new orders", "run intake",
  or "let's do morning intake". It controls Gmail in Chrome via screen automation
  to find new print job emails, stars them, creates entries in the Notion Print
  Jobs tracker, and drafts confirmation replies to clients — all in one routine.
metadata:
  version: "0.2.0"
  author: "Go Postal"
---

# Morning Intake Routine

Automate Go Postal's morning print job collection. Run this routine each morning to sweep Gmail for new print orders, log them to Notion, and draft client confirmation replies.

## Overview

This skill has two capability tiers — use whichever matches your tools:

| Step | With computer use + Notion MCP | With Gmail MCP + Notion MCP | No tools (plain chat) |
|------|-------------------------------|----------------------------|-----------------------|
| Read emails | Screen control in Chrome | `mcp__Gmail__search_threads` / `get_thread` | Michael pastes email text |
| Star emails | Screen control click | `mcp__Gmail__label_message` (starred label) | Michael stars manually |
| Write to tracker | `mcp__Notion__notion-create-pages` | `mcp__Notion__notion-create-pages` | Output formatted block for manual paste |
| Draft reply | Screen control in Chrome | `mcp__Gmail__create_draft` | Output reply text for Michael to copy |

**Current reply mode: DRAFT** — compose the reply and leave it as a draft for Michael to review and send manually. Do not auto-send. When Michael confirms a high success rate, this mode can be switched to auto-send.

## Execution Flow

Run these steps sequentially. After each email is processed, move to the next before finishing the session.

### Step 1 — Access Gmail

**Computer use path**: Take a screenshot to confirm Chrome is visible or open it. Navigate to `https://mail.google.com`. Confirm the account is `gopostalsd@gmail.com`; switch accounts if needed.

**Gmail MCP path**: Use `mcp__Gmail__search_threads` directly — no browser needed. Confirm you are operating on the `gopostalsd@gmail.com` account.

**No-tools path**: Ask Michael to paste the email text he wants to log, then proceed from Step 5.

### Step 2 — Search for New Print Job Emails

In the Gmail search bar, enter this query to find candidate emails:

```
is:unread (subject:print OR subject:menu OR subject:menus OR subject:poster OR subject:flyer OR subject:flier OR subject:bulletin OR subject:card OR subject:sign OR subject:signage OR subject:bookmark OR subject:order OR subject:flyers OR subject:fliers)
```

Also check: `is:unread` emails from known client domains even if subject doesn't match keywords (see `references/email-detection-rules.md` for the full sender list).

**Computer use**: take a screenshot and read the result list. **Gmail MCP**: read thread subjects and senders from the tool response.

### Step 3 — Evaluate Each Email

Open each candidate email and determine if it is a **new print job request**. Load `references/email-detection-rules.md` for the full decision logic.

Quick filter — it IS a new job if the body contains:
- Specific quantities, sizes, or materials
- Request language: "could we please", "we need", "I need", "please print", "can you print"
- Attachment (file to print)

It is NOT a new job if:
- Body is only a thank-you or pickup confirmation
- Body is about billing, charges, claims, or payment
- Body says "scratch", "cancel", or "never mind"
- It is Michael's own forwarded email with no new client content

### Step 4 — Star the Email

**Computer use**: click the star icon. **Gmail MCP**: call `mcp__Gmail__label_message` with the STARRED label. **No tools**: note the email in the summary so Michael can star it manually.

### Step 5 — Extract Job Details

From the email, extract every available field. Apply the confidence model:
- **Certain** — explicitly stated → use it
- **Probable** — strongly implied → note it, confirm with Michael before writing to Notion
- **Unknown** — not present → apply default rule or flag for Michael

Fields to extract (see `references/print-job-schema.md` for full schema and rules):
- **Job** (title): item type + premium add-ons only. No size, quantity, or client name in the title.
- **Client**: match to the Notion Clients database. If not found, create a new client page.
- **Quantity**: plain integer
- **Size**: must match an existing Notion option exactly
- **Paper Type**: resolve via decision tree in `references/print-job-schema.md`
- **Rcvd**: today (America/Los_Angeles), date only
- **Promised**: next business day at 4:00 PM (America/Los_Angeles) unless email states otherwise
- **Done**: always "Not started" for new jobs
- **Cost per**: only if explicitly stated

### Step 6 — Check Workload for Estimated Completion Date

Before creating the tracker entry, count active jobs to estimate completion.

**With Notion access**: query the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`) and count jobs with status "Not started" or "In progress". Use `notion-query-database-view` or `notion-fetch`.

**Without Notion access**: ask Michael, "How many jobs are currently in the queue?" and use his answer.

Load `references/completion-date-logic.md` to calculate the estimated completion date based on current queue depth. This estimated date is what you will tell the client.

### Step 7 — Run Pre-Flight Gate

Before creating the Notion row, verify every item passes. Do not create a partial row.

- [ ] Target is the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`)
- [ ] Job, Client, Size, Quantity, Paper Type, Promised, Done are all non-empty
- [ ] Size, Paper Type, and Done values exist in the live schema
- [ ] Job title contains billing add-ons and excludes Size/Quantity/Client name
- [ ] Rcvd = today; Promised = next business day 4 PM (or confirmed otherwise); Promised is not before Rcvd
- [ ] No likely duplicate (same Client + Size + Quantity + Paper Type + Promised in last 30 days)
- [ ] All Probable values were confirmed with Michael

If any item fails: stop, flag the issue to Michael, and move to the next email. Do not create a guessed row.

### Step 7b — Confirm Before Write

Show Michael the extracted entry and wait for a go-ahead before writing to the tracker:

```
Ready to log — confirm?

  Client:   [Client]
  Job:      [Job title]
  Qty:      [Quantity] × [Size]
  Paper:    [Paper Type]
  Rcvd:     [date]
  Promised: [date] at 4:00 PM
  Status:   Not started

Reply ✅ to log, or correct any field.
```

Do not write until Michael confirms. Batch all emails in one message when processing multiple: "Found 3 jobs — confirm all or flag any to fix."

### Step 8 — Write to the Tracker

**With Notion API access** (`notion-create-pages` or equivalent): create a new page in the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`) with the confirmed fields. Verify the returned values match. Fix any mismatch immediately.

**Without Notion access**: output the formatted block for Michael to paste:

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
─────────────────────────────
```

### Step 9 — Draft Client Reply

**Computer use**: return to the email in Gmail and click Reply.

**Gmail MCP**: call `mcp__Gmail__create_draft` with the reply text.

**No tools**: output the reply text for Michael to copy-paste.

Draft a short, professional reply in Michael's voice. Load `references/reply-template.md` for the exact format and tone.


Key rules:
- Address client by first name
- Confirm the order briefly (item + quantity, no need to list every spec)
- Give the estimated completion date/time calculated in Step 6
- Sign as Michael
- Do NOT click Send — leave as a draft for Michael to review

### Step 10 — Summary Report

After processing all emails, report to Michael:

```
Morning Intake Complete — [date]

✅ Jobs logged: [N]
⚠️  Flagged for review: [N] (list any ambiguous emails)
❌ Skipped (not print jobs): [N]

Jobs created:
- [Client] — [Job title] — Promised [date]
- ...

Drafts waiting in Gmail: [N]
```

## Important Rules

- Never create a Notion row while any required field is unknown and unconfirmed.
- Never put Size, Quantity, or Client name in the Job title.
- Never set a same-day Promised date by default.
- Never send a reply — always leave as draft in current mode.
- If an email is ambiguous, flag it in the summary and ask Michael rather than guessing.
- If a client doesn't exist in Notion, create their Client page immediately (then remind Michael to add logo + cover image).

## Reference Files

- `references/email-detection-rules.md` — known senders, keyword logic, negative signals
- `references/print-job-schema.md` — full data model, job title rules, paper type decision tree, normalization
- `references/completion-date-logic.md` — how to estimate completion date from queue depth
- `references/reply-template.md` — Michael's reply voice, format, and examples
