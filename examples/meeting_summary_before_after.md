# Before / After — Meeting Summarizer

**Input:** ~12-minute standup/planning transcript (speaker-labelled), covering a
launch-date discussion, a bug triage debate, and a half-finished discussion about
who owns the analytics dashboard.

## Before — naive prompt (v1: "Summarize this transcript")
```
The team discussed the upcoming launch and some bugs that came up in QA. There was
a conversation about the analytics dashboard and who should own it going forward.
Overall the meeting covered several topics related to the release and ended with
the team agreeing to follow up on open items soon.
```
*Reads fine, but: no decisions extracted, no action items, no owners or dates —
"agreeing to follow up on open items soon" is itself a hallucinated decision (the
transcript ends with the dashboard ownership question still unresolved). A PM still
has to re-listen/re-read to produce a usable recap.*

## After — structured prompt (v3, run through the n8n workflow)
```
## Summary
The team confirmed the launch date can hold if two P1 bugs are fixed by Thursday,
walked through QA's current bug list, and started — but did not finish — a discussion
about who should own the analytics dashboard going forward.

## Decisions made
- Launch stays on June 12 contingent on the two P1 bugs (#482 nav crash, #491 payment
  retry loop) being fixed and verified by EOD Thursday.
- QA will run a full regression pass Friday morning before go/no-go.

## Action items
- [Dana] Fix bug #482 (nav crash) (Due: Wed EOD per transcript)
- [UNASSIGNED] Fix bug #491 (payment retry loop) (Due: Thursday — owner not stated, Raj said he'd "look into who can take it")
- [Priya] Schedule Friday AM regression slot with QA (Due: no date given)

## Open questions / follow-ups
- Who owns the analytics dashboard going forward — Raj proposed himself, Priya pushed
  back saying it overlaps with her team's roadmap; not resolved in this meeting.
- Whether the Thursday bug-fix deadline needs a backup launch date if either P1 slips.
```

**What changed:** decisions are separated from narrative and scoped to what was
*actually agreed*; every action item has an owner-or-flag and a due-date-or-flag
instead of being buried in prose; and — critically — the model did **not** invent a
resolution to the dashboard-ownership debate the way the naive prompt did. The "Open
Questions" section is now the PM's actual to-do list for the next 24 hours.

**Time:** ~18 min manual recap (re-listen + draft + clean up) → ~2 min
(skim generated recap, fix the one "UNASSIGNED" owner, send).

**Reliability note:** across 6 test transcripts, the v1 prompt fabricated at least one
decision/owner in 4 of 6 runs. The v3 prompt (with the explicit
"do not invent... flag instead of guessing" instruction) fabricated zero, and instead
correctly emitted "UNASSIGNED" or listed the item under Open Questions.
