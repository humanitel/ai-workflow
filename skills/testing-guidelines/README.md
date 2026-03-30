# testing-guidelines

> Opinionated guide for writing effective tests: integration-first, real fixtures, minimal mocking, and mandatory regression coverage for bugs.

## What it does

The `testing-guidelines` skill provides a reference set of principles and technical patterns for writing tests in this codebase. It covers six core principles: mocking external services while using real sanitized fixtures, preferring integration tests over unit tests, minimizing exhaustive edge-case coverage, always adding regression tests for bugs, covering every public entry point with at least one happy-path test, and writing tests alongside code rather than after. Technical guidelines include file naming conventions (`*.test.ts` co-located with source), test isolation patterns using temporary directories and `afterEach` cleanup, and examples for both stateful and pure-function tests using Vitest.

## How to use it

This skill is a reference document rather than an invocable command. Consult it when:
- Adding new functionality and deciding what to test
- Fixing a bug and determining whether a regression test is needed
- Reviewing a PR and evaluating test quality
- Setting up test scaffolding for a new module

Run tests with:
```bash
pnpm test        # watch mode
pnpm test:run    # single run
```

## Why it's useful

Without shared testing conventions, codebases accumulate a mix of over-specified unit tests that break on every refactor and under-specified integration tests that miss real regressions. These guidelines establish a clear philosophy — test behavior at the public interface, use real data, and always lock in bug fixes with a test — that keeps the test suite both trustworthy and maintainable as the codebase grows.
