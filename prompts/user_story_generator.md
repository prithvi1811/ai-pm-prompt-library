# Prompt Library — User Story Generator

## The PM pain point
Turning a rough feature idea or ticket into well-formed user stories with acceptance
criteria is repetitive, easy to do inconsistently across a team, and eats 20-40 minutes
per feature when done by hand (story splitting, edge cases, INVEST checks, AC format).

## Structured prompt (v1 → v3)

### v1 — naive prompt
```
Write user stories for: "Add a dark mode toggle to the settings page"
```
Output is inconsistent: sometimes one story, sometimes five, no acceptance criteria,
no sizing signal, mixes UI detail with business value.

### v2 — add role + format constraints
```
You are a senior product manager. Convert the feature description below into 2-5
user stories in "As a ___, I want ___, so that ___" format. Add 3-5 acceptance
criteria per story in Given/When/Then format.

Feature: "Add a dark mode toggle to the settings page"
```
Better, but still misses edge cases (persistence, system-theme sync, accessibility)
and gives no sense of relative size or risk.

### v3 — final structured prompt (used in the n8n workflow)
```
You are a senior product manager writing stories for an engineering team to estimate
and build directly from. Given the feature description below, produce:

1. 2-6 user stories, each as:
   - Title (≤8 words)
   - Story: "As a [persona], I want [capability], so that [benefit]"
   - Acceptance criteria: 3-6 bullet points in Given/When/Then form
   - Edge cases to consider: 1-3 bullets (things engineers will ask about)
   - Relative size: XS / S / M / L (gut-check, not a commitment)

2. A short "Open questions for the PM" list — anything ambiguous in the brief that
   should be resolved before sprint planning.

Be concrete. Avoid filler language ("seamless", "robust", "intuitive"). If the
feature description is too vague to split into stories, say so and list the
clarifying questions instead of guessing.

Feature description:
"""
{{feature_description}}
"""
```

## Why v3 works
- **Role + audience framing** ("for an engineer to estimate from") pushes the model
  toward concrete, testable output instead of marketing copy.
- **Enumerated output contract** (title/story/AC/edge cases/size/open questions) makes
  output parseable and consistent — which is what lets this run unattended in n8n and
  feed straight into a ticket or doc.
- **Escape hatch** ("if too vague, ask questions") prevents confident hallucination on
  underspecified input — the single biggest failure mode of v1/v2.
- **Banned filler list** removes the most common AI-writing tell or PM Don't write style note.

## Before / after (see examples/user_story_before_after.md for full output)
| | Before (manual) | After (v3 prompt via n8n) |
|---|---|---|
| Time to first draft | ~25 min | ~90 sec |
| Stories missing AC | common | 0 (enforced by format) |
| Edge cases surfaced | only if PM remembers | 1-3 per story, every time |
| Consistency across PMs | low (style varies) | high (same schema every run) |
