# Prompt-Engineering Case Study: A Workflow Library for Repetitive PM Tasks

**TL;DR:** I picked two tasks that eat a disproportionate share of a PM's week:
writing user stories from feature ideas, and turning meeting transcripts into
recaps with owners and dates. I built a versioned prompt library for each
(naive, then constrained, then production-ready), wired the final version into
an n8n workflow that runs on a click or a webhook, and measured the before/after
difference in time and quality.

This is meant to demonstrate the muscle that's differentiating PM candidates
right now: not just "I can use ChatGPT," but "I can identify where AI removes
friction from a real workflow, design the prompt like a spec, and ship it as
something a team could actually run."

## Why these two tasks
They're high frequency: they happen every sprint or every meeting, so small time
savings compound fast. They're inconsistent when done by hand, since output
quality depends heavily on who's writing it and how rushed they are, which is
exactly the kind of variance a structured prompt can flatten out. And they're
easy to check. "Does this user story have testable Given/When/Then AC?" and
"does this recap give every action item an owner and a date, without inventing
anything?" are concrete yes-or-no questions, not vibes.

## What's in this repo
```
prompts/
  user_story_generator.md      (prompt versions v1 to v3, with the reasoning behind each change)
  meeting_summarizer.md        (same, for meeting recaps)
workflows/
  user_story_generator.json    (importable n8n workflow: Manual Trigger -> Claude -> output)
  meeting_summarizer.json      (importable n8n workflow: Manual Trigger -> Claude -> output)
examples/
  user_story_before_after.md   (full before/after generation on a real input)
  meeting_summary_before_after.md
```

## The prompt-engineering journey (the part that actually matters)
The interesting part isn't "I asked Claude to do X." It's what changed between
versions and why, because that's the same skill as writing a spec: define what
"good" looks like precisely enough that the model produces it reliably.

For both tasks, the arc was the same:
1. **v1 (naive):** ask for the thing directly. The output looks plausible but is
   inconsistent and not directly usable. Looks done, isn't done.
2. **v2 (add structure):** impose an output schema. Consistency improves, but the
   model still fills gaps with plausible-sounding guesses, like a due date that
   was never mentioned, or an AC that's really just the requirement restated.
3. **v3 (constrain and add an escape hatch):** name the failure mode directly
   ("don't invent decisions, owners, or dates, flag them instead") and give the
   model a structured way to say "I don't know" (`UNASSIGNED`, `no date given`,
   "Open Questions"). Of the three changes, this is the one that mattered most:
   it turns hallucination from a hidden risk into something visible and fixable.

Full version-by-version breakdowns with rationale: [`prompts/user_story_generator.md`](prompts/user_story_generator.md)
and [`prompts/meeting_summarizer.md`](prompts/meeting_summarizer.md).

## Measured impact
| Task | Manual time | With workflow | Quality delta |
|---|---|---|---|
| User stories from a feature idea | ~25 min/feature | ~90 sec gen + ~3 min review | 100% of stories ship with testable Given/When/Then AC and surfaced edge cases (vs. inconsistent by hand) |
| Meeting transcript to recap | ~18 min/meeting | ~2 min review/edit | Fabricated decisions/owners dropped from 4-of-6 test runs (naive prompt) to 0-of-6 (v3 prompt) |

At roughly 3 features and 5 meetings a week, that adds up to about 9 to 10 hours
a month given back, time that goes into the judgment calls AI can't make for
you: prioritization, stakeholder alignment, customer conversations.

## How to run it
1. Install n8n locally (`npx n8n` or via Docker) and open it at `localhost:5678`.
2. In n8n, create a Header Auth credential, e.g. named "Anthropic API Key", with
   header `x-api-key` set to your Claude API key (get one at console.anthropic.com).
3. Import a workflow: Workflows -> Import from File, then pick
   `workflows/user_story_generator.json` or `workflows/meeting_summarizer.json`.
4. Open the imported workflow, attach your credential to the Claude HTTP Request
   node (it's pre-wired to call `api.anthropic.com/v1/messages`), edit the
   "Sample Input" node with your own feature description or transcript, and
   click Test workflow.
5. The final node extracts the model's markdown so it can be piped onward, for
   example by adding a Slack, Notion, or Jira node after it to post the result
   directly where the team works. That's the obvious next version of this project.
