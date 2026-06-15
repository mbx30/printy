# Go Postal — Print Job Intake Plugin

Automates the morning print job intake routine for Go Postal (gopostalsd@gmail.com). Sweeps Gmail for new print orders, creates entries in the Notion Print Jobs tracker, and drafts client confirmation replies.

## Skills

### `morning-intake` — Morning Routine

Run each morning when you arrive at the office.

**Trigger phrases**: "run morning intake", "start my morning routine", "collect print jobs", "check for new orders"

**What it does**:
1. Controls Chrome via screen automation to open Gmail
2. Searches for unread print job emails using subject keywords + known sender domains
3. Evaluates each email to confirm it's a genuine new job (not a thank-you, billing email, or cancellation)
4. Stars confirmed print job emails
5. Extracts job specs (client, quantity, size, paper type, etc.)
6. Checks the Notion queue to estimate the completion date
7. Creates a row in the Print Jobs tracker with all validated fields
8. Drafts a short confirmation reply in Gmail for Michael to review and send

**Current mode**: Draft (replies are saved as Gmail drafts, not sent automatically). Switch to auto-send after proving reliability.

---

### `log-print-job` — Manual Job Entry

Log a single print job from pasted email text or spoken specs.

**Trigger phrases**: "log this job", "add this to the tracker", "create a print job", "new print job from [client]", or paste an email and ask to log it

**What it does**:
- Extracts job details from pasted email text or dictated specs
- Validates all fields against the live Notion schema
- Asks the minimum number of clarifying questions
- Creates one correct row in the Print Jobs tracker
- Returns a confirmation with the new job link

---

## Requirements

- **Notion MCP** must be connected (for reading/writing the Print Jobs and Clients databases)
- **Computer use** must be enabled (for Gmail screen control during morning intake)
- Gmail account: gopostalsd@gmail.com must be signed in to Chrome

## Key Notion Databases

| Database | ID |
|----------|-----|
| Print Jobs | `32c9cb079ddb807eba29dd54fee53aac` |
| Clients | `2ee9cb079ddb809d81f2fa9a9c2a35d3` |

## Notes

- The plugin never guesses a required field — it asks Michael or applies explicit defaults
- All rules (job title format, paper type mapping, date defaults, forbidden patterns) are in `skills/morning-intake/references/print-job-schema.md`
- Michael's reply voice and tone examples are in `skills/morning-intake/references/reply-template.md`
- To switch to auto-send mode, update the Draft Mode section in `reply-template.md`
