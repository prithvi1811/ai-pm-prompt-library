# Prompt Library — Meeting Summarizer

## The PM pain point
PMs sit in 10-15 hrs/week of meetings and are usually the ones expected to push out
a recap with decisions and owners within the hour. Manual notes-to-recap is ~15-20
minutes per meeting and quality drops the later in the day it gets done.

## Structured prompt (v1 → v3)

### v1 — naive prompt
```
Summarize this meeting transcript: {{transcript}}
```
Produces a wall-of-text paragraph. No owners, no deadlines, decisions buried in
narrative, unusable as a Slack post without rewriting.

### v2 — add sections
```
Summarize this transcript into: Summary, Decisions, Action Items.

Transcript: {{transcript}}
```
Better structure, but action items come back without owners or due dates, and the
model sometimes invents a decision that was actually left open ("we should...").

### v3 — final structured prompt (used in the n8n workflow)
```
You are a PM producing a same-day recap from a raw meeting transcript. Be precise —
do not invent decisions, owners, or dates that aren't in the transcript. If something
is ambiguous (e.g., an action item with no clear owner), flag it instead of guessing.

Produce the recap in exactly this structure:

## Summary
2-4 sentences: what was the meeting for, and what was the outcome at a high level.

## Decisions made
- Bullet list. Only include things the group explicitly agreed on. If none, write
  "No firm decisions — see Open Questions."

## Action items
- [Owner — use "UNASSIGNED" if unclear] Action (Due: date if stated, else "no date given")

## Open questions / follow-ups
- Things raised but not resolved — these are what the PM needs to chase after the recap goes out.

Transcript:
"""
{{transcript}}
"""
```

## Why v3 works
- **Anti-hallucination instruction up front** — the single highest-risk failure for
  meeting recaps is a fabricated decision or due date. Naming the failure mode in the
  prompt measurably cuts it (see before/after doc).
- **"UNASSIGNED" / "no date given" placeholders** force the model to surface gaps
  rather than paper over them — which is exactly the information a PM needs to chase
  down before the recap is trustworthy.
- **Fixed markdown structure** means the n8n workflow can post it straight to Slack /
  Notion / email without reformatting — output is the deliverable, not a draft.
- **Open Questions section** turns the recap into a follow-up tool, not just an archive.

## Before / after (see examples/meeting_summary_before_after.md for full output)
| | Before (manual) | After (v3 prompt via n8n) |
|---|---|---|
| Time to recap sent | ~18 min | ~2 min (mostly read-through/edit) |
| Action items with owner+date | ~60% (PM had to chase) | flagged explicitly when missing |
| Fabricated/assumed decisions | occasional, hard to catch | near-zero (explicit instruction + flagging) |
| Format ready to paste into Slack | rarely | yes, every time |
