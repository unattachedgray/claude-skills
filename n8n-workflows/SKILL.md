---
name: n8n-workflows
description: Manage and trigger n8n automation workflows via MCP tools for email, research, data processing, and multi-step automations
---

# n8n Workflow Tools

You have access to n8n automation workflows via MCP. These are real integrations — they execute actions in the real world.

## Available Outbound Tools (agent calls n8n)

### mcp__n8n__echo
- **Purpose:** Test connectivity to n8n
- **Parameters:** `{ message: string }`
- **Use when:** Testing that n8n integration is working

### mcp__n8n_2__send_email
- **Purpose:** Send email via Julian's Gmail account
- **Parameters:** `{ to: string, subject: string, body: string }`
- **Use when:** User asks to send an email, follow up with someone, or share information via email
- **Important:** Always confirm recipient and content with the user before sending

### mcp__n8n_3__search_drive
- **Purpose:** Search Google Drive for files by name or content
- **Parameters:** `{ query: string }`
- **Returns:** List of matching files with names, types, modification dates, and web links
- **Use when:** User asks to find files in Google Drive, look up documents, or search for specific content

### mcp__n8n_3__download_drive_file
- **Purpose:** Download a file from Google Drive and save it locally
- **Parameters:** `{ query: string }` — Pass a JSON string: `{"file_id": "...", "filename": "optional_name.ext"}`
- **Returns:** Container path where the file was saved (`/workspace/group/downloads/filename`)
- **Use when:** User asks to download, fetch, or get a file from Google Drive
- **Workflow:** Uses the file ID from `search_drive` results. Supports all file types: ZIP, PDF, images, and Google Workspace files (Docs→text, Sheets→CSV)
- **Example:** After finding a file with `search_drive`, download it:
  ```
  search_drive({ query: "project plan" })
  → { files: [{ id: "1abc...", name: "Project Plan.docx", ... }] }
  download_drive_file({ query: '{"file_id": "1abc...", "filename": "project-plan.docx"}' })
  → "File downloaded to /workspace/group/downloads/project-plan.docx"
  ```

## Inbound Workflows (n8n triggers agent)

These workflows run automatically and may trigger you with a prompt:

### Gmail Watcher (real-time)
- **Trigger:** New email arrives (checked every minute)
- **Source:** `gmail-watcher`
- **What you receive:** Sender, subject, and snippet of new emails
- **What to do:** Summarize the email briefly and notify Julian. Flag urgent items. Don't forward full email content — just a concise summary.

### Email Heartbeat (periodic)
- **Trigger:** Every 3 hours
- **Source:** `email-heartbeat`
- **What you receive:** A numbered list of all currently unread emails (from, subject, snippet)
- **What to do:** Send Julian a brief digest of unread emails. Group by urgency/topic. Highlight anything that needs immediate attention. Skip if everything looks low-priority — just confirm "inbox looks quiet."
- **Note:** This only fires when there are unread emails (n8n filters empty inboxes)

### Morning Briefing (daily)
- **Trigger:** Daily at 8:00 AM
- **Source:** `morning-briefing`
- **What you receive:** Pre-gathered data including unread email count and summaries, plus a prompt asking for a daily briefing
- **What to do:** Use the provided email data plus check scheduled tasks, recent conversations, and stored notes. Compile a concise, actionable morning summary with a prioritized action list for the day.

### Drive Watcher (real-time)
- **Trigger:** File created or updated in Google Drive (checked every 5 minutes)
- **Source:** `drive-watcher`
- **What you receive:** File name, type, and web link of changed files
- **What to do:** Briefly notify Julian about the file change. Mention the file name and whether it was created or updated. Include the link so he can open it directly.

## Guidelines

- Always confirm with the user before sending emails or taking irreversible actions
- When triggered by inbound workflows, send your response via `mcp__nanoclaw__send_message`
- Keep automated responses concise — WhatsApp messages should be scannable
- If an inbound trigger seems like spam or noise, log it internally rather than messaging the user
- For heartbeat workflows (email-heartbeat, morning-briefing): n8n pre-gathers data to save tokens. Use the provided data instead of making redundant API calls.
