# retro

> Generate a comprehensive weekly engineering retrospective from git history, with per-person analysis, trend tracking, and persistent snapshots.

## What it does

The `retro` skill analyzes `origin/main` commit history to produce a structured engineering retrospective covering shipping velocity, work session patterns, commit type breakdowns, hotspot analysis, and PR size distribution. It is team-aware: it identifies who is running the command (via `git config user.name`) and provides a deep personal analysis alongside praise and growth feedback for every other contributor. Results are output directly to the conversation, and a JSON snapshot is saved to `.context/retros/` for trend tracking — subsequent retros automatically compare against the prior snapshot and display deltas. All timestamps are reported in Pacific time.

## How to use it

Invoke with `/retro` followed by an optional time window or `compare` flag.

```
/retro                 — last 7 days (default)
/retro 24h             — last 24 hours
/retro 14d             — last 14 days
/retro 30d             — last 30 days
/retro compare         — compare current 7-day window vs prior 7-day window
/retro compare 14d     — compare with an explicit 14-day window
```

## Why it's useful

It turns raw git history into an actionable narrative — surfacing late-night coding clusters, fix-chain patterns, churn hotspots, and test coverage gaps that are invisible in day-to-day work. The per-teammate sections give engineering leads concrete, commit-anchored talking points for 1:1s. Persistent JSON snapshots mean every retro builds on the last, turning a one-off review into a running record of team health and velocity trends.
