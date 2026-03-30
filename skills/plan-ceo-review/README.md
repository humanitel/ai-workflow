# plan-ceo-review

> CEO/founder-mode plan review that challenges premises, maps the 10-star product vision, and stress-tests a plan across architecture, security, edge cases, observability, and deployment with maximum rigor.

## What it does

`plan-ceo-review` opens with a full system audit (recent git history, in-flight changes, existing TODOs, architecture docs) before touching the plan. It then runs a nuclear scope challenge that questions whether the right problem is being solved, maps the current-state-to-ideal-state trajectory, and asks the user to select one of three review modes: SCOPE EXPANSION (dream bigger), HOLD SCOPE (make it bulletproof), or SCOPE REDUCTION (cut to the minimum viable version). After mode selection the skill walks through ten structured review sections — architecture, error/rescue mapping, security threat modeling, data flow edge cases, code quality, test coverage, performance, observability, deployment/rollout, and long-term trajectory — pausing after each section to resolve issues interactively with opinionated, lettered recommendations. It produces mandatory ASCII diagrams, a complete error/rescue registry, a failure modes registry, and a TODOS.md update proposal.

## How to use it

Invoke via the `/plan-ceo-review` slash command, then point it at your plan file or paste the plan content:

```
/plan-ceo-review
```

The skill will run its system audit automatically, present Step 0 findings, and ask you to choose a mode before proceeding. No arguments are required up front — the interactive flow handles everything.

## Why it's useful

Engineering reviews often optimize for correctness within the stated scope; CEO-mode review asks whether the scope itself is right. The three-mode structure means the same skill works for greenfield features (EXPANSION), bug fixes (HOLD SCOPE), and overbuilt plans (REDUCTION) without changing your workflow. The mandatory error/rescue registry and failure modes registry surface silent failure paths that are routinely missed in standard reviews, reducing production incidents from undiscovered edge cases.
