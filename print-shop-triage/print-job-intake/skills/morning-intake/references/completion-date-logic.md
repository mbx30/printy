# Completion Date Logic

## How to Estimate

Before creating each Notion entry, query the Print Jobs database for active jobs (status = "Not started" or "In progress"). Use this count to set the estimated completion date communicated to the client.

This estimated date is what goes in the client reply draft. The actual **Promised** field in Notion still defaults to next business day at 4:00 PM unless the client specifies otherwise — but the reply should reflect the realistic estimate based on workload.

## Queue Depth → Estimated Completion

| Active jobs (Not started + In progress) | Estimated completion |
|-----------------------------------------|----------------------|
| 0–3 | Same day (afternoon) or next business morning |
| 4–7 | Next business day by 4 PM |
| 8–12 | 2 business days |
| 13–20 | 3 business days |
| 20+ | Flag to Michael — do not auto-estimate |

**Always use business days (Monday–Friday). Skip weekends.**

## What to Tell the Client

Translate the estimate into a specific, human-readable time. Examples:
- "We'll have these ready by this afternoon."
- "We'll have these ready tomorrow by 4 PM."
- "We can have these ready by [day of week], [date]."

Do not say "2 business days" — convert to an actual date.

## Overrides

If the client specifies a due date or urgency in the email ("need these by Friday", "RUSH", "can we get these by noon"):
- Use their requested date as the Promised date in Notion.
- In the reply, confirm that specific date: "We'll have these ready by [their date]."
- If their date is same-day and the queue is heavy, flag it to Michael instead of confirming automatically.

## Querying Active Jobs

Use the Notion MCP to query the Print Jobs database and filter for:
- Done = "Not started"
- Done = "In progress"

Count the results to determine queue depth, then apply the table above.
