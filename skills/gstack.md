# gstack

> **External packaged skill** — requires separate installation

gstack turns Claude Code from one generic assistant into a team of specialists you can summon on demand. Eight opinionated workflow skills for Claude Code.

## What it includes

| Skill | Description |
|-------|-------------|
| `/plan-ceo-review` | Rethink the problem — find the 10-star product hiding inside the request |
| `/plan-eng-review` | Architecture, data flow, edge cases, scaling |
| `/review` | Pre-landing diff analysis |
| `/ship` | Full shipping workflow: sync, test, changelog, push, PR |
| `/browse` | Persistent headless Chromium for testing and dogfooding |
| `/qa` | Systematic QA with health scores and screenshots |
| `/setup-browser-cookies` | Import browser cookies into headless sessions |
| `/retro` | Weekly retrospective with git history analysis |

## Official source

**GitHub:** https://github.com/garrytan/gstack

## Installation

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

Then add to your project's `CLAUDE.md`:
```
Use the /browse skill from gstack for all web browsing.
Available skills: /plan-ceo-review, /plan-eng-review, /review, /ship, /browse, /qa, /setup-browser-cookies, /retro
```
