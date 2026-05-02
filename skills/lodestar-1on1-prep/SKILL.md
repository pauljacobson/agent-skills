---
name: lodestar-1on1-prep
description: >
  Drafts a 1:1 agenda for Paul's next call with Sarah by aggregating signal
  from the last 14 days across the lodestar nudge log, journal entries,
  gh-inbox, the goals doc, and the most recent Performance Highlights notes.
  Read-only across all sources; surfaces 4 grouped buckets in chat (Wins,
  Blockers, Follow-ups from previous 1:1, Emerging areas) — does NOT write
  the formal agenda file. After surfacing, offers a hand-off to the
  lead-call-prep skill's "Prepare Agenda" workflow if Paul wants to
  formalise. Use when Paul says "prep my 1:1", "what should I raise with
  Sarah", "lodestar 1:1", or runs /lodestar-1on1-prep.
---

# lodestar-1on1-prep

Bridges goal signal, journal entries, GitHub activity, performance
highlights, and emerging-area notes into a draft agenda for the next 1:1.
Surfaces candidate topics in chat as four grouped buckets; doesn't formalise
the agenda — that hand-off goes to the existing `lead-call-prep` skill (in
`~/Git/a8c/lead-calls-prep/SKILL.md`), workflow "Prepare Agenda."

This skill is a **bridge**, not a nudge. Where `lodestar-goals-nudge` picks
one thing and gently surfaces it, this one casts a wider net and shows
candidates — Paul picks which actually go on the agenda.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Read `~/Git/Projects/lodestar/references/goal-mapping.md` for keyword
   sets used to map journal mentions back to goals.
3. Use `obsidian-cli` (skill: `obsidian:obsidian-cli`) for vault reads
   (journal entries, Performance Highlights notes).

## Window

Default: **last 14 days** (matches Paul's bi-weekly 1:1 cadence with Sarah).

Accept overrides:

- `--since 7d` — narrower (e.g. weekly 1:1 cadence, or a quick check-in)
- `--since 28d` — wider (e.g. catching up after PTO or a re-org)

If Paul invokes the skill conversationally without a window, default
silently to 14 days. If the most recent file in
`~/Git/a8c/lead-calls-prep/notes/` is older than ~21 days, ask once: *"Last
1:1 note is from YYYY-MM-DD — want me to widen the window past the default
14 days?"* — otherwise don't ask.

## Procedure

### Step 1 — Gather signal in parallel

These five reads are independent — fire them in parallel:

1. **Lodestar nudge log** — read
   `~/Git/Projects/lodestar/nudges/log.jsonl`. Filter to entries within
   the window. Group by `goal`. For each goal, note: count of entries,
   how many had `"action":"acted"` (Paul engaged), how many were declined
   or just surfaced. A goal that fired multiple times and Paul declined
   each time is a candidate for **Blockers** (something is in the way).
   A goal with strong "acted" signal is a candidate for **Wins**.

2. **Journal entries** — list `journal`-tagged notes in the window
   (typically ~10 entries on a Sun-Thu work week × 2 weeks). Read each.
   Pull bullets matching:
   - **Wins**: `shipped`, `merged`, `done`, `landed`, `released`,
     `published`, `Brandon happy`, `positive feedback`, completion verbs.
   - **Blockers**: `stuck on`, `waiting on`, `unsure about`, `blocked by`,
     `decision needed`, `not sure how to`, `tricky`, `frustrating`.
   - **Topic mentions**: anything mentioning Sarah, Marie, Brandon, or
     other names that recur in `lead-calls-prep/notes/` (likely worth
     raising).

3. **gh-inbox** — run the **non-mutating** fetch:
   ```bash
   ~/.claude/skills/gh-inbox/scripts/fetch.sh --since 14d
   ```
   (Adjust the `14d` if Paul passed a different window.) Parse the JSON.
   Group by category: assignments, body mentions, comment mentions.
   Filter to ones that look like topics worth raising — cross-team,
   blocking, repeat-mentioned, or where the issue/PR title suggests a
   decision/strategy point. Drop pure code-review pings unless they
   recur on the same PR (suggests stuck).

   **Critical**: only run with `--since` (or `--all`). Default mode
   mutates gh-inbox state and would corrupt Paul's own `/gh-inbox` flow.

4. **Goals doc** — read
   `~/Git/a8c/lead-calls-prep/references/goals.md`. Extract:
   - Each Goal heading + `Status:`
   - "Emerging Areas / To Discuss" bullets in full
   - "Ongoing Development Focus" sections (Attention to Detail, Root
     Cause troubleshooting)

5. **Performance Highlights notes** (see `config.md` → "Performance
   Highlights notes") — list files in `Notes Hub/knowledgemattic/`
   matching `* Performance Highlights.md`, sort descending, read the
   most recent **2** (covers the 14-day window typically). Use them to
   enrich the **Wins worth surfacing** bucket — they already group
   highlights by category, kudos, and CSAT. If none exist yet, skip
   without noting it (Paul knows; the consumer code in `config.md`
   handles the failure mode).

**Read-only across every source.** No writes, no state mutation, no
archive operations.

### Step 2 — Cross-reference previous 1:1 follow-ups

Read the most recent 2-3 files in `~/Git/a8c/lead-calls-prep/notes/`
(filename pattern `YYYYMMDD Lead call notes.md`, sort descending). For
each, look at:

- **`### Action Items` → `#### Mine`** — items Paul committed to. For
  each, check whether it's surfaced in journal/gh-inbox/Performance
  Highlights since. If not, flag as a follow-up (either it's done and
  not recorded, or it's slipped).
- **`### Notes for Next Call`** — explicitly flagged for follow-up.
  These all become candidate follow-ups.
- **Decisions made** — a decision in a previous 1:1 may have downstream
  questions ("we decided X — has that played out?").

This becomes the **Follow-ups** bucket.

### Step 3 — Cross-reference 1:1 history with Emerging Areas

For each "Emerging Areas / To Discuss" item from goals.md (Step 1
source 4), check the last ~6 weeks of `lead-calls-prep/notes/` for any
mention. If an area hasn't been mentioned in any recent 1:1 note, it's
a strong candidate for the **Emerging areas to discuss** bucket — Sarah
flagged it but it hasn't actually surfaced in conversation.

This avoids re-raising things Paul and Sarah already covered last
fortnight.

### Step 4 — Group and surface

Print the draft agenda in chat as four buckets, in this order. Item
order within a bucket is **recency-descending** (most recent first):

```
### Draft 1:1 prep — N items across <window> days

**Wins worth surfacing** (M items)
- <specific shipped thing> — <date or evidence cite>
  e.g. "Block plugins workflow PR merged 2026-04-25; Brandon happy with v1
  per journal entry same day."
- ...

**Blockers or decisions needed** (M items)
- <specific blocker, with what's stuck and what's needed>
  e.g. "Same-site migration ownership unclear — me or Marie? Mentioned in
  journal 2026-04-22 with no resolution since; would help to align with
  Sarah."
- ...

**Follow-ups from previous 1:1** (M items)
- <action item from notes/YYYYMMDD> — current status
  e.g. "From 2026-04-15 notes: 'Send Sarah the Training Simulator triage
  doc' — no journal mention since; status?"
- ...

**Emerging areas to discuss** (M items)
- <emerging area> — <reason it's worth raising now>
  e.g. "Boot.dev course (Sarah suggestion, ~$261/yr expensable) — hasn't
  appeared in any 1:1 note in the last 6 weeks; commit, defer, or close
  the loop?"
- ...
```

Bucket-handling rules:

- **An empty bucket gets one line, not a long apology.**
  e.g. *"**Blockers or decisions needed**: nothing specific from the data
  — anything come to mind that the journal didn't capture?"* That's it.
- **If everything is empty** (very rare): output a single line and stop:
  > Nothing jumped out across the last <window> days. Want me to widen
  > the window?
- **Cap each bucket at ~5 items.** If more candidates exist, pick the
  most recent / most cross-referenced and note "+ N more candidates if
  you want to dig deeper."
- **One bucket per item.** Don't double-list. If a candidate could fit
  Blockers and Emerging Areas, prefer the more action-oriented bucket
  (Blockers).

### Step 5 — Offer the formal hand-off

After surfacing, end with a single offer:

> Want me to take this into `lead-call-prep` to formalise the agenda?
> (It'll write the agenda file to `lead-calls-prep/agendas/YYYYMMDD Lead
> call agenda.md`.)

If Paul says yes (or "go ahead", "formalise it"), hand off to the
`lead-call-prep` skill's "Prepare Agenda" workflow. Pass the four
buckets as starting input rather than re-deriving from scratch. The
mapping into `lead-call-prep`'s sections:

| Lodestar bucket | `lead-call-prep` section |
|---|---|
| Wins worth surfacing | Progress on goals |
| Blockers or decisions needed | Blockers or challenges |
| Follow-ups from previous 1:1 | Action items from last call |
| Emerging areas to discuss | Discussion topics |

`lead-call-prep` will ask for the call date and any additions; lodestar's
job ends at the hand-off.

If Paul says no or doesn't engage on the offer, leave it. The chat output
is the deliverable — he can copy/paste if useful. **Don't push.**

### Step 6 — Log it

Append a single entry to `~/Git/Projects/lodestar/nudges/log.jsonl`:

```json
{"date":"2026-05-02","skill":"lodestar-1on1-prep","goal":"none","action":"surfaced"}
```

This is informational — `lodestar-status` reads the log for "last 1:1 prep
ran on" awareness. Don't log the surfaced items themselves; the journal,
gh-inbox JSON, and Performance Highlights notes are the canonical record.

If Paul accepts the hand-off, append a follow-up:

```json
{"date":"2026-05-02","skill":"lodestar-1on1-prep","goal":"none","action":"acted"}
```

## Rules

- **Read-only across every source.** Never edit journal entries; never
  mutate gh-inbox state (always use `--since N-days` or `--all`, never
  default mode); never edit goals.md or 1:1 notes; never write the agenda
  file (that's `lead-call-prep`'s contract).
- **Don't auto-trigger `lead-call-prep`.** The hand-off requires explicit
  consent ("yes, formalise it" / "go ahead").
- **Empty buckets are valid.** "No blockers this fortnight" is real
  signal — don't pad it with weak candidates.
- **Be specific.** Per `nudge-tone.md`: "Stuck on Same-site migration
  ownership" beats "had some blockers." Always cite a date, a journal
  entry, a PR number, or a notes file.
- **Don't pre-rank within buckets.** The skill surfaces candidates; Paul
  picks. Order: recency-descending. Bucket order is fixed (Wins,
  Blockers, Follow-ups, Emerging).
- **Don't include private channels.** Email and Slack are out of scope
  for this skill — only journal, gh-inbox, lodestar log, goals doc, 1:1
  notes, and Performance Highlights notes.
- **No emoji or exclamation marks** in surfaced output, per
  `nudge-tone.md`.
- **Don't repeat the same item across runs.** If a candidate appeared
  in the last `lodestar-1on1-prep` run (check the previous run's log
  entry's date — if Paul invoked this skill within the last 7 days,
  consider deduplication), only re-raise if its status has changed.
