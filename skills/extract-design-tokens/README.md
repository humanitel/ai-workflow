# extract-design-tokens

> Fetches one or more URLs and generates a complete Tokens Studio for Figma JSON design token set from the page's visual design language.

## What it does

Given one or more URLs, the skill fetches each page, systematically analyzes its visual design across all dimensions (colors, typography, spacing, borders, shadows, opacity, transitions, breakpoints, and z-index), and organizes the findings into a semantic token hierarchy. Primitive values (e.g., `blue.500`) are defined once and referenced by semantic aliases (e.g., `color.primary.default`). Where token values are not directly visible on the page, the skill infers reasonable values consistent with the overall design language and marks them with a `description` field. If multiple URLs are provided with differing designs, it unifies them into a single coherent token set and documents conflicts inline.

## How to use it

Trigger automatically when given one or more URLs and asked to extract, generate, or create a design system, design tokens, or Tokens Studio JSON.

Example prompts:
- "Extract design tokens from https://example.com"
- "Generate a Tokens Studio JSON for these two pages: [url1] [url2]"
- "Create a design system from the visual language at this URL"

Output is a single JSON code block in Tokens Studio for Figma format, ready to import directly into the Tokens Studio plugin. No explanation is included before or after the JSON.

## Why it's useful

Manually building a design token set from an existing product is tedious and error-prone, typically taking hours of back-and-forth between designers and engineers. This skill compresses that process to seconds and produces a structured, reference-linked token file that can immediately feed into Figma, style-dictionary, or any token-aware design system toolchain.
