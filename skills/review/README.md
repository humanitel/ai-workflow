# review

> Run a pre-landing structural review of the current branch's diff against main, catching issues that tests don't cover.

## What it does

The `review` skill performs a two-pass analysis of `git diff origin/main` against a curated checklist stored at `.claude/skills/review/checklist.md`. Pass 1 (CRITICAL) targets SQL and data safety issues and LLM output trust boundary violations. Pass 2 (INFORMATIONAL) covers conditional side effects, magic numbers, string coupling, dead code, LLM prompt issues, test gaps, and frontend concerns. For each critical finding, it prompts the user to fix it now, acknowledge it, or mark it as a false positive — and applies any chosen fixes automatically. The skill is read-only by default and never commits, pushes, or creates PRs.

## How to use it

Invoke with `/review` from any feature branch. The skill will abort with an informational message if run on `main` or if there is no diff against main.

```
/review
```

The checklist file at `.claude/skills/review/checklist.md` must be present; the skill stops and reports an error if it cannot be read.

## Why it's useful

Automated tests validate behavior but rarely catch structural patterns like unsafe raw SQL interpolation, LLM outputs used directly in privileged operations, or orphaned code paths — the kind of issues that slip through review and cause incidents. Running `/review` before every merge adds a fast, consistent safety net without requiring a human reviewer to remember what to look for.
