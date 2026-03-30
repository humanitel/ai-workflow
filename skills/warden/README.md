# warden

> Run Warden to analyze code changes before committing.

## What it does

Warden is a pre-commit code analysis tool that inspects uncommitted changes (or a specified file set / git ref range) against a configurable set of skills defined in `warden.toml`. It reports findings by severity (high, medium, low), supports auto-applying suggested fixes with `--fix`, and integrates into CI via exit codes. Reference documents bundled with the skill cover the CLI, configuration schema, and how to author custom skills.

## How to use it

Trigger this skill by asking Claude to "run warden", "check my changes", "review before commit", or any Warden-related task. Common invocations:

```bash
# Analyze all uncommitted changes
warden

# Run a specific skill
warden --skill <skill-name>

# Analyze specific files
warden src/auth.ts src/database.ts

# Analyze a git range
warden main..HEAD

# Auto-apply suggested fixes
warden --fix

# Fail CI on high-severity findings
warden --fail-on high
```

Set `WARDEN_ANTHROPIC_API_KEY` or authenticate via `claude login` before running.

## Why it's useful

Warden catches bugs, security issues, and code quality problems before they reach a PR, reducing review cycles and the risk of merging regressions. Because it runs against uncommitted diffs rather than the full codebase, it stays fast and focused on exactly what changed. The structured severity levels and exit codes make it easy to enforce standards in both local workflows and CI pipelines.
