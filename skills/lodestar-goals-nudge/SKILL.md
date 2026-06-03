---
name: lodestar-goals-nudge
description: >
  Cross-references the last 7 days of journal entries against the goals doc
  and surfaces ONE goal or emerging area that's gone quiet. Slower cadence
  than lodestar-daily-check (2-3 times per week, not daily). Use when Paul
  says "goals nudge", "what am I neglecting", "check my goals", or runs
  /lodestar-goals-nudge.
---

# lodestar-goals-nudge

The deeper periodic check. Where `lodestar-daily-check` is "what should I
focus on today," this is "what have I been ignoring across this whole goal
arc?"

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Read `~/Git/Projects/lodestar/references/nudge-tone.md` — strict tone
   guidance.
3. Read `~/Git/Projects/lodestar/references/goal-mapping.md` for keywords.

## Procedure

### Step 0 — PTO check

Before reading goals, probe journal recency. Per
`~/Git/Projects/lodestar/references/pto-detection.md`: if there are **zero
`journal`-tagged notes in the last 5 calendar days**, set `pto_mode = True`,
output the one-liner:

> No journal entries in N days — looks like PTO. Nothing to flag.

…and **end the turn**. Do NOT log a `surfaced` entry to `nudges/log.jsonl`
(PTO mode is silent so the dedup window doesn't get polluted).

If `pto_mode = False`, proceed to Step 1.

### Step 1 — Read goals

Read `lead-calls-prep/references/goals.md`. Extract:

- Each Goal heading and its `Status:`
- The "Emerging Areas / To Discuss" bullets
- The "Ongoing Development Focus" sections (Attention to Detail, Root Cause)

### Step 2 — Read journal window

Read the last 7 entries with tag `journal` (typically last 7 workdays, but
literal last 7 entries to handle weekends/PTO). Read also the most recent
`weekly_update`-tagged note if there is one within 14 days.

### Step 3 — Score each goal/area

For each goal and emerging area:

- Count keyword matches in the journal window (case-insensitive,
  word-boundary).
- Note Linear/PR mentions if visible in journal.
- Note whether the area appears in the recent weekly update.

For **tracked Linear issues** (`config.md` § Linear), score differently — by
**open-gap age, not journal keywords**: read open gaps (`resolvedAt: null`) via
the standalone Linear MCP (ContextA8C fallback; read-only) and record the oldest
gap's age. These are independent candidates, not gated behind the parent goal's
journal quietness.

Build a small scoreboard in working memory:

```
Goal 1 (Cybersecurity Certificate, Paused): 0 mentions, last seen YYYY-MM-DD
Goal 2 (AI-Assisted Workflows, Active):     14 mentions
Goal 3 (Scripting/CLI, Active):              2 mentions
Training Simulator (Emerging):               0 mentions, last seen YYYY-MM-DD
Same-site migration tracking (Emerging):     1 mention
TSCODE-406 (LibreChat agent → Goal 2):       9 open gaps, oldest 11 days
...
```

### Step 4 — Pick one

Decision logic, in order:

1. **Recent log dedup.** Read last 14 days of `nudges/log.jsonl`. Skip any
   goal/area that was nudged in the last 5 days unless Paul logged
   `"action":"acted"` on it (i.e. he engaged — feel free to re-engage).
1b. **Tracked-issue eligibility (tighter dedup).** Tracked Linear issues use a
   **3-day** dedup window (not 5), keyed by issue id (e.g. `TSCODE-406`). A
   tracked issue with an open gap older than **7 days** and not surfaced in the
   last 3 days is an eligible candidate; rank it by oldest-gap age alongside
   quiet goals/areas in rule 3. Log it with `"goal":"<issue-id>"`.
2. **Emerging area priority.** If the **Training Simulator plugin** has been
   quiet 14+ days, prefer it (explicit instruction in `goal-mapping.md`).
3. **Quiet-longest wins.** Among remaining candidates, pick the goal/area
   with the longest quiet streak.
4. **Status-aware tone.** If the chosen goal is paused (e.g. Goal 1), frame
   as a check-in, not a push (see `goal-mapping.md`).

### Step 5 — Phrase it

Per `nudge-tone.md`. Specifically for goals-nudge:

- Cite the count and last-seen date.
- Acknowledge the goal's status (Paused / Active / Emerging area).
- Offer a small concrete next step.
- Allow Paul to say "not now" without follow-up.

Example (for a paused goal):

> Goal 1 (Cybersecurity Certificate) hasn't appeared in your journal since
> 12 April — about 18 days. Sarah explicitly said not to stress about
> timeline, so this is just a check-in: still feeling like the right time
> to leave it, or worth a 30-minute module this week?

Example (for an emerging area):

> Training Simulator hasn't shown up in your journal in 16 days. The goals
> doc flags it as easy to forget when the queue is busy. Want me to add a
> "scan TS issues" task to Todoist for tomorrow, or skip?

Example (no-op):

> All three goals had signal this week (Goal 2 strong, Goal 3 light, Goal 1
> still paused as planned). Nothing to flag.

### Step 6 — Log it

Append to `~/Git/Projects/lodestar/nudges/log.jsonl` as
`lodestar-goals-nudge`. If Paul accepts the offer (e.g. "yes, add it"),
update the entry with `"action":"acted"` after the action completes.

## Routine review output (daily-note surfacing)

After surfacing the goal/area and appending to `nudges/log.jsonl`, route the
output into the daily note via `scripts/routine_review.py`. See
`plans/20260530-routine-review-daily-note-design.md`.

1. `python3 ~/Git/Projects/lodestar/scripts/routine_review.py status` — read
   `daily_note_exists` and `last_summary_posted`.
2. Compose **detail** (full goals-nudge output) and **summary** (1-3 lines;
   catch-up since `last_summary_posted` if a prior run was deferred).
3. Post:
   ```bash
   python3 ~/Git/Projects/lodestar/scripts/routine_review.py post \
     --routine "Goals Nudge" \
     --detail-file /tmp/rr_detail.md \
     --summary-file /tmp/rr_summary.md
   ```
4. A `deferred` result is expected on early starts; the next run catches up.
   Do not create the daily note yourself.

**Idempotent for the slot:** re-invoking refreshes the single block.

### Step 7 — Self-perpetuating reminder (optional)

Same pattern as `lodestar-daily-check`: after the run, check Todoist for the next scheduled `Run /lodestar-goals-nudge` task. If missing, offer to create it (default cadence: Tue/Thu 16:00). Skip the offer if recently declined.

## Rules

- **One thing per invocation.** If two goals are equally quiet, pick one and
  hold the other for next time.
- **Respect explicit pauses.** Goal 1 has Sarah's "don't stress" context;
  any nudge there should reflect that.
- **No-op is a valid output.** If everything's healthy, say so briefly and
  end the turn.
- **Don't double up with daily-check.** If `daily-check` already surfaced a
  goal today, this skill should pick a different one (or no-op).
