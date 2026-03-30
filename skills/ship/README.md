# ship

> Fully automated end-to-end ship workflow: merge main, run tests, review diff, bump version, update CHANGELOG, commit, push, and open a PR.

## What it does

The `ship` skill runs a non-interactive pipeline that takes a feature branch from "ready to ship" to "PR open" without any prompts. It merges `origin/main`, runs Rails and Vitest test suites in parallel, conditionally runs LLM eval suites when prompt-related files are changed, performs a pre-landing structural review via the `review` checklist, auto-selects a MICRO or PATCH version bump based on diff size, generates a CHANGELOG entry from the commit history, splits staged changes into bisectable commits, pushes the branch, and creates a GitHub PR with a structured body. It stops only for test failures, complex merge conflicts, unresolvable critical review findings, or MINOR/MAJOR version bumps that require human judgment.

## How to use it

Invoke with `/ship` from any feature branch. The workflow runs straight through and outputs the PR URL as its final line.

```
/ship
```

The only decisions you may be asked to make:
- MINOR or MAJOR version bump confirmation (if the diff warrants it)
- How to handle each CRITICAL pre-landing review finding (fix now / acknowledge / false positive)

Do not run from `main` — the skill will abort.

## Why it's useful

Shipping a PR correctly involves a tedious, error-prone sequence of steps that most engineers abbreviate or skip under time pressure. `/ship` enforces the full protocol every time — tests, evals, review, versioning, CHANGELOG, and bisectable commits — so that quality gates are never skipped and the PR body always contains the context reviewers need. The result is consistent release hygiene with zero manual ceremony.
