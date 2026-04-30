---
name: lodestar-daily-check
description: >
  Brief weekday-morning check. Surfaces ONE thing worth attention today,
  drawn from goals, in-flight projects, and the last few journal entries.
  Idempotent for the day — re-invoking shows the same surface, not a new one.
  Use when Paul says "daily check", "what should I focus on today", "lodestar
  morning check", or runs /lodestar-daily-check.
---

# lodestar-daily-check

Short, specific, one-thing-only. Sets the tone for the day without nagging.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Read `~/Git/Projects/lodestar/references/nudge-tone.md` and follow it
   strictly — this skill is the most public-facing nudge channel.
3. Read `~/Git/Projects/lodestar/references/goal-mapping.md` for keyword sets.

## Idempotency

Before generating output, check `~/Git/Projects/lodestar/nudges/log.jsonl` for
an entry from `lodestar-daily-check` dated today. If one exists, replay the
same message rather than generating a fresh one — Paul re-invoking should not
produce a different "thing of the day."

## Procedure

### Step 1 — Gather signal

In parallel:

- Read today's journal entry if it exists (`journal`-tagged note dated today).
- Read yesterday's journal entry (helps detect "I'll finish this tomorrow"
  promises).
- Read the "Working on" view of `Bases/Projects.base` — projects with active
  status.
- Read `lead-calls-prep/references/goals.md` for current goals + Emerging
  Areas section.
- Read the last 5 entries in `nudges/log.jsonl` to avoid recycling.

### Step 2 — Pick the surface

Choose **one** of:

1. **An unfinished commitment from yesterday.** Highest priority. If
   yesterday's journal says "I'll do X tomorrow" and X isn't in today's
   journal yet, surface X.
2. **A project with a stale review.** If a project has `reviewed:` older than
   2 weeks AND was mentioned in journal recently, surface "this project
   hasn't been reviewed in 2+ weeks but you've been working on it — want a
   quick GTD pass on just that one?"
3. **A quiet goal.** Use the keyword/duration thresholds in
   `goal-mapping.md`. Pick the goal that's been quiet longest.
4. **An emerging area** from `goals.md`. The Training Simulator plugin is
   explicitly flagged as easy to forget — give it preference among emerging
   areas if it's been quiet 14+ days.
5. **Nothing.** If none of the above triggers, output:
   > Nothing pressing this morning. [Brief observation about what's looking
   > healthy — e.g. "Goal 2 had three journal mentions yesterday."] Have a
   > good day.

### Step 3 — Phrase it

Apply `nudge-tone.md`. Specifically:

- Cite a date or count
- Offer a small concrete next step (sized in minutes)
- Ask, don't tell
- One thing only
- No emoji, no exclamation marks

### Step 4 — Log it

Append to `~/Git/Projects/lodestar/nudges/log.jsonl`:

```json
{"date":"2026-04-30","skill":"lodestar-daily-check","goal":"Goal 1","message":"<the message you sent>","action":"surfaced"}
```

If Paul responds (in the same conversation) with "scheduled it" / "added to
todoist" / etc., append a follow-up entry with `"action":"acted"`.

### Step 5 — Self-perpetuating reminder (optional)

After the run, check Todoist for an upcoming `Run /lodestar-daily-check in Claude` task scheduled for the next weekday. If none exists, offer:

> Want me to add tomorrow's daily check to Todoist?

If Paul says yes, create the task in Todoist with the appropriate due date. This keeps the reminder loop alive without him having to maintain it manually. Skip the offer if he's declined the same offer in the last 7 days (check `nudges/log.jsonl` for `"action":"declined-perpetuation"` entries).

## Rules

- **One thing.** Not a list. Not a scorecard.
- **Idempotent for the day.** Re-invoking returns the same surface.
- **Don't repeat yesterday's nudge.** If `goals-nudge` or
  `daily-check` flagged Goal 1 yesterday and Paul didn't act, surface
  something else today — not the same goal again.
- **Skip if Paul is already on it.** If today's journal mentions the goal
  you'd surface, drop down to "Nothing pressing this morning."
- **Don't push to Todoist** unless Paul explicitly says yes to a "want me to
  add this to Todoist?" offer.
