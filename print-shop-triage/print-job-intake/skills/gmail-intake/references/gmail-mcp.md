# Gmail MCP Tool Reference

> **Optional file.** The `gmail-intake` skill loads this file for tool names and parameters. If the Gmail MCP server is replaced or the email provider changes, update only this file — the skill logic itself does not need to change.

## Server

MCP server name: `Gmail` (tool prefix: `mcp__Gmail__`)
Target account: `gopostalsd@gmail.com`

---

## Tools Used by gmail-intake

### Search for emails (Step 2)

**Tool**: `mcp__Gmail__search_threads`

```
query: "is:unread (subject:print OR subject:menu OR ...)"
```

Returns: list of thread objects with `id`, `snippet`, and message metadata.

Also run separate queries for known client domains from `email-detection-rules.md`, e.g.:
```
query: "is:unread from:lomaclubreservations.com"
```

---

### Read a full email thread (Step 3)

**Tool**: `mcp__Gmail__get_thread`

```
threadId: [id from search results]
```

Returns: full message bodies, sender, subject, date. Use this to evaluate whether the email is a print job.

---

### Star a confirmed print job email (Step 4)

**Tool**: `mcp__Gmail__label_message`

```
messageId: [id of the message to star]
labelIds: ["STARRED"]
```

Apply to the first (or most relevant) message in the thread.

---

### Create a reply draft (Step 9)

**Tool**: `mcp__Gmail__create_draft`

```
to: [client email address]
subject: "Re: [original subject]"
body: [reply text composed per reply-template.md]
threadId: [original thread id]
```

Do not set `send: true`. The draft sits in Gmail for Michael to review and send manually.

---

## Swapping Providers

If this workflow is handed off to a user on a different email platform (Outlook, etc.):

1. Replace this file with a new `[provider]-mcp.md` using the same section headings.
2. Map each step to the equivalent tool in the new provider's MCP server.
3. No changes to `gmail-intake/SKILL.md` are needed as long as step numbers and behavior match.
