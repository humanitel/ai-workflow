---
name: fuse-review-pr
description: This commands verify the current code against a set of pr review instructions
---

# PR Review Instructions

## Correctness & Logic

1. Verify the code does what the PR description claims — flag any gaps between intent and implementation
2. Check for edge cases: null/undefined values, empty collections, boundary conditions, race conditions
3. Ensure error handling is adequate — no swallowed exceptions, meaningful error messages, proper fallback behavior

## Security

4. Flag any security concerns: SQL injection, XSS, secrets in code, insecure deserialization, improper auth checks
5. Check that user input is validated and sanitized before use
6. Ensure sensitive data isn't logged or exposed in error responses

## Code Quality

7. Ensure cyclomatic and cognitive complexity is not too high — suggest extraction into smaller functions where needed
8. Flag code duplication — suggest reusable abstractions where patterns repeat
9. Check naming clarity — variables, functions, and classes should communicate intent without needing comments
10. Ensure no dead code, unused imports, or commented-out blocks are left behind

## Architecture & Design

11. Verify the change follows existing codebase patterns and conventions — flag deviations
12. Check for proper separation of concerns — business logic shouldn't leak into controllers, UI layers, etc.
13. Flag any tight coupling or dependencies that will make future changes painful

## Performance

14. Identify potential N+1 queries, unnecessary loops, or expensive operations inside hot paths
15. Check for missing database indexes on new query patterns
16. Flag any unbounded operations (queries without limits, collections that could grow indefinitely)

## Testing

17. Verify new logic has corresponding tests — especially for branching paths and error scenarios
18. Check that tests are meaningful, not just asserting the obvious or testing implementation details
19. Flag any existing tests that should be updated due to behavior changes

## Observability & Operability

20. Ensure adequate logging at appropriate levels for debugging production issues
21. Check that new features have proper metrics or monitoring hooks where relevant
22. Verify database migrations are backward-compatible and reversible

## General

23. Check for breaking changes to public APIs or shared interfaces — ensure they're intentional and documented
24. Flag any TODO/FIXME comments that should be tracked as issues rather than left in code
25. Be constructive — distinguish between blocking issues, suggestions, and nits