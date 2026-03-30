# browse

> **External packaged skill** — part of the gstack plugin

`browse` is a persistent headless Chromium skill for QA testing and site dogfooding. It ships as a compiled binary inside the gstack package.

## What it does

Navigate any URL, interact with elements, verify page state, diff before/after actions, take annotated screenshots, check responsive layouts, test forms and uploads, handle dialogs, and assert element states. ~100ms per command after a ~3s cold start.

## Official source

**GitHub:** https://github.com/garrytan/gstack

## Installation

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

## Related skills

- [gstack](./gstack.md) — the parent package
- [qa](./qa.md) — full systematic QA test pass (uses browse under the hood)
- [setup-browser-cookies](./setup-browser-cookies.md) — import browser cookies into headless sessions
