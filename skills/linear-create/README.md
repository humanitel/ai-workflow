# linear-create

> Creates a well-formed Linear ticket via the Linear MCP server, automatically inferring team and project from branch names, PR titles, or commit messages.

## What it does

`linear-create` connects to the Linear MCP server and creates a new issue with the supplied title and description. It attempts to auto-detect the correct team and project by parsing a `project_hint` — a branch name, PR title, or commit message — for Linear issue identifiers like `ENG-123`. When a parent issue is found, the new ticket is linked as a related issue and placed in the same project. If Linear is unavailable or the team cannot be resolved, the skill fails gracefully and tells the user what to fix. Every AI-generated ticket gets a standard footer marking it as AI-assisted so humans know to review it.

## How to use it

This skill is typically called by other skills (e.g., `pr-review`) but can also be invoked directly. Provide the following context:

- **title** (required) — The ticket title
- **description** (required) — Markdown-formatted body
- **project_hint** (optional) — Branch name, PR title, or commit message to infer team/project
- **labels**, **priority**, **source** (optional)

Example (standalone invocation):
```
/linear-create
title: "Fix null reference in PaymentService"
description: "PaymentService#charge raises NoMethodError when user has no payment method on file..."
project_hint: "fix/ENG-456-payment-null-ref"
priority: high
```

The skill confirms the resolved team and project before creating the ticket.

## Why it's useful

Manually creating Linear tickets mid-review breaks flow and often results in under-specified issues. `linear-create` automates the ticket from context already present in the git branch or PR, ensuring traceability between code and project management without extra copy-pasting. The confirmation step and AI-generated footer keep humans in the loop while removing the mechanical work of opening Linear, picking a team, and writing a description from scratch.
