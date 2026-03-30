# qa

> Systematically QA test a web application and produce a structured report with health score, screenshots, and reproduction steps.

## What it does

The `qa` skill acts as a QA engineer that tests web applications like a real user — clicking every element, filling every form, and checking every state. It supports three modes: **full** (systematic exploration of every reachable page), **quick** (30-second smoke test of the homepage and top 5 navigation targets), and **regression** (compares a full run against a saved baseline to surface new and fixed issues). All findings are written incrementally to a structured Markdown report with annotated screenshots saved under `.gstack/qa-reports/`. A weighted health score (0–100) is computed across categories including Console, Functional, UX, Accessibility, and more.

## How to use it

Invoke by asking Claude to "qa", "QA", "test this site", "find bugs", or "dogfood". Provide a target URL; all other parameters are optional.

```
qa https://myapp.com
qa http://localhost:3000 --quick
qa https://myapp.com --regression .gstack/qa-reports/baseline.json
qa https://myapp.com Focus on the billing page
qa https://myapp.com Sign in to user@example.com
```

Parameters:
- **Target URL** — required
- **Mode** — `--quick` or `--regression <baseline.json>` (default: full)
- **Output dir** — default `.gstack/qa-reports/`
- **Scope** — optional focus area (e.g. "Focus on the checkout flow")
- **Auth** — credentials or a cookie file for authenticated testing

## Why it's useful

Manual QA is time-consuming and easy to skip under deadline pressure; this skill automates the systematic exploration and produces verifiable, screenshot-backed evidence for every issue. The health score and baseline regression mode make it easy to track quality over time and catch regressions before they reach production. Framework-specific guidance (Next.js, Rails, WordPress, SPAs) ensures the right failure modes are checked for each stack.
