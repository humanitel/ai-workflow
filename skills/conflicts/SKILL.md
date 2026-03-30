# Merge Conflict Resolver (TypeScript / Node.js)

Resolve Git merge conflicts in TypeScript and Node.js projects the way a Staff+ engineer at a top-tier company would: with semantic understanding of what both sides intended, a structured decision framework, and ironclad safety nets to prevent losing anyone's work.

---

## Philosophy

Most merge conflict tools treat conflicts as text problems. They diff lines and ask you to pick a side. A senior engineer treats conflicts as **intent problems**: what did each author mean to accomplish, and how do we preserve both intents in the final result?

Three principles guide every resolution:

1. **Understand before resolving.** Read both sides. Understand the _why_ behind each change before touching a single line.
2. **Never lose work.** Create safety snapshots before every resolution. Every change either side made must be accounted for in the final result — kept, intentionally superseded, or explicitly noted as dropped with justification.
3. **Verify after resolving.** A conflict isn't resolved until the code compiles, tests pass, and the semantic intent of both sides is preserved.

---

## Workflow Overview

When you need help with merge conflicts, follow this sequence:

```
1. SNAPSHOT  →  Create a safety backup before touching anything
2. DIAGNOSE  →  Understand the scope and nature of all conflicts
3. CLASSIFY  →  Categorize each conflict by type and complexity
4. RESOLVE   →  Apply the right strategy per conflict type
5. VERIFY    →  Confirm nothing was lost and code is correct
6. COMPLETE  →  Finish the merge/rebase and clean up
```

Each step is detailed below. Do not skip steps. The safety guarantees depend on following the full sequence.

---

## Step 1: SNAPSHOT — Create a Safety Net

Before resolving anything, create an escape hatch. This is non-negotiable.

```bash
# Create a named snapshot of the current state
git stash push -m "pre-conflict-resolution-$(date +%Y%m%d-%H%M%S)" --include-untracked 2>/dev/null

# If in the middle of a merge/rebase, stash won't work — use a backup branch instead
git branch backup/pre-resolution-$(date +%Y%m%d-%H%M%S) HEAD 2>/dev/null

# Also record the exact state of both sides for reference
git log --oneline -5 HEAD > /tmp/conflict-head-log.txt 2>/dev/null
git log --oneline -5 MERGE_HEAD > /tmp/conflict-merge-head-log.txt 2>/dev/null
```

Tell the user what backup was created and how to restore it if anything goes wrong. Provide the exact restore command upfront — e.g., `git merge --abort` or `git rebase --abort` or `git checkout backup/pre-resolution-XXXXX`.

---

## Step 2: DIAGNOSE — Understand the Conflict Landscape

Get the full picture before diving into individual files.

### Quick Diagnosis Commands

```bash
# List all conflicted files
git diff --name-only --diff-filter=U

# Show the conflict summary — how many conflicts per file
git diff --check

# Understand what operation caused the conflicts
if [ -f .git/MERGE_HEAD ]; then
    echo "STATE: merge in progress"
    echo "MERGING: $(cat .git/MERGE_HEAD)"
elif [ -d .git/rebase-merge ] || [ -d .git/rebase-apply ]; then
    echo "STATE: rebase in progress"
    cat .git/rebase-merge/head-name 2>/dev/null || echo "(interactive rebase)"
elif [ -f .git/CHERRY_PICK_HEAD ]; then
    echo "STATE: cherry-pick in progress"
fi

# Show what each side changed (high-level summary)
git log --oneline --left-right HEAD...MERGE_HEAD 2>/dev/null
```

### Full Diagnosis Script

For a comprehensive diagnosis report, run the following script. It detects the operation type, lists all conflicted files, classifies each by type and complexity, creates a safety backup branch, and outputs a structured summary table.

```bash
#!/usr/bin/env bash
# diagnose-conflicts.sh — Generate a full conflict diagnosis report
set -euo pipefail

if ! git rev-parse --is-inside-work-tree &>/dev/null; then
    echo "ERROR: Not inside a git repository"
    exit 1
fi

echo "╔══════════════════════════════════════════════════════════════╗"
echo "║              MERGE CONFLICT DIAGNOSIS REPORT                ║"
echo "╚══════════════════════════════════════════════════════════════╝"
echo ""

# Detect operation type
echo "── Operation ──"
if [ -f .git/MERGE_HEAD ]; then
    echo "  Type: MERGE"
    echo "  Merging commit: $(cat .git/MERGE_HEAD)"
    echo "  Current branch: $(git branch --show-current)"
    echo "  Incoming: $(git log --oneline -1 MERGE_HEAD 2>/dev/null || echo 'unknown')"
elif [ -d .git/rebase-merge ] || [ -d .git/rebase-apply ]; then
    echo "  Type: REBASE"
    if [ -f .git/rebase-merge/head-name ]; then
        echo "  Rebasing: $(cat .git/rebase-merge/head-name)"
        TOTAL=$(cat .git/rebase-merge/end 2>/dev/null || echo '?')
        CURRENT=$(cat .git/rebase-merge/msgnum 2>/dev/null || echo '?')
        echo "  Progress: commit $CURRENT of $TOTAL"
    fi
elif [ -f .git/CHERRY_PICK_HEAD ]; then
    echo "  Type: CHERRY-PICK"
    echo "  Cherry-picking: $(git log --oneline -1 CHERRY_PICK_HEAD 2>/dev/null || echo 'unknown')"
else
    echo "  Type: UNKNOWN (no merge/rebase/cherry-pick in progress)"
    echo "  (Conflicts may already be partially resolved)"
fi
echo ""

# List conflicted files
CONFLICTED=$(git diff --name-only --diff-filter=U 2>/dev/null || true)
if [ -z "$CONFLICTED" ]; then
    echo "No unresolved conflicts found."
    exit 0
fi

FILE_COUNT=$(echo "$CONFLICTED" | wc -l)
echo "── Conflicted Files: $FILE_COUNT ──"
echo ""

printf "%-50s | %-8s | %-10s | %s\n" "File" "Type" "Complexity" "Recommendation"
printf "%-50s-+-%-8s-+-%-10s-+-%s\n" \
    "$(printf '%0.s-' {1..50})" "--------" "----------" "$(printf '%0.s-' {1..25})"

while IFS= read -r file; do
    MARKERS=$(grep -c '<<<<<<<' "$file" 2>/dev/null || echo "0")
    TYPE="?" ; COMPLEXITY="?" ; RECOMMENDATION="?"

    if file "$file" | grep -qv text; then
        TYPE="Binary" ; COMPLEXITY="Low" ; RECOMMENDATION="Choose version or regenerate"
    elif echo "$file" | grep -qE '\.(lock)$|lock\.json$|lock\.yaml$|\.generated\.ts$'; then
        TYPE="D (Lock)" ; COMPLEXITY="Low" ; RECOMMENDATION="Regenerate from source"
    elif echo "$file" | grep -qE 'tsconfig.*\.json$|\.eslintrc|\.prettierrc|jest\.config|next\.config|vite\.config|\.env'; then
        TYPE="D (Config)" ; COMPLEXITY="Low" ; RECOMMENDATION="Merge both additions"
    elif echo "$file" | grep -qE 'package\.json$'; then
        TYPE="D (Deps)" ; COMPLEXITY="Low" ; RECOMMENDATION="Merge deps + npm install"
    else
        if [ "$MARKERS" -le 1 ]; then
            OURS_SIZE=$(git show ":2:$file" 2>/dev/null | wc -l || echo "0")
            THEIRS_SIZE=$(git show ":3:$file" 2>/dev/null | wc -l || echo "0")
            if [ "$OURS_SIZE" -eq 0 ] || [ "$THEIRS_SIZE" -eq 0 ]; then
                TYPE="C (Move)" ; COMPLEXITY="Medium" ; RECOMMENDATION="Check if renamed/moved"
            else
                TYPE="A (Parallel)" ; COMPLEXITY="Low" ; RECOMMENDATION="Combine both changes"
            fi
        elif [ "$MARKERS" -le 3 ]; then
            TYPE="A/B" ; COMPLEXITY="Medium" ; RECOMMENDATION="Review intent, combine"
        else
            TYPE="B (Refactor)" ; COMPLEXITY="High" ; RECOMMENDATION="Manual review needed"
        fi
    fi

    printf "%-50s | %-8s | %-10s | %s\n" "$file" "$TYPE" "$COMPLEXITY" "$RECOMMENDATION"
done <<< "$CONFLICTED"

echo ""
echo "── Summary ──"
echo "  Total conflicted files: $FILE_COUNT"
echo "  Total conflict regions: $(grep -rch '<<<<<<<' $CONFLICTED 2>/dev/null | awk '{s+=$1}END{print s}')"
echo ""

echo "── Abort Commands (if needed) ──"
if [ -f .git/MERGE_HEAD ]; then echo "  git merge --abort"
elif [ -d .git/rebase-merge ] || [ -d .git/rebase-apply ]; then echo "  git rebase --abort"
elif [ -f .git/CHERRY_PICK_HEAD ]; then echo "  git cherry-pick --abort"
fi

echo ""
echo "── Safety Backup ──"
BACKUP_NAME="backup/pre-resolution-$(date +%Y%m%d-%H%M%S)"
echo "  Creating backup branch: $BACKUP_NAME"
git branch "$BACKUP_NAME" HEAD 2>/dev/null && echo "  Backup created successfully." \
    || echo "  (Backup branch may already exist)"
echo "  Restore with: git checkout $BACKUP_NAME"
```

Present a summary to the user including how many files are conflicted, what operation caused the conflicts, which branches are involved, and a quick characterization of scope (trivial / moderate / complex).

---

## Step 3: CLASSIFY — Categorize Each Conflict

Read the conflict markers in each file and classify them. This classification drives the resolution strategy.

### Conflict Type Taxonomy

**Type A — Parallel Edits (Non-Overlapping Intent)**
Both sides edited the same region but for different reasons. Example: one side added a parameter, the other changed a return type. Resolution: combine both changes.

**Type B — Divergent Refactors (Same Intent, Different Approach)**
Both sides tried to solve the same problem differently. Example: one side renamed a function, the other restructured the same function. Resolution: pick the better approach, but port any unique improvements from the other.

**Type C — Structural Conflicts (File-Level Reorganization)**
One side moved or renamed files while the other edited them. Or one side deleted code the other modified. Resolution: requires understanding the structural change and replaying edits in the new structure.

**Type D — Dependency Conflicts (Import / Config / Lock Files)**
Package.json, tsconfig.json, import statements, or generated files. Resolution: usually combine both additions; regenerate lock files.

**Type E — Trivial / Formatting**
Whitespace, line endings, import ordering, auto-formatted code. Resolution: accept either side, then re-run formatter.

For each conflicted file, run:

```bash
# Show the conflict with full context (not just the markers)
git diff --merge HEAD -- <file>

# Show what each side independently changed vs the common ancestor
git diff :1:<file> :2:<file>   # base vs ours
git diff :1:<file> :3:<file>   # base vs theirs
```

Stage versions `:1:`, `:2:`, `:3:` represent base, ours, and theirs respectively. Comparing each side against base (not against each other) reveals the **independent intent** of each change.

Present the classification as a table:

```
File                        | Type | Complexity | Recommendation
----------------------------|------|------------|------------------
src/auth/login.ts           |  A   | Low        | Combine both
src/api/handler.ts          |  B   | High       | Manual review needed
package.json                |  D   | Low        | Merge deps + regenerate lock
src/utils/helpers.ts        |  E   | Trivial    | Accept theirs + format
```

---

## Step 4: RESOLVE — Apply the Right Strategy

### Resolution Decision Framework

For each conflict, follow this decision tree:

```
Is the conflict in a generated/lock file?
  → YES: Accept either side, regenerate the file
  → NO: Continue

Do both sides have the same intent (fixing the same bug, adding the same feature)?
  → YES: Pick the more complete/correct implementation. Port any unique
          improvements from the other side. (Type B)
  → NO: Continue

Are the changes logically independent (different concerns in the same region)?
  → YES: Combine both changes, preserving the intent of each. (Type A)
  → NO: Continue

Did one side delete/move code that the other modified?
  → YES: Understand the structural change. Replay the modifications
          in the new location/structure. (Type C)
  → NO: Requires careful manual resolution.
```

### Resolution Techniques

**Combine (Type A):** Merge both changes into the region. Read the base version first to understand what was originally there, then apply both modifications.

```bash
# View the three versions side by side
git show :1:<file> > /tmp/base-version
git show :2:<file> > /tmp/ours-version
git show :3:<file> > /tmp/theirs-version

# Use a 3-way comparison to understand all changes
diff3 /tmp/ours-version /tmp/base-version /tmp/theirs-version
```

**Select + Port (Type B):** Choose the primary implementation, then carefully review the discarded side for any improvements that should be ported over.

**Structural Replay (Type C):** Identify the structural change (rename, move, delete), then manually apply the other side's edits in the new structure.

**Regenerate (Type D):** For dependency files, merge the dependency declarations in package.json by including additions from both sides, then regenerate the lock file:

```bash
# npm
npm install
# yarn
yarn install
# pnpm
pnpm install
```

**Format + Accept (Type E):** Accept either version, then run the project's formatter (typically Prettier or ESLint --fix).

### Per-File Resolution Process

For each conflicted file:

1. Open the file and locate all conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
2. For each conflict region:
   a. Read the base version (what was there before either change)
   b. Read "ours" (the current branch's change) and understand its intent
   c. Read "theirs" (the incoming branch's change) and understand its intent
   d. Apply the resolution strategy from the decision framework
   e. Remove the conflict markers
3. After resolving all markers in the file, do a **change audit** (see below)
4. Stage the file: `git add <file>`

### Change Audit (Critical — Do Not Skip)

After resolving each file, verify no changes were silently dropped. This is the key safety mechanism that prevents losing work.

```bash
# Compare our resolution against what each side intended
# Check: did we preserve everything "ours" added?
diff <(git diff :1:<file> :2:<file>) <(git diff :1:<file> <file>)

# Check: did we preserve everything "theirs" added?
diff <(git diff :1:<file> :3:<file>) <(git diff :1:<file> <file>)
```

Every line that was added, modified, or removed by either side must be:
- **Present** in the final resolution, OR
- **Explicitly superseded** by the other side's better version, OR
- **Documented** as intentionally dropped (with a reason)

If any change from either side is missing without justification, flag it before continuing.

### Full Change Audit Script

For a thorough audit of a resolved file, use this script:

```bash
#!/usr/bin/env bash
# change-audit.sh — Verify no changes were silently dropped during conflict resolution
# Usage: bash change-audit.sh <file>
set -euo pipefail

FILE="${1:?Usage: change-audit.sh <file>}"

if ! git rev-parse --is-inside-work-tree &>/dev/null; then
    echo "ERROR: Not inside a git repository"
    exit 1
fi

BASE=$(git show ":1:$FILE" 2>/dev/null) || { echo "SKIP: No base version (new file on one side)"; exit 0; }
OURS=$(git show ":2:$FILE" 2>/dev/null) || OURS=""
THEIRS=$(git show ":3:$FILE" 2>/dev/null) || THEIRS=""
RESOLVED=$(cat "$FILE" 2>/dev/null) || { echo "ERROR: Cannot read resolved file: $FILE"; exit 1; }

TMPDIR=$(mktemp -d)
trap 'rm -rf "$TMPDIR"' EXIT

echo "$BASE"     > "$TMPDIR/base"
echo "$OURS"     > "$TMPDIR/ours"
echo "$THEIRS"   > "$TMPDIR/theirs"
echo "$RESOLVED" > "$TMPDIR/resolved"

echo "=== AUDIT: $FILE ==="
echo ""

OURS_ADDED=$(diff "$TMPDIR/base" "$TMPDIR/ours" | grep '^>' | sed 's/^> //' | grep -v '^\s*$' || true)
THEIRS_ADDED=$(diff "$TMPDIR/base" "$TMPDIR/theirs" | grep '^>' | sed 's/^> //' | grep -v '^\s*$' || true)

MISSING_OURS=0
MISSING_THEIRS=0

if [ -n "$OURS_ADDED" ]; then
    echo "Lines added by OURS that may be missing from resolution:"
    while IFS= read -r line; do
        trimmed=$(echo "$line" | sed 's/^[[:space:]]*//' | sed 's/[[:space:]]*$//')
        if [ -n "$trimmed" ] && ! grep -qF "$trimmed" "$TMPDIR/resolved"; then
            echo "  MISSING: $line"
            MISSING_OURS=$((MISSING_OURS + 1))
        fi
    done <<< "$OURS_ADDED"
fi

if [ -n "$THEIRS_ADDED" ]; then
    echo "Lines added by THEIRS that may be missing from resolution:"
    while IFS= read -r line; do
        trimmed=$(echo "$line" | sed 's/^[[:space:]]*//' | sed 's/[[:space:]]*$//')
        if [ -n "$trimmed" ] && ! grep -qF "$trimmed" "$TMPDIR/resolved"; then
            echo "  MISSING: $line"
            MISSING_THEIRS=$((MISSING_THEIRS + 1))
        fi
    done <<< "$THEIRS_ADDED"
fi

echo ""
if [ "$MISSING_OURS" -eq 0 ] && [ "$MISSING_THEIRS" -eq 0 ]; then
    echo "AUDIT PASSED: All additions from both sides are present in the resolution."
else
    echo "AUDIT WARNING: $MISSING_OURS line(s) from OURS and $MISSING_THEIRS line(s) from THEIRS may be missing."
    echo "Review the lines above. If they were intentionally superseded, this is OK — document the reason."
    exit 1
fi
```

---

## Step 5: VERIFY — Confirm Correctness

After all files are resolved:

```bash
# Confirm no remaining conflict markers anywhere
grep -rn '<<<<<<<\|=======\|>>>>>>>' --include='*.ts' --include='*.tsx' \
    --include='*.js' --include='*.jsx' --include='*.json' --include='*.mts' \
    --include='*.cts' .

# Confirm no unresolved files remain
git diff --name-only --diff-filter=U

# Type-check the project
npx tsc --noEmit 2>&1 | head -30

# Lint check
npx eslint . --max-warnings=0 2>&1 | tail -10

# Run the project's test suite
npm test 2>&1 | tail -20
```

Present verification results. If anything fails, diagnose and fix before completing.

---

## Step 6: COMPLETE — Finish the Operation

```bash
# For a merge:
git commit  # Git will auto-generate the merge commit message

# For a rebase — continue to the next commit:
git rebase --continue

# For a cherry-pick:
git cherry-pick --continue
```

After completing, provide a summary:
- How many files were resolved
- What resolution strategy was used for each
- Any changes that were intentionally dropped (with reasons)
- Verification results
- How to undo if something looks wrong (the backup branch/stash from Step 1)

---

## Special Scenarios

### Rebase Conflicts Across Many Commits

When rebasing a branch with many commits onto a diverged target, the same file may conflict repeatedly across different commits. This is the most error-prone scenario for losing changes.

**Strategy: Commit-by-Commit with State Tracking**

Track which commits have been resolved and what decisions were made:

```bash
# Before starting, know what you're dealing with
git log --oneline <upstream>..HEAD | wc -l  # how many commits to replay
git log --oneline <upstream>..HEAD           # what each one does
```

For each commit in the rebase:
1. Identify which files conflict and check if they also conflicted in a previous commit
2. If the same file conflicted before, review the prior resolution to maintain consistency
3. Resolve using the standard framework (classify → decide → resolve → audit)
4. `git rebase --continue` to advance to the next commit

Key risks:
- **Accumulated drift**: By commit 5, the resolved version may have drifted far from both the original "ours" and "theirs". Always compare against the base version, not just the previous resolution.
- **Recurring conflicts in the same region**: If the same code region conflicts across multiple commits, consider squashing those commits first and resolving once.

**When to Suggest Squash-and-Rebase Instead:**

If more than 3 commits conflict in the same file:

```bash
git rebase --abort
git rebase -i <upstream>      # squash related commits first
git rebase <target>           # then rebase the squashed branch
```

### Monorepo Conflicts

In TypeScript/Node.js monorepos (Turborepo, Nx, Lerna, npm/yarn/pnpm workspaces), conflicts can span multiple packages that depend on each other. Resolution order matters.

**Strategy: Dependency-Ordered Resolution**

1. Map the dependency graph between conflicted packages
2. Resolve bottom-up: start with leaf dependencies, work toward the root
3. After resolving each package, check that its dependents still compile with `npx tsc --noEmit`

```bash
# Identify which packages are affected
git diff --name-only --diff-filter=U | cut -d'/' -f1-2 | sort -u

# If using Turborepo, check the dependency graph
npx turbo run build --dry-run 2>/dev/null | head -20
```

Special concerns:
- **Shared configuration files** (tsconfig.json, tsconfig.base.json, .eslintrc, workspace config): Merge both additions carefully — especially `paths` and `references` in tsconfig.
- **Cross-package imports**: If one side refactored a shared utility package, all consuming packages need their imports updated.
- **Root lock file**: Regenerate after resolving all package.json files across the workspace.
- **TypeScript project references**: Ensure the dependency order is consistent after resolution.

### Binary File Conflicts

Git cannot merge binary files. You must choose one version or regenerate.

```bash
# See which binary files conflict
git diff --name-only --diff-filter=U | while read f; do
  file "$f" | grep -qv text && echo "BINARY: $f"
done
```

Decision framework:
- **Generated binaries** (bundled assets, webpack/esbuild output): Regenerate from source after resolving source conflicts
- **Design assets** (images, icons, fonts): Ask which version to keep, or if both are needed
- **Data files** (SQLite DBs, .wasm files): Usually need regeneration from the source

```bash
git checkout --ours <binary-file> && git add <binary-file>    # Accept ours
git checkout --theirs <binary-file> && git add <binary-file>  # Accept theirs
```

Always note which version was kept and why.

### Submodule Conflicts

Submodule conflicts happen when both branches updated the submodule pointer to different commits.

```bash
# See the conflict
git diff <submodule-path>

# Check what each side points to
git ls-tree HEAD <submodule-path>           # ours
git ls-tree MERGE_HEAD <submodule-path>     # theirs
```

Resolution:
1. Determine which submodule commit is "ahead" (usually the newer one is correct)
2. If both sides made independent changes to the submodule, you may need to merge within the submodule itself
3. Update the pointer:

```bash
cd <submodule-path>
git checkout <desired-commit>
cd ..
git add <submodule-path>
```

### Large-Scale Rename/Move Conflicts

When one branch moved files to a new location while another edited them in the old location, Git may not detect the rename and will report the edit as a conflict against a deletion.

**Detection:**

```bash
git status | grep "deleted by"
git log --diff-filter=R --summary MERGE_HEAD -- <old-path>
```

**Strategy:**

1. Find where the file was moved to
2. Apply the edits from the other branch to the file in its new location
3. Confirm the old path is removed and the new path has the combined changes

```bash
git log --diff-filter=R --name-status MERGE_HEAD | grep <old-filename>
git diff :1:<old-path> :3:<old-path> > /tmp/their-edits.patch
patch <new-path> /tmp/their-edits.patch
git add <new-path>
git rm <old-path> 2>/dev/null
```

### Octopus Merges (Multiple Branches)

Git's octopus strategy does not handle content conflicts — it will abort. Instead, merge branches one at a time in dependency order:

```bash
git merge feature-a    # resolve any conflicts
git merge feature-b    # resolve any conflicts
git merge feature-c    # resolve any conflicts
```

### Conflict in Generated Code

Files like `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `.generated.ts`, GraphQL codegen outputs, or Prisma client code should **never be manually merged**.

Strategy:
1. Accept either side (doesn't matter which)
2. Resolve the source files first (package.json, .graphql schemas, prisma/schema.prisma, etc.)
3. Regenerate the derived file:

```bash
# npm lock files
git checkout --theirs package-lock.json && npm install

# Yarn
git checkout --theirs yarn.lock && yarn install

# pnpm
git checkout --theirs pnpm-lock.yaml && pnpm install

# GraphQL codegen
npx graphql-codegen

# Prisma client
npx prisma generate

# Other generated code
npm run generate 2>/dev/null || npm run codegen 2>/dev/null
```

Always regenerate rather than manually editing generated files.

---

## Communication Style

When presenting conflict analysis and resolutions:

- Lead with the **summary table** so you see scope at a glance
- For each non-trivial conflict, explain **what each side intended** before showing the resolution
- Use diff-style formatting to show what the resolution looks like
- Always mention the **safety backup** and how to restore it
- If a resolution requires judgment (Type B), present both options with tradeoffs and let the user decide
- Never silently drop changes — if something must be removed, say so explicitly with reasoning
