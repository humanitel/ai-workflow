# agent-prompt

> Reference guide for writing effective agent prompts and Warden skills.

## What it does

This skill activates a prompt engineering specialist persona that helps users design, review, and improve agent prompts and Warden skill files. It references a curated library of documents covering core principles, skill file structure, Warden's system-prompt architecture, structured JSON output design, agentic tool-use patterns, common anti-patterns, Claude 4.x model guidance, and context delivery research. The skill reads only the documents relevant to the user's specific question before answering.

## How to use it

Trigger by asking Claude to create a new skill, review prompt quality, or understand Warden's prompt architecture. The skill uses `Read`, `Grep`, and `Glob` tools to pull in reference material. Skill files are expected at `.agents/skills/{name}/SKILL.md` and must include at minimum:

```markdown
---
name: skill-name
description: One-line description for discovery.
allowed-tools: Read Grep Glob
---

[Role statement]

## Your Task
[What to analyze and criteria to apply]

## Severity Levels
[Definitions tied to impact]
```

## Why it's useful

Prompt quality directly determines how reliably an agent performs its task — poorly structured prompts produce inconsistent or incomplete results. Having a specialist skill that draws on battle-tested reference material means guidance is grounded in documented best practices rather than ad hoc intuition. It also enforces a consistent skill structure across a team, making skills easier to maintain, discover, and extend.
