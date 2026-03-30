# architecture-review

> Staff-level codebase health review that surfaces structural issues before they compound.

## What it does

Performs a comprehensive architectural audit across five dimensions: module complexity (file size, responsibility count, fan-out), silent failure patterns (swallowed errors, empty returns, missing error handlers), TypeScript type safety gaps (unsafe casts, regex assumptions, generic index access), test coverage holes (untested critical paths, integration gaps), and LLM-friendliness (JSDoc coverage, naming clarity, actionable error messages). It works macro-first — focusing on systemic issues that grow worse over time rather than individual style preferences. Each finding is backed by specific file names, line numbers, and concrete recommendations.

## How to use it

Invoke via slash command: `/architecture-review`

The skill auto-navigates the codebase: it reads entry points to map module structure, identifies the largest files, traces error paths, audits type assertions, maps test coverage, and samples public API documentation. No arguments are required — point it at any TypeScript/Node.js project.

Output is structured as an executive summary followed by prioritized findings (critical → high → medium → low), each with problem description, evidence, and a recommended fix. A "What's Working Well" section is always included.

## Why it's useful

Large codebases accumulate technical debt invisibly — a monolithic module here, a swallowed error there — until reliability degrades in production. Running this review periodically catches structural rot early and produces a concrete, prioritized backlog rather than vague concerns. It also explicitly flags LLM-unfriendly patterns, making the codebase easier to maintain with AI assistance over time.
