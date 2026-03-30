# find-warden-bugs

> Detects bugs at Warden's known architectural seams using patterns distilled from 40+ historical fix commits.

## What it does

Analyzes scoped code chunks from Warden's diff pipeline against nine targeted checks, each mapped to a specific architectural zone (SDK, CLI, Config, Output, Types, Action, Triggers) and grounded in historically recurring bug patterns. The checks cover: SDK response shape assumptions, dual code path desync between `runSkill()` and `runSkillTask()`, config threading and default semantics across the three-level merge chain, concurrent task and Ink rendering coordination, output rendering consistency across terminal/JSON/JSONL/GitHub formats, scope and filtering logic for hunk line ranges, early-exit path completeness, state tracking accuracy, and error context preservation. Each finding requires HIGH confidence — traceable to specific code and confirmed triggerable — before it is reported. LOW-confidence resemblances are discarded.

## How to use it

Invoke via slash command: `/find-warden-bugs`

Provide scoped code (a diff, a file, or a set of files) from the Warden codebase. The skill first classifies which architectural zones the code touches, then runs only the checks relevant to those zones. Each reported finding includes the file path, line number, which check matched, a one-sentence description of the bug, the specific trigger condition, and a suggested fix when the fix is unambiguous. If no checks fire, the skill reports nothing rather than inventing findings.

## Why it's useful

Warden's architecture has well-documented failure modes that appear repeatedly at the same seams — the dual report path, config threading, concurrent rendering. Catching these during code review rather than in production prevents the classes of bugs that have historically required dedicated fix commits, and the confidence calibration ensures the signal-to-noise ratio stays high enough to act on without manual verification overhead.
