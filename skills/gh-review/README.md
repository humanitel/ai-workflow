# gh-review

> Submits a structured GitHub PR review via the `gh` CLI, with optional follow-up Linear ticket creation for deferrable findings.

## What it does

Takes the output of a prior `fuse-review-pr` analysis and submits it as a formal GitHub PR review using the appropriate action: `APPROVE` (no issues), `COMMENT` (suggestions or nits only), or `REQUEST_CHANGES` (any blocking issue). The review body is formatted as a markdown summary with color-coded sections for blocking issues, suggestions, and nits, plus a mandatory AI disclosure footer. Findings are split into `in-pr` items (included in the GitHub review) and `follow-up` items (deferred tech debt). After submission, if follow-up items exist, the skill offers to create a Linear ticket linking back to the PR with all deferred findings tracked at low priority.

## How to use it

Invoke via slash command: `/gh-review`

Prerequisites: `fuse-review-pr` must have been run first to generate the review analysis, and `gh` must be authenticated (`gh auth status`). The skill auto-detects the current PR from the active branch; if no PR is found, it asks for a PR number. The Linear ticket step is opt-in — the skill asks before creating anything.

## Why it's useful

Manually formatting and submitting PR reviews is repetitive and easy to do inconsistently. This skill standardizes the review format, automates the `gh` submission, and closes the loop on deferrable findings by routing them into Linear rather than letting them disappear into review comment threads — keeping the PR focused on what actually needs to change now.
