# dependabot-review

> Analyzes open Dependabot alerts and assesses their actual exploitability in the codebase before recommending action.

## What it does

Fetches all open Dependabot alerts for the repository via the GitHub API, then for each alerted package determines whether it is a direct or transitive dependency, whether it is production or dev-only, and whether the specific vulnerable code path is actually reachable given how the package is used in `src/`. It cross-references the vulnerability's stated preconditions against real import patterns and user-controlled input flows. The result is a prioritized impact table and grouped recommendations that distinguish alerts safe to dismiss from those requiring immediate patching — including the exact `npm install` command needed for each fix.

## How to use it

Invoke via slash command: `/dependabot-review`

No arguments needed — the skill auto-detects the current repository owner and name from the git remote. It uses `gh api`, `npm ls`, `npm outdated`, and code search internally. The skill is read-only and makes no code changes; it produces a report only.

## Why it's useful

Raw Dependabot severity ratings routinely overstate risk for transitive or dev-only dependencies, leading teams to either panic-patch or ignore everything. This skill cuts through alert fatigue by grounding each finding in actual code usage, so engineers spend time on vulnerabilities that genuinely matter and can confidently deprioritize the rest.
