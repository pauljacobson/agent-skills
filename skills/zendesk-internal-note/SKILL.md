---
name: zendesk-internal-note
description: Use when asked to write, draft, or format an internal note for a Zendesk ticket — including phrasings like "create an internal note", "internal note for this ticket", "Zendesk note", "add a note for the next agent", or "leave a handoff note". Produces an internal note using the standard team template (previous note pointer, progress, next steps) so handoffs between agents stay consistent.
---

# Zendesk Internal Note

Internal notes are how agents hand a ticket off to whoever picks it up next. Consistency matters more than prose here: a teammate scanning the ticket should be able to find, at a glance, what's already been said, what's been done, and what's still outstanding. This template enforces that structure.

## The template

Produce the note using exactly this structure — the emoji, headings, and order are part of the convention and should not be changed:

```
📌 Please see previous note/s

📐 Progress: 
- 

✅ Next steps: 
- 
```

## How to fill it in

- **📌 Please see previous note/s** — Keep this line as-is. It's a standing pointer telling the next agent to read the earlier notes for full context.
- **📐 Progress** — Replace the empty bullet with what has actually been done so far on the ticket: investigation done, what was found, actions taken, customer replies sent. Add one bullet per distinct point. If you don't have the ticket details, leave the single empty bullet so the agent can fill it in.
- **✅ Next steps** — Replace the empty bullet with the concrete actions still outstanding, ideally phrased so the next person knows exactly what to pick up. One bullet per step.

Keep the trailing space after `Progress:` and `Next steps:` and the blank lines between sections — the layout is intentional so the note renders cleanly in Zendesk.

If the user hasn't given you any ticket content, return the blank template verbatim so they can paste and fill it in themselves. If they've described what they did and what's left, fill in the bullets accordingly while preserving the structure.
