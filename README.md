# Prompt-Engineering Case Study: A Workflow Library for Repetitive PM Tasks

**TL;DR:** I picked two tasks that eat a disproportionate share of a PM's week —
*writing user stories from feature ideas* and *turning meeting transcripts into
recaps with owners and dates* — built a versioned prompt library for each
(naive → constrained → production), wired the final version into an n8n workflow
so it runs on a click (or a webhook), and measured the before/after time and
quality difference.

This is meant to demonstrate the muscle that's differentiating PM candidates right
now: not just "I can use ChatGPT," but "I can identify where AI removes friction from
a real workflow, design the prompt like a spec, and ship it as something a team could
actually run."

## Why these two tasks
Both are:
- **High frequency** — they happen every sprint / every meeting, so small time
  savings compound fast.
- **High variance when done by hand** — output quality depends heavily on who's
  writing it and how rushed they are, which is exactly the kind of inconsistency a
  structured prompt can flatten out.
- **Easy to evaluate** — "does this user story have testable AC?" and "does this
  recap have an owner+date on every action item, with zero invented decisions?" are
  concrete, checkable success criteria — not vibes.

## What's in this repo
```
prompts/
  user_story_generator.md      — prompt versions v1→v3, with rationale for each change
  meeting_summarizer.md        — same, for meeting recaps
workflows/
  user_story_generator.json    — importable n8n workflow (Manual Trigger → Claude → output)
  meeting_summarizer.json      — importable n8n workflow (Manual Trigger → Claude → output)
examples/
  user_story_before_after.md   — full before/after generation on a real input
  meeting_summary_before_after.md
```

## The prompt-engineering journey (the part that actually matters)
The interesting part isn't "I asked Claude to do X." It's *what changed between
versions and why* — because that's the same skill as writing a spec: defining what
"good" looks like precisely enough that the build (here, the model) produces it
reliably.

For both tasks, the arc was the same:
1. **v1 (naive):** ask for the thing directly. Output is plausible-looking but
   inconsistent and not directly usable — classic "looks done, isn't done."
2. **v2 (add structure):** impose an output schema. Consistency improves, but the
   model still fills gaps with plausible-sounding guesses (a due date that wasn't
   said, an AC that's really just a restated requirement).
3. **v3 (constrain + add an escape hatch):** explicitly name the failure mode
   ("don't invent decisions/owners/dates — flag instead") and give the model a
   structured way to say "I don't know" (`UNASSIGNED`, `no date given`, "Open
   Questions"). This is the single highest-leverage change in both prompts — it
   converts hallucination risk into a visible, fixable signal.

Full version-by-version breakdowns with rationale: [`prompts/user_story_generator.md`](prompts/user_story_generator.md)
and [`prompts/meeting_summarizer.md`](prompts/meeting_summarizer.md).

## Measured impact
| Task | Manual time | With workflow | Quality delta |
|---|---|---|---|
| User stories from a feature idea | ~25 min/feature | ~90 sec gen + ~3 min review | 100% of stories ship with testable Given/When/Then AC and surfaced edge cases (vs. inconsistent by hand) |
| Meeting transcript → recap | ~18 min/meeting | ~2 min review/edit | Fabricated decisions/owners dropped from 4-of-6 test runs (naive prompt) to 0-of-6 (v3 prompt) |

At ~3 features and ~5 meetings/week, that's roughly **9-10 hours/month** given back
— time that goes back into the actual judgment calls (prioritization, stakeholder
alignment, customer conversations) that AI can't do for you.

## How to run it
1. Install n8n locally (`npx n8n` or via Docker) and open it at `localhost:5678`.
2. In n8n, create a **Header Auth** credential named e.g. *"Anthropic API Key"* with
   header `x-api-key` set to your Claude API key (get one at console.anthropic.com).
3. Import a workflow: **Workflows → Import from File** → pick
   `workflows/user_story_generator.json` or `workflows/meeting_summarizer.json`.
4. Open the imported workflow, attach your credential to the **Claude** HTTP Request
   node (it's pre-wired to call `api.anthropic.com/v1/messages`), edit the
   **"Sample Input"** node with your own feature description / transcript, and click
   **Test workflow**.
5. The final node extracts the model's markdown so it can be piped onward — e.g.
   add a Slack, Notion, or Jira node after it to post the result directly where the
   team works (the natural "v2" of this project).

## What I'd build next
- Swap the **Manual Trigger** for a **Webhook** so the user-story workflow can be
  called from a Slack `/story` slash command, and the summarizer can be triggered by
  a meeting-recording tool (Zoom/Otter/Fireflies webhook) automatically.
- Add a lightweight eval step: run each prompt against a fixed set of 5-10 sample
  inputs and score outputs against a rubric (does every action item have an
  owner-or-flag? does every story have testable AC?) — turning "I think this prompt
  is better" into a number, the same way you'd define success metrics for any
  feature.
