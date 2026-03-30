# judge-plan

> Independent plan and spec reviewer that evaluates a plan file against the workspace and returns a structured verdict with findings.

## What it does

`judge-plan` acts as an impartial reviewer of plans, proposals, or technical specs written by another agent or human. It reads the target plan file, cross-checks claims against the actual codebase using Glob, Grep, and Read, and then scores the plan across five dimensions: repo fit, technical correctness, scope and sequencing, evaluation rigor, and safety/operations. Results are reported as a structured verdict with a confidence bar, strengths, blocking findings, non-blocking findings, and open questions. When an external judge runner (e.g., Codex) is configured, it delegates to that runner instead of running inline.

## How to use it

Invoke via the `/judge-plan` slash command with an optional focus mode and the path to the plan file:

```
/judge-plan [--focus balanced|architecture|evaluation|product|operations|safety] <plan-file> [supporting-files...]
```

Examples:
```
/judge-plan docs/plan.md
/judge-plan --focus architecture docs/plan.md src/services/payment.rb
/judge-plan --focus safety plans/llm-integration.md
```

If no plan file is provided, the skill will ask for one. Focus defaults to `balanced` when omitted.

## Why it's useful

Having the same agent that wrote a plan also review it creates a blind spot — `judge-plan` breaks that loop by enforcing an independent, adversarial reading before implementation begins. The structured verdict (approve / approve_with_changes / request_major_revision) gives a clear go/no-go signal, while the focus modes let you concentrate scrutiny on the dimensions that matter most for a given change. Catching blocking issues in a plan is far cheaper than finding them mid-implementation.
