---
name: fuse-linear-ticket
description: Creates a Linear ticket using the Linear MCP server. Can be invoked standalone or from other skills.
---

# Linear Ticket Creation

## Prerequisites

- The Linear MCP server must be connected. Verify by attempting to list teams.
- If Linear is not available, inform the caller/user and exit gracefully.

## Input

This skill accepts the following context (provided by the user or a calling skill):

- **title** (required) — The ticket title
- **description** (required) — Markdown-formatted ticket body
- **source** (optional) — Where this request originated (e.g., "PR review", "bug triage", manual)
- **labels** (optional) — Suggested labels
- **priority** (optional) — Suggested priority (urgent, high, medium, low, none)
- **project_hint** (optional) — A Linear issue identifier, branch name, PR title, or commit message to infer the project from

## Workflow

### Step 1: Verify Linear is available

1. Attempt to list Linear teams
2. If Linear is not connected, inform the user: "Linear is not available. Please ensure the Linear MCP server is connected."
3. Exit gracefully if unavailable

### Step 2: Resolve team and project from context

#### Auto-detect from `project_hint`

If a `project_hint` is provided (branch name, PR title, commit message), attempt to extract a Linear issue identifier using these patterns:

- **Team prefix with issue number:** `ENG-123`, `FE-456`, `PLAT-789` — matches the pattern `[A-Z]{2,10}-\d+`
- **Common branch formats:**
  - `feat/ENG-123-some-description`
  - `fix/ENG-123/some-description`
  - `ENG-123-some-description`
  - `bugfix/ENG-123_some_description`
- **PR title or commit message formats:**
  - `[ENG-123] Add authentication`
  - `ENG-123: Fix login bug`
  - `feat: add auth (ENG-123)`

Extract the **team prefix** (e.g., `ENG`) and the **issue number** (e.g., `123`):

1. Use the team prefix to identify the Linear team
2. Use the full identifier (e.g., `ENG-123`) to look up the parent issue in Linear
3. If found, place the new ticket in the **same project** as the parent issue
4. If the parent issue has no project, use the team default

#### Fallback resolution

If auto-detection fails or no `project_hint` is provided:

1. If there is only one team, use it automatically
2. If there are multiple teams, ask the user which team to use
3. Ask the user if they want to assign it to a specific project

#### Confirm with the user

Before creating the ticket, briefly confirm the resolved team/project:

> I'll create this ticket under **[Team Name]** in project **[Project Name]** (based on `ENG-123` from the branch name). Sound good?

Proceed if confirmed, or let the user override.

### Step 3: Create the ticket

1. Create the Linear issue with the resolved team, project, title, description, labels, and priority
2. If a parent issue was identified from the `project_hint`, link the new ticket as a **related issue**
3. Append the following footer to the description:

```
   ---
   > 🤖 **AI-Generated Ticket** — This ticket was created using AI assistance. Please review and adjust priority, assignee, and details as needed.
```

### Step 4: Confirm creation

1. Report to the user:
   - The ticket identifier, title, and URL
   - The team and project it was assigned to
   - Any linked parent issue
   - Any labels applied
2. If creation fails, show the error and suggest checking Linear MCP connectivity
