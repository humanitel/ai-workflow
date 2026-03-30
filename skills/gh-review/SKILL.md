---
name: fuse-gh-review
description: Submits a GitHub PR review using the gh CLI based on the output of the pr-review skill
---

# GitHub PR Review Submission

## Prerequisites

- The `fuse-review-pr` skill MUST be run first to generate the review analysis
- The `gh` CLI must be authenticated (`gh auth status`)
- You must be in a git repository with a valid remote

## Workflow

### Step 1: Identify the PR

1. Determine the current PR number from context. Try in order:
   - Check if a PR number was provided by the user
   - Use the current branch: `gh pr view --json number -q .number`
2. If no PR can be identified, ask the user for the PR number
3. Gather PR metadata for later use:

```bash
   gh pr view <PR_NUMBER> --json number,title,headRefName,url,commits
```

Store the **PR number**, **PR title**, **branch name**, **PR URL**, and **latest commit messages** — these will be passed to `fuse-linear-ticket` if needed.

### Step 2: Gather the review from `fuse-review-pr`

1. If `fuse-review-pr` has not been run yet, run it first and wait for the results
2. Collect all findings from the review, categorized by severity and deferability:
   - **Blocking** — issues that must be fixed before merge (always `in-pr`)
   - **Suggestions** — improvements that should be considered (may be `in-pr` or `follow-up`)
   - **Nits** — minor style or preference items (may be `in-pr` or `follow-up`)

   Mark a finding as `follow-up` when it is:
   - Not directly related to the PR's stated intent
   - A refactoring or code quality improvement
   - A performance optimization that isn't urgent
   - Tech debt surfaced during review
   - Nice-to-have test coverage or documentation additions

### Step 3: Determine the review action

Based on the findings from `fuse-review-pr`, decide the appropriate action:

| Condition                            | Action            |
| ------------------------------------ | ----------------- |
| Any **blocking** issues found        | `REQUEST_CHANGES` |
| Only **suggestions** and/or **nits** | `COMMENT`         |
| No issues found                      | `APPROVE`         |

### Step 4: Format the review body

Structure the review body in markdown like this:

```
## PR Review Summary

### 🔴 Blocking Issues
<!-- List each blocking issue with file path and line reference if possible -->
- **[file:line]** Description of the issue and why it blocks

### 🟡 Suggestions
<!-- List each suggestion -->
- **[file:line]** Description of the suggested improvement

### 🟢 Nits
<!-- List each nit -->
- **[file:line]** Minor item

### ✅ What looks good
<!-- Briefly note things that were done well — be constructive -->

---
> 🤖 **AI-Generated Review** — This review was generated using AI assistance via `fuse-review-pr`. Findings should be validated by a human reviewer before acting on them. AI reviews may contain false positives or miss context-specific nuances.
```

Omit any section that has no items (don't include empty headings). The AI disclosure footer MUST always be included — it is never omitted.

Only include `in-pr` items in the review body. Do NOT include `follow-up` items in the GitHub review — these are handled separately in Step 7.

### Step 5: Submit the review

Run the appropriate `gh` command:

```bash
# For REQUEST_CHANGES:
gh pr review <PR_NUMBER> --request-changes --body "<REVIEW_BODY>"

# For COMMENT:
gh pr review <PR_NUMBER> --comment --body "<REVIEW_BODY>"

# For APPROVE:
gh pr review <PR_NUMBER> --approve --body "<REVIEW_BODY>"
```

### Step 6: Confirm submission

1. Report to the user:
   - The review action taken (approved, commented, or requested changes)
   - A count of findings by severity
   - The PR number and URL
2. If the `gh` command fails, show the error and suggest:
   - Checking `gh auth status`
   - Verifying the PR number exists
   - Ensuring the user has permission to review the PR

### Step 7: Offer follow-up ticket for deferrable items

After submitting the review, check if any findings were marked as `follow-up`.

If `follow-up` items exist:

1. Present them to the user:

   > I found **N item(s)** that could be tracked as separate follow-up work instead of blocking this PR:
   >
   > 1. **[brief description]** — [file:line]
   > 2. **[brief description]** — [file:line]
   >
   > Would you like me to create a Linear ticket for these?

2. **If the user agrees**, invoke the `fuse-linear-ticket` skill with:
   - **title:** `Follow-up: Code review suggestions from PR #<NUMBER>`
   - **description:** Markdown body containing:
     - A link to the PR URL
     - Each follow-up item with file path, line reference, and description
   - **source:** `PR review`
   - **priority:** `low`
   - **project_hint:** The **branch name** from Step 1 (e.g., `feat/ENG-123-add-auth`). If no Linear identifier is found in the branch, fall back to the **PR title**, then the **latest commit message**.

3. **If the user declines** or `fuse-linear-ticket` reports that Linear is unavailable, acknowledge and end the workflow.

4. **If there are no `follow-up` items**, skip this step entirely — do not ask the user about ticket creation.
