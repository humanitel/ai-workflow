# pr-review

> Structured pull request review covering correctness, security, code quality, architecture, performance, testing, and observability across 25 explicit checkpoints.

## What it does

`pr-review` applies a consistent, comprehensive checklist to any PR diff. It verifies the code matches the PR description's intent, hunts for edge cases (null values, boundary conditions, race conditions), checks for security issues (injection, improper auth, secrets in code), evaluates code quality (complexity, duplication, naming, dead code), assesses architectural fit with existing patterns, flags N+1 queries and unbounded operations, confirms new logic has meaningful tests, and checks that logging and migrations are production-safe. Every finding is categorized as a blocking issue, suggestion, or nit, so reviewers and authors can quickly distinguish what must change from what is advisory.

## How to use it

Invoke via the `/pr-review` slash command. The skill will review the current PR or the diff in context:

```
/pr-review
```

No arguments are required. The skill reads the available diff and produces a structured review organized by the seven categories (Correctness, Security, Code Quality, Architecture, Performance, Testing, Observability).

## Why it's useful

Manual PR reviews are inconsistent — different reviewers catch different things, and important categories like observability or migration safety are frequently skipped under time pressure. `pr-review` guarantees a full sweep of 25 checkpoints every time, reducing the chance that a security issue or silent failure mode ships unnoticed. The blocking / suggestion / nit distinction keeps review conversations focused on what actually blocks merge.
