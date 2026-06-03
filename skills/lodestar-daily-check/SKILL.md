---
name: lodestar-daily-check
description: >
  Brief workday-morning check (Paul's work week: Sun–Thu). Surfaces ONE
  thing worth attention today,
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

### Step 0 — PTO check

Before gathering any signal, probe journal recency. Per
`~/Git/Projects/lodestar/references/pto-detection.md`: if there are **zero
`journal`-tagged notes in the last 5 calendar days**, set `pto_mode = True`,
output the one-liner:

> No journal entries in N days — assuming you're off; nothing to surface today.

…and **end the turn**. Do NOT log a `surfaced` entry to `nudges/log.jsonl`
(PTO mode is silent in the log to keep dedup windows clean and to avoid
implying the skill acted).

If `pto_mode = False`, proceed to Step 1.

### Step 1 — Gather signal

In parallel:

- Read today's journal entry if it exists (`journal`-tagged note dated today).
- Read yesterday's journal entry (helps detect "I'll finish this tomorrow"
  promises).
- Read the "Working on" view of `Bases/Projects.base` — projects with active
  status.
- Read `lead-calls-prep/references/goals.md` for current goals + Emerging
  Areas section.
- Read the **tracked Linear issues** from `config.md` § Linear. For each, fetch
  open gaps (`resolvedAt: null`) and note the oldest gap's age. Read-only;
  standalone Linear MCP primary, ContextA8C fallback. Skip silently if neither
  is reachable (don't block the morning check on Linear).
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
4. **A tracked Linear issue with a stale gap.** Per `config.md` § Linear and
   the `goal-mapping.md` "Tracked Linear issues" map. Surface when an open gap
   (`resolvedAt: null`) is older than **7 days** AND the issue wasn't surfaced
   in the last **3 days** (`nudges/log.jsonl`, keyed by issue id, e.g.
   `TSCODE-406`). Independent of the parent goal's journal signal. Cite the
   open-gap count + the oldest gap's date; offer a small concrete step (e.g.
   "want to slot 20 minutes to clear one, or push it to Todoist?").
5. **An emerging area** from `goals.md`. The Training Simulator plugin is
   explicitly flagged as easy to forget — give it preference among emerging
   areas if it's been quiet 14+ days.
6. **Nothing.** If none of the above triggers, output:
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

For a **tracked-issue surface** (option 4), set `"goal"` to the issue id (e.g.
`"TSCODE-406"`) per `config.md` § Linear — this keeps its 3-day dedup
independent of the parent goal's journal nudges.

If Paul responds (in the same conversation) with "scheduled it" / "added to
todoist" / etc., append a follow-up entry with `"action":"acted"`.

## Routine review output (daily-note surfacing)

After surfacing the day's item and appending to `nudges/log.jsonl`, route the
output into the daily note via `scripts/routine_review.py`. See
`plans/20260530-routine-review-daily-note-design.md`.

1. Check state and daily-note existence:
   ```bash
   python3 ~/Git/Projects/lodestar/scripts/routine_review.py status
   ```
   Read `daily_note_exists` and `last_summary_posted` from the JSON.

2. Compose two pieces of text:
   - **detail** — the full daily-check output for the routine note (the same
     content you'd show in chat).
   - **summary** — a tight 1-3 line markdown summary for the daily note. If
     `last_summary_posted` is older than today (a prior run was deferred), make
     the summary a **catch-up** covering everything since that date (read the
     relevant `nudges/log.jsonl` entries).

3. Write each to a temp file and post:
   ```bash
   python3 ~/Git/Projects/lodestar/scripts/routine_review.py post \
     --routine "Daily Check" \
     --detail-file /tmp/rr_detail.md \
     --summary-file /tmp/rr_summary.md
   ```

4. If the result is `{"status":"deferred"}`, the daily note didn't exist yet —
   that's expected on early starts; the next run with a daily note catches up.
   Do not retry or create the daily note yourself.

**Idempotent for the slot:** re-invoking the skill the same day refreshes the
single `### Routine review` block rather than duplicating it.

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
