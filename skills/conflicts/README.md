# conflicts

> Resolves Git merge conflicts in TypeScript/Node.js projects with semantic intent analysis and ironclad safety nets.

## What it does

Treats merge conflicts as intent problems rather than text problems: before touching a single line, it identifies what each author meant to accomplish, then finds a resolution that preserves both intents. The workflow follows a strict six-step sequence — Snapshot, Diagnose, Classify, Resolve, Verify, Complete — with automated scripts for each phase. Every conflict is categorized by type (parallel edits, divergent refactors, structural reorganization, dependency files, or formatting), and each type gets a dedicated resolution strategy. A post-resolution change audit compares the final file against both sides of the merge to guarantee no work is silently dropped.

## How to use it

Invoke via slash command: `/conflicts`

Trigger it whenever you hit merge conflicts during a `git merge`, `git rebase`, or `git cherry-pick`. The skill handles all common scenarios including rebases across many commits, monorepo package conflicts, binary file conflicts, submodule pointer conflicts, and large-scale rename/move conflicts. It provides ready-to-run bash scripts for diagnosis and auditing, and always creates a backup branch before making any changes so recovery is a single command.

## Why it's useful

Merge conflicts are one of the most common sources of accidentally dropped work during fast-moving development. This skill eliminates that risk by enforcing a structured process with explicit safety checkpoints at every step, so you finish with code that compiles, tests that pass, and confidence that no change from either branch was lost without deliberate justification.
