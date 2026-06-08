# Before / After — User Story Generator

**Input feature description:**
> Add a dark mode toggle to the settings page so users can switch between light and dark themes.

## Before — naive prompt (v1: "Write user stories for: ...")
```
User Story: As a user, I want to be able to switch to dark mode so that I can use
the app comfortably at night.

Acceptance Criteria:
- There is a toggle in settings
- The app changes to dark colors
- It should be easy to use
```
*One story, vague AC ("easy to use" isn't testable), no edge cases, no sizing,
no sense of what else might need to be specified.*

## After — structured prompt (v3, run through the n8n workflow)
```
## Story 1: Add dark mode toggle
As a user, I want a dark mode toggle in Settings, so that I can switch the app's
appearance to suit my environment and preference.
Acceptance criteria:
- Given I'm on the Settings page, when I open Appearance, then I see a Light/Dark/
  System toggle
- Given I select Dark, when I navigate the app, then all screens render in the dark
  theme without unstyled flashes
- Given I select System, when my OS theme changes, then the app theme updates to match
- Given I close and reopen the app, when I had selected Dark, then it remains Dark
Edge cases to consider:
- What happens to user-uploaded images/brand colors that assume a light background?
- Behavior on devices/browsers that don't support `prefers-color-scheme`
Relative size: M

## Story 2: Persist theme preference across sessions
As a returning user, I want my theme choice remembered, so that I don't have to
reset it every time I open the app.
Acceptance criteria:
- Given I select a theme, when I log out and back in, then my theme persists
- Given I'm on a new device, when I log in, then my last-saved theme preference applies
Edge cases to consider:
- Conflict resolution if the same account sets different themes on different devices
Relative size: S

## Open questions for the PM
- Should "System" be the default for new users, or should we default to Light?
- Do we need an analytics event to track adoption of dark mode?
```

**What changed:** two clearly-scoped, independently shippable stories instead of one
vague one; testable Given/When/Then AC; edge cases an engineer would otherwise raise
in planning (now surfaced before the meeting); a sizing signal; and two open questions
that would have come up mid-sprint and caused a re-plan.

**Time:** ~25 min manual draft → ~90 sec generation + ~3 min PM review/edit.
