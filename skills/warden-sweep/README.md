# warden-sweep

> Full-repository code sweep: scan every file, verify findings with deep tracing, and create draft PRs for validated issues.

## What it does

Warden Sweep is a five-phase automated pipeline that runs Warden across every file in a repository, not just uncommitted changes. Phase 1 (Scan) enumerates all files and runs Warden per file, collecting raw findings. Phase 2 (Verify) launches parallel subagent tasks to deep-trace each finding and qualify or disqualify it. Phases 3-5 create a GitHub tracking issue, open draft PRs with fixes for validated findings, and produce a final summary report. All phases are incremental — a sweep can be resumed from any point using the manifest in `.warden/sweeps/<run-id>/data/manifest.json`.

**Requires**: `warden`, `gh`, `git`, `jq`, `uv`

## How to use it

Invoke by asking Claude to "sweep the repo", "scan everything", "find all bugs", "full codebase review", or "run warden across the entire repository". The skill is driven by bundled Python scripts located at `${CLAUDE_SKILL_ROOT}/scripts/`:

```bash
# Phase 1: Scan all files
uv run ${CLAUDE_SKILL_ROOT}/scripts/scan.py

# Resume a partial scan
uv run ${CLAUDE_SKILL_ROOT}/scripts/scan.py --sweep-dir .warden/sweeps/<run-id>

# Phase 3: Create tracking issue
uv run ${CLAUDE_SKILL_ROOT}/scripts/create_issue.py <sweep-dir>

# Phase 5: Organize and generate summary report
uv run ${CLAUDE_SKILL_ROOT}/scripts/organize.py <sweep-dir>
```

Phases 2 and 4 (Verify and Patch) are orchestrated by the Claude agent using subagents, not direct script calls.

## Why it's useful

A targeted pre-commit check only sees what changed; a sweep finds latent bugs and security vulnerabilities across the entire codebase that would otherwise go undetected. Automated verification with deep tracing dramatically cuts false-positive noise, so every draft PR that comes out of a sweep represents a real, confirmed issue. The deduplication logic in Phase 4 prevents duplicate PRs when the same code region has already been addressed, keeping the review queue clean.
