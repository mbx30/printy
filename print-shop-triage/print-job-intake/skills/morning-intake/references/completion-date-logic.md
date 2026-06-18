# Completion Date Logic

Queue depth at the time of intake determines the **Promised** date written to Notion. This replaces the fixed "next business day" default — Promised is now always calculated, not assumed.

## Step 1 — Query Queue Depth

Use the Notion MCP to query the Print Jobs database (`32c9cb079ddb807eba29dd54fee53aac`) and count jobs where:
- Done = "Not started"  
- Done = "In progress"

## Step 2 — Map Depth to Business Days Out

| Active jobs | Business days out | Example (received Monday) |
|-------------|------------------|--------------------------|
| 0–7         | 1 (tomorrow)     | Tuesday at 4:00 PM       |
| 8–12        | 2                | Wednesday at 4:00 PM     |
| 13–20       | 3                | Thursday at 4:00 PM      |
| 20+         | Flag to Michael  | Do not auto-set Promised |

**Business days = Monday–Friday. Skip Saturday and Sunday.**

To calculate "N business days from today":
1. Start from today (America/Los_Angeles).
2. Count forward N weekdays, skipping Saturday and Sunday.
3. Set Promised to that date at 4:00 PM (America/Los_Angeles).

## Step 3 — Client Overrides

If the email specifies a due date or urgency ("need these by Friday", "RUSH", "can we get these by noon"):
- Use the client's requested date as Promised.
- If their date is same-day and queue depth is 8+, flag to Michael rather than confirming automatically.

For ambiguous deadline language, the queue-based calculation is still the default — see the decision table in `print-job-schema.md`. "No rush", "whenever", and "for today" all resolve to the queue calculation, never same-day.
