# plan-eng-review

> Engineering-manager-mode plan review that locks in the execution plan through interactive, section-by-section analysis of architecture, code quality, tests, and performance.

## What it does

`plan-eng-review` starts with a scope challenge (Step 0) that checks whether existing code already solves sub-problems, flags unnecessary complexity, and asks you to choose a review mode: SCOPE REDUCTION, BIG CHANGE (full interactive review section by section), or SMALL CHANGE (compressed single-pass). Once a mode is chosen, the skill commits to it and walks through four review sections — architecture, code quality, test coverage, and performance — raising one issue per `AskUserQuestion` call with concrete options, a clear recommendation, and the reasoning mapped to explicit engineering preferences (DRY, minimal diff, explicit over clever, edge cases over speed). It also produces a test diagram, failure modes list, and a TODOS.md update proposal at the end.

## How to use it

Invoke via the `/plan-eng-review` slash command and supply or paste your plan:

```
/plan-eng-review
```

The skill self-directs from there. At Step 0 it will ask which review mode you want, then proceed section by section, pausing for your input before moving on.

## Why it's useful

This skill replaces the common pattern of getting a wall of review comments all at once — instead it surfaces one decision at a time, forces prioritization, and records unresolved decisions explicitly rather than silently defaulting. The retrospective check against git history makes it more aggressive in areas that have already been problematic on the branch, turning recurring issues into visible architectural signals. The required test diagram ensures test coverage is planned, not assumed.
