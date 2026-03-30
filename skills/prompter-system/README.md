# prompter-system

> Crafts production-grade system prompts for chatbots and AI assistants by guiding you through use case, audience, tone, and constraints to produce concise, token-efficient output.

## What it does

`prompter-system` acts as a System Prompt Engineer. It gathers four inputs — use case, target audience, tone, and key constraints — before writing anything. If you supply an existing prompt, it first runs a diagnostic step that maps each reported problem (e.g., "goes off-topic", "makes up pricing info") to a root cause, then addresses every diagnosed issue in the new draft. The draft follows a proven ordering (identity → rules → examples → context) with action-oriented imperatives, explicit scope boundaries, graceful deflection patterns instead of hard refusals, and optional few-shot examples. The result is self-reviewed against an eight-point checklist and targets 100–200 words unless the use case demands more. After delivery it recommends five specific test messages (happy path, boundary, edge case, tone check, hallucination probe) and points to evaluation tooling.

## How to use it

Auto-triggers when you ask to create, write, improve, or review a system prompt, or when you mention "system prompt", "chatbot prompt", "assistant instructions", or "write a prompt for X". You can also invoke it directly:

```
/prompter:system
```

The skill will ask for use case, audience, tone, and constraints in a single message if they are not already provided. Paste an existing prompt to get a diagnostic improvement instead of a blank-slate draft.

## Why it's useful

Vague system prompts produce inconsistent chatbot behavior that is hard to debug — `prompter-system` enforces the structural patterns (scope boundaries, tone anchors, deflection over refusal) that separate prompts that work in demos from ones that hold up in production. The built-in diagnostic step means improving an existing prompt is as structured as writing a new one, so you always know why a change was made. The test message recommendations turn prompt delivery into a testable artifact rather than a hope.
