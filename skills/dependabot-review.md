---
name: dependabot-review
description: Analyze Dependabot alerts and assess actual impact on the codebase.
category: Security
tags: [security, dependabot, dependencies]
---

**Guardrails**

- Do NOT make any code changes. This skill is research and reporting only.
- Always verify whether the vulnerable code path is actually reachable before rating risk.
- Distinguish clearly between direct and transitive dependencies, and between dev-only and production dependencies.
- Do not dismiss alerts solely based on severity rating; always check actual usage.

**Steps**

1. Fetch all open Dependabot alerts for the current repository using `gh api repos/{owner}/{repo}/dependabot/alerts` filtered to `state=open`. Parse each alert's number, severity, package name, and summary.
2. For each alerted package, determine the dependency type:
   - Check `package.json` for direct dependencies and devDependencies.
   - Run `npm ls <package>` to identify the full dependency chain for transitive dependencies.
   - Note which parent package pulls in the vulnerable transitive dependency.
3. For each alerted package, search the codebase (`src/`) for direct imports or usage:
   - Search for `import ... from '<package>'` and `require('<package>')` patterns.
   - If not directly imported, identify if/how the transitive consumer uses it (e.g., AWS SDK parsing XML, Express parsing query strings).
4. Assess vulnerability reachability for each alert:
   - Read the vulnerability description and identify the specific precondition or API that must be used for exploitation.
   - Cross-reference against actual usage found in step 3 to determine if the vulnerable code path is exercised.
   - Consider whether user-controlled input can reach the vulnerable code path.
5. Check available patched versions for each alerted package:
   - For each alert, extract the `first_patched_version` from the Dependabot alert data using `gh api repos/{owner}/{repo}/dependabot/alerts --jq '.[] | select(.state=="open") | {number, package: .security_vulnerability.package.name, current_version: .security_vulnerability.vulnerable_version_range, patched_version: .security_vulnerability.first_patched_version.identifier}'`.
   - For direct dependencies, run `npm outdated <package>` to compare the current installed version, wanted version, and latest available version.
   - For transitive dependencies, identify which direct dependency needs to be updated to pull in the patched transitive version (e.g., updating `@aws-sdk/client-s3` to get a fixed `fast-xml-parser`).
   - Note any breaking changes or major version bumps that would be required to reach the patched version.
6. Produce a summary report in this format:

   **Impact Analysis Table** — one row per alert with columns: Alert #, Package, Severity, Direct/Transitive, In Production?, Vulnerable Path Reachable?, Installed Version, Patched Version, Risk Level.

   **Key Findings** — one bullet per package explaining why it is or isn't reachable, referencing specific files and code patterns found.

   **Available Updates** — for each package with a known fix:
   - Current version vs patched version.
   - For direct deps: the `npm install <package>@<version>` command to run.
   - For transitive deps: which parent dependency to update and the command to run.
   - Flag any major version bumps that may require code changes.

   **Recommendations** — group alerts into:
   - **Safe to dismiss**: dev-only or completely unreachable.
   - **Accept risk / monitor**: in production but not exploitable given current usage.
   - **Update for good hygiene**: not currently exploitable but a patched version is available — include the update command.
   - **Action required**: reachable vulnerable paths that need patching or mitigation.

**Reference**

- Use `gh api repos/{owner}/{repo}/dependabot/alerts` to fetch alerts (supports `--jq` for filtering).
- Use `npm ls <package>` to trace dependency trees.
- Use `npm outdated <package>` to check current vs latest versions for direct dependencies.
- Use `npm view <package> versions --json` to list all available versions when checking for specific patched releases.
- Search code with `rg` or Grep for import patterns and specific vulnerable API calls mentioned in advisories.
- Check `package.json` and `package-lock.json` for version information.
