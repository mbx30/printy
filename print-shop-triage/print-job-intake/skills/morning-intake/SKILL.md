---
name: morning-intake
description: >
  This skill should be used when Michael says "run morning intake", "start my
  morning routine", "collect print jobs", "check for new orders", "run intake",
  or "let's do morning intake". It controls Gmail in Chrome via screen automation
  to find new print job emails, stars them, creates entries in the Notion Print
  Jobs tracker, and drafts confirmation replies to clients — all in one routine.
metadata:
  version: "0.1.0"
  author: "Go Postal"
---

# Morning Intake Routine

Automate Go Postal's morning print job collection. Run this routine each morning to sweep Gmail for new print orders, log them to Notion, and draft client confirmation replies.

## Overview

This skill controls the screen (Chrome + Gmail) for email retrieval and starring, uses the Notion MCP to write tracker entries, and uses screen control to draft replies in Gmail. Do NOT use the Gmail API for reading emails — use computer use only.

**Current reply mode: DRAFT** — compose the reply and leave it as a draft for Michael to review and send manually. Do not auto-send. When Michael confirms a high success rate, this mode can be switched to auto-send.

## Execution Flow

Run these steps sequentially. After each email is processed, move to the next before finishing the session.

### Step 1 — Open Gmail in Chrome

Use computer use to:
1. Take a screenshot to confirm Chrome is visible or open it.
2. Navigate to `https://mail.google.com` in the active tab.
3. Confirm the account is `gopostalsd@gmail.com`. If a different account is showing, switch accounts.

### Step 2 — Search for New Print Job Emails

In the Gmail search bar, enter this query to find candidate emails:

```
is:unread (subject:print OR subject:menu OR subject:menus OR subject:poster OR subject:flyer OR subject:flier OR subject:bulletin OR subject:card OR subject:sign OR subject:signage OR subject:bookmark OR subject:order OR subject:flyers OR subject:fliers)
```

Also check: `is:unread` emails from known client domains even if subject doesn't match keywords (see `references/email-detection-rules.md` for the full sender list).

Take a screenshot and read the result list.

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

For each confirmed print job email, star it in Gmail using the star icon (computer use click).

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

Before creating the Notion entry, query the Print Jobs tracker to count active jobs.

Use the Notion MCP tool `notion-query-database-view` or `notion-fetch` on the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`) to count jobs with status "Not started" or "In progress".

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

### Step 8 — Create Notion Entry

Use the Notion MCP to create a new page in the Print Jobs database with the validated fields.

After creation, verify the returned values match the intended values. Fix any mismatch immediately.

### Step 9 — Draft Client Reply in Gmail

Return to the email in Gmail (computer use). Click Reply.

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
