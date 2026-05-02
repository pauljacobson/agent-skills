---
name: lodestar-coaching-prep
description: >
  Surfaces candidate themes for Paul's next external coaching session with
  Nadezhda by cross-referencing recent coaching notes (most recent 2-3
  sessions) with lodestar signal: nudge log, journal entries, goals doc,
  weekly updates. Read-only across all sources; produces 4 reflective
  buckets in chat (Carried over from previous sessions, Recurring tensions,
  Areas of growth or stuckness, New territory) using the active coaching
  themes as a lens. Does NOT write a coaching prep note — that's Paul's
  job, integrating into the existing template at "Coaching session
  prep/Coaching Session Prep Template.md". Default 4-week window. Use when
  Paul says "prep my coaching session", "coaching prep", "lodestar
  coaching", or runs /lodestar-coaching-prep.
---

# lodestar-coaching-prep

Sibling to `lodestar-1on1-prep` but tuned for external coaching with
Nadezhda. Where 1:1 prep is work-tactical (who's blocking what, what
should Sarah know), coaching prep is reflective: what patterns am I
living, what do I keep avoiding, where am I growing or stuck?

This skill **surfaces signal** — it never writes the prep note itself.
Paul integrates the surfaced themes into the existing prep workflow
documented at `Notes Hub/Coaching session prep/Coaching Session Prep -
Instructions.md`, using the template at the same path.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Read `~/Git/Projects/lodestar/references/goal-mapping.md` — specifically
   the coaching-themes section if present (used to map journal mentions to
   active themes).
3. Read the canonical coaching context **on every invocation** (so theme
   changes propagate automatically):
   - `Notes Hub/Coaching session prep/Coaching Session Prep - Instructions.md`
     — has the active themes list (currently 5, as of Feb 2026) and the
     CoachHub goals.
   - `Notes Hub/Coaching session prep/Coaching Themes & Key Framings.md`
     — accumulated reframes and observations from past sessions; often
     a recurring tension shows up here as a quote worth re-raising.
4. Use `obsidian-cli` (skill: `obsidian:obsidian-cli`) for vault reads.

## Window

Default: **last 4 weeks** (28 days). Coaching cadence is roughly every 5
weeks based on the historical pattern, so 4 weeks usually captures the
full inter-session period without padding.

Accept overrides:

- `--since 6w` — wider (e.g. catching up after a longer gap or PTO)
- `--since 2w` — narrower (e.g. follow-up coaching session shortly after
  the previous one)

If the most recent file at `Notes Hub/` matching `* Coaching session
notes.md` is older than ~6 weeks, ask once: *"Last coaching session was
YYYY-MM-DD — N weeks ago. Want me to widen the window past the default
4 weeks?"* Otherwise default silently.

## Procedure

### Step 1 — Read existing coaching context

Read the prep instructions (Step 3 of the prep process — "Map progress to
coaching themes") and pull out the **active themes list**. As of Feb 2026
the list is:

1. AI Orchestration role
2. Internal visibility
3. Measurable impact
4. Career positioning
5. Architecture of work

If the instructions file lists a different set, use that — the file is
canonical. The five-themes list is the **lens** the skill uses to
categorize signal in Step 4.

Also read:
- The **Coaching Themes & Key Framings** file — pull out coach quotes
  and reframings. These often surface again as recurring tensions
  when a journal entry or nudge echoes the same pattern.
- The **CoachHub goals** at the bottom of the instructions file (career
  transition, AI-augmented adaptation, influence/visibility, stress &
  resilience, purpose & strategy). Use these as a secondary lens — too
  broad for primary categorization but useful for "New territory" items
  that don't fit a tactical theme.

### Step 2 — Read the most recent coaching session notes

List files in `Notes Hub/` matching `* Coaching session notes.md`, sort
descending by filename. Read the most recent **2-3** sessions. Pull:

- **Action items** from each session — both inline `- [ ]` items in the
  session notes and any companion file in `Coaching session prep/Action
  Items — Post YYYYMMDD Session.md`. Track which are still unchecked.
- **Topics queued for next session** — the session notes typically have
  a "Topics for next session" or "Parked for later" section. These are
  explicit carry-overs.
- **Coach observations and reframings** — Nadezhda's framings often
  point at unresolved patterns; if the same framing applies to recent
  journal entries, that's a recurring tension.

### Step 3 — Gather lodestar signal in parallel

These four reads are independent — fire them in parallel:

1. **Lodestar nudge log** —
   `~/Git/Projects/lodestar/nudges/log.jsonl`. Filter to entries within
   the window. For coaching purposes, the interesting patterns are:
   - Same goal/area surfaced repeatedly with `"action":"declined"` or
     `"surfaced"` (no follow-through) → strong signal of stuckness
   - Same goal/area surfaced repeatedly with `"action":"acted"` → growth
   - Goals that were quiet for the full window → avoidance pattern
     worth a coaching look

2. **Journal entries** — list `journal`-tagged notes in the window.
   Read each and pull bullets matching:
   - **Tensions / frustrations**: `frustrated`, `stuck`, `unsure`,
     `avoiding`, `keep meaning to`, `should but`, `tired of`, `tension
     between`, repeated phrasings of the same problem across multiple
     entries (the recurrence is the signal, not the words).
   - **Personal-development moments**: `realized`, `noticed`, `pattern`,
     `growth`, `learning`, `tried something different`, breakthrough or
     "first time I" framings.
   - **Theme-keyword mentions** (per coaching-themes section of
     `goal-mapping.md`) — count mentions per theme to see which are
     well-represented and which are quiet.

3. **Goals doc** —
   `~/Git/a8c/lead-calls-prep/references/goals.md`. Pull the "Emerging
   Areas / To Discuss" bullets. Filter to ones that touch personal
   growth or career direction (vs. purely tactical work items). Items
   like "Boot.dev course", "Snyk.io / Code Rabbit", "Session Zero idea"
   are coaching-shaped; "Same-site migration tracking" or "Hebrew
   Calendar plugin" are tactical and belong in 1:1 prep instead.

4. **Recent weekly updates** — list `weekly_update`-tagged notes in the
   window (typically 2-3 over 4 weeks). Read each and pull "Work
   Highlights ⭐️" and "My Goals, My Impact 🎯" sections. These show what
   Paul is choosing to surface publicly — useful for the **Areas of
   growth** bucket (what he's proud enough of to write up).

### Step 4 — Map signal to coaching themes

For each piece of signal gathered in Steps 2-3, tag it against the
active themes from Step 1. A single item can touch multiple themes —
that's expected (e.g. a journal entry about "wrote a P2 post about my
Claude orchestration setup" hits both AI Orchestration and Visibility).

Build a small working-memory table:

```
Theme                   | Signal count | Notable items
------------------------|--------------|------------------------------------
AI Orchestration role   | 8            | strong; multiple journal mentions, kudos
Internal visibility     | 2            | quiet; one P2 post, no division-level
Measurable impact       | 0            | absent; no metrics in any weekly update
Career positioning      | 1            | one journal entry musing about role
Architecture of work    | 0            | absent (parked from previous session)
```

Themes with zero or near-zero signal across the window are strong
candidates for **Areas of growth or stuckness** (Paul committed to
working on it, signal hasn't shown up). Themes with high signal but no
concrete progress (lots of mentions, no shipped work) suggest a
recurring tension.

### Step 5 — Group into 4 buckets and surface

Print the draft prep in chat in this order. Item ordering within a bucket
is **theme-grouped** (not strict recency) — a coaching prep is more
useful when items are clustered by what they're about:

```
### Draft coaching prep — N items across <window> weeks
Last session: YYYY-MM-DD. Active themes: <comma-separated list>.

**Carried over from previous sessions** (M items)
- <action item or "Topics for next session" entry> — <status>
  e.g. "From 26 Feb action items: 'Pick 2-3 trackable metrics' — still
  unchecked; no metrics surfaced in any weekly update since."
- <"queue for next session" item from previous session notes>
  e.g. "Parked from 26 Feb: 'engineering of what would be' / architecture
  of work — explicitly queued; no journal mention since."
- ...

**Recurring tensions** (M items)
- <pattern name> — <evidence cite>
  e.g. "Visibility-vs-tactical pull: 4 journal entries mention 'should
  write a P2 post' since 2026-04-01; only 1 was actually written. Same
  framing as Nadezhda's 'marketing of what is' from Feb session."
- ...

**Areas of growth or stuckness** (M items)
- <theme or area> — <growth or stuck signal>
  e.g. "Stuck: 'Measurable impact' theme has 0 signal across the window —
  no metrics in weekly updates, no journal mentions. This was committed
  to in Feb."
  e.g. "Growth: 'AI Orchestration' has strong signal (8 mentions, 3
  shipped pieces of work, public P2 post). Worth naming what's working."
- ...

**New territory** (M items)
- <fresh item not previously covered> — <why coaching-shaped>
  e.g. "Journal entries 2026-04-22, 2026-04-29 mention feeling 'tired of
  context-switching between Fusion and bug blitzes' — this hasn't come
  up in coaching but is recurring and personal in shape."
- ...
```

Bucket-handling rules:

- **An empty bucket gets one line, no apology.** e.g.
  *"**New territory**: nothing fresh jumped out — coaching window has been
  consistent."* That's it.
- **If everything is empty** (very rare given the 4-week window): output
  a single line and stop:
  > Nothing distinct surfaced across the last 4 weeks. Want me to widen
  > the window or read further back into coaching notes?
- **Cap each bucket at ~5 items.** If more candidates exist, surface the
  most cross-referenced (signal that touches multiple sources beats
  signal from one source) and note "+ N more candidates if you want to
  dig further."

### Step 6 — End the surface, no hand-off

Per the issue's explicit constraint: **don't write back to the vault**.
Paul integrates this into the prep note himself, following the existing
prep instructions (`Coaching session prep/Coaching Session Prep -
Instructions.md`, Step 5).

End with a single closing line, no offer to formalise:

> That's the surface — let me know if you want to dig into any of these
> further before you draft the prep note.

If Paul wants to dig deeper on a specific item, do that conversationally
(read more context, surface specific journal quotes, etc.) without ever
writing to the vault.

### Step 7 — Log it

Append a single entry to `~/Git/Projects/lodestar/nudges/log.jsonl`:

```json
{"date":"2026-05-02","skill":"lodestar-coaching-prep","goal":"none","action":"surfaced"}
```

Informational only — `lodestar-status` reads the log for "last coaching
prep ran on" awareness. Don't log the surfaced themes themselves.

## Rules

- **Read-only across every source.** Never edit coaching notes, action
  items files, journal entries, goals doc, or weekly updates. Never
  write a prep note (that's Paul's job per the existing prep
  instructions).
- **Reflective, not tactical.** This skill's tone differs from
  `lodestar-1on1-prep`. Avoid action-item language ("ship X", "decide
  Y"). Use pattern language ("you've mentioned X four times", "this
  theme is quiet", "this echoes Nadezhda's framing of...").
- **No emoji or exclamation marks.** Per `nudge-tone.md`.
- **Empty buckets are valid signal.** If "Measurable impact" has zero
  signal across 4 weeks despite being a committed theme, that's exactly
  what coaching is for — surface it explicitly rather than padding.
- **Never include private channels.** Email, Slack, and DM content are
  out of scope (the existing prep instructions list Slack threads as a
  source, but lodestar doesn't have a Slack signal lane — leave that to
  Paul's manual review per the prep instructions).
- **Don't pre-judge stuckness.** Surface evidence, not conclusions. "0
  signal in window" is fact; "you're avoiding this" is a conclusion
  Paul (with Nadezhda) gets to draw.
- **Re-read the active themes each run.** Don't cache the 5-theme list
  — read it from the prep instructions file every invocation so changes
  to themes propagate without skill edits.
- **Don't double-list.** A signal that fits both "Recurring tensions"
  and "Areas of stuckness" goes in the more pattern-shaped bucket
  (Recurring tensions).
- **Defer to the vault.** When in doubt about structure, format, or
  emphasis, look at how the existing coaching infrastructure does it
  (prep template, themes file, post-session notes). Match that idiom.
