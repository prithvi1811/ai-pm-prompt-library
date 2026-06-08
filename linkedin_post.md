I spent the week building something I kept telling other PMs they should build: proof
that I can use AI to remove friction from real PM workflows — not just talk about it.

The result is a small open-source case study: a versioned prompt library + working
n8n automations for two tasks that eat a disproportionate share of a PM's week —

→ Turning a feature idea into user stories with testable acceptance criteria
→ Turning a meeting transcript into a recap with owners and dates attached

The interesting part wasn't "ask Claude to do X." It was iterating the prompt the same
way you'd tighten a spec — v1 (naive) produced plausible-but-inconsistent output, v2
(add structure) was better but still confidently filled gaps with guesses, and v3 (name
the failure mode + give the model a way to say "I don't know") is what actually made it
trustworthy enough to run unattended.

One number that stuck with me: the naive prompt fabricated a decision or owner in 4 of
6 test meeting transcripts. Adding one explicit instruction — "don't invent decisions,
owners, or dates; flag instead of guessing" — took that to zero.

Wired the final prompts into n8n workflows (trigger → Claude → formatted output) so
they're not just demos — they're things a team could actually run.

Full write-up, prompt versions with rationale, before/after examples, and the
importable workflows are here:
🔗 github.com/prithvi1811/ai-pm-prompt-library

#ProductManagement #AI #PromptEngineering #n8n #BuildInPublic
