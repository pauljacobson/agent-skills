---
name: lodestar-weekly-review
description: >
  Walks Paul through a guided GTD weekly review following the four-stage flow
  (Get Clear → Get Current → Get Creative → Wrap-Up) from his Obsidian note
  "Weekly Review based on GTD.md". Sweeps Todoist Inbox, GitHub inbox, and
  journal; processes the "To review" projects from Projects.base; checks
  Waiting-For (pending) and Someday/Maybe lists; identifies Top 3 next
  actions for the week ahead. Updates each project's `reviewed:` frontmatter
  (with confirmation) and saves a session summary to
  ~/Git/Projects/lodestar/reviews/YYYY-MM-DD.md. Use when Paul says "weekly
  review", "GTD review", "let's review my projects", or runs
  /lodestar-weekly-review. Typically Saturday morning (Paul's work week is
  Sun–Thu; Saturday is the focused planning slot).
---

# lodestar-weekly-review

Facilitates Paul's weekly review using the GTD framework documented in his
vault at `Weekly Review based on GTD.md`. Paul drives — lodestar surfaces,
prompts, and records.

## Source

The procedure below mirrors **"A Typical Weekly Review Flow"** from
`/Users/pauljacobson/Dropbox/Text notes/Notes Hub/Weekly Review based on GTD.md`:

1. **Get Clear** — capture everything (inboxes, notes, thoughts); review
   and clear inboxes.
2. **Get Current** — review calendar (past + future); review projects
   (active, inactive, waiting-for); ensure each active project has a next
   action.
3. **Get Creative** — review Someday/Maybe; consider new projects or areas
   of focus.
4. **Wrap-Up** — update Todoist with next actions; identify Top 3 for the
   week ahead; close ready and clear.

If Paul has updated the vault note, treat that as canonical and adapt the
flow accordingly.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Read `~/Git/Projects/lodestar/references/weekly-review-procedure.md` for
   the GTD prompts adapted to this workflow.
3. Use `obsidian-cli` (skill: `obsidian:obsidian-cli`) for vault access.

## Procedure

The whole review usually fits in 60-90 minutes. If Paul has limited time,
offer a partial pass — Get Clear is the highest-value bit; Get Current and
Get Creative can be deferred.

---

## Stage 0 — Preflight

A ~30-second pass before Stage 1 to probe every external source the review
depends on, surface their availability up front, and let Paul opt
in/out per source. The point: avoid mid-review surprises like "gh isn't
authenticated" (silent zero-result Stage 1 sweep) or "no `gmail-rules.md`
yet" (unexpected 30-minute exploration sub-flow at Step 4).

This stage was added after the 2026-05-02 first-run review, where both
problems above hit mid-Stage-1. See that review's session summary in
`~/Git/Projects/lodestar/reviews/2026-05-02.md`.

### Step 0 — Probe each source

Fire these probes **in parallel** — most are cheap, and the slow ones
(MCP connector pings) overlap fine:

| Source | Probe | Green | Yellow | Red |
|---|---|---|---|---|
| Obsidian vault | check `config.md` vault root path exists | path exists | — | path missing |
| Todoist | `td --version` (or first `td inbox --json` line) | command succeeds | — | command fails or auth error |
| Journal | count of `journal`-tagged notes in last 7 days via obsidian-cli | ≥1 entry | 0 entries (vacation? PTO?) | obsidian-cli error |
| GitHub inbox | `gh auth status` and existence of `~/.claude/skills/gh-inbox/scripts/fetch.sh` | both succeed | script missing but `gh` ok | `gh auth` not authenticated |
| Gmail | `~/Git/Projects/lodestar/references/gmail-rules.md` exists AND Gmail connector reachable (`list_labels` returns) | both true | rules missing (exploration mode would fire) OR connector ok but no rules | connector unreachable |
| Calendar | Calendar connector `list_calendars` returns | succeeds; primary calendar present | succeeds but multiple calendars (need scope choice) | connector unreachable |
| Performance Highlights | list `Notes Hub/knowledgemattic/* Performance Highlights.md`, sort desc | most recent within 14 days | most recent older than 14 days OR none yet | obsidian-cli error |

Don't fail the review on any individual probe error — the worst probe
case is "skip this source today," not "abort everything."

### Step 1 — Report and choose

Render a compact table with one line per source. Use `✓` / `⚠` / `✗`
markers (no emoji elsewhere in the review, but the preflight is the
exception — the markers carry useful information at a glance). Example:

```
### Preflight — sources
✓ Obsidian vault       — Notes Hub reachable
✓ Todoist              — `td` authenticated, 2 in Inbox
✓ Journal              — 8 entries in last 7 days
✗ GitHub inbox         — `gh` not authenticated; Stage 1 Step 2 will return 0
⚠ Gmail                — no gmail-rules.md yet; exploration mode would fire (~30 min)
⚠ Calendar             — connector ok; 4 calendars detected (need scope)
- Performance Highlights — none yet (expected; workflow hasn't run)

Two yellow, one red. For each non-green source, choose: skip / fix / abort.
```

Then prompt **once per non-green source**, in this order: red sources
first (they're most likely to derail something), then yellow:

> **GitHub inbox** — `gh` not authenticated. Options:
> - **skip** — bypass Stage 1 Step 2 today; the session summary will
>   record this as "GitHub sweep skipped (gh not authenticated)"
> - **fix** — pause here while you run `gh auth login`; resume when ready
> - **abort** — bail on the whole review

Capture Paul's choice. Hold it in working memory (the conversation) as a
**skip flag** keyed by source name, e.g.:

```
preflight_skip = {
  "github_inbox": True,
  "gmail": True,
  "calendar": False,  # not skipped, but with a scope choice
}
preflight_calendar_scope = ["primary"]  # or ["primary", "Automattic"]
```

For Calendar specifically, when the probe returns multiple calendars:

> **Calendar** — 4 calendars detected: primary, Automattic, Family,
> Public Holidays. Default for the review is primary. Want to include
> any of the others?

Default to primary if Paul says no/skip; otherwise add to
`preflight_calendar_scope`. This replaces the "ask once" mid-Stage-2
calendar-scope question described in `config.md` § Google Calendar.

### Step 2 — Confirm and start

After all non-green sources have a choice, summarise once:

```
Starting the review with:
- Todoist, Journal, Calendar (primary + Automattic), Performance
  Highlights
- Skipping: GitHub inbox, Gmail
```

Then proceed to Stage 1.

### Honoring skip flags downstream

Each Stage 1+ source-step opens with a check:

```
If preflight_skip[<source>] is True:
  silently skip; move to the next step. Do NOT print "skipping..." —
  that creates noise. The omission is recorded in Step 11 (Session
  summary) under "Sources skipped this session."
```

Specifically:
- **Stage 1 Step 2 (GitHub inbox)** — if `github_inbox` skipped, omit.
- **Stage 1 Step 4 (Gmail)** — if `gmail` skipped, omit.
- **Stage 2 Step 5 (Calendar)** — if `calendar` skipped, omit; otherwise
  use `preflight_calendar_scope` to drive `list_events` calls.

Stages 1 Step 1 (Todoist), Step 3 (Journal), and all Stage 2+ vault-only
steps don't need skip flags — those sources are local and reliable.

### Skip-aware Step 11

The session summary written to
`~/Git/Projects/lodestar/reviews/YYYY-MM-DD.md` should include a brief
**"Sources skipped this session"** line if any skips occurred, naming
each skipped source and the reason captured during preflight. This makes
the review log honest about scope.

---

## Stage 1 — Get Clear

Process every inbox before assessing projects. The point: when you review
projects in Stage 2, you're already holding a complete picture of what's
incoming.

### Step 1 — Todoist Inbox sweep

The Todoist Inbox is the GTD "stuff pile." Process **one at a time, not
bulk** — bulk-processing defeats the conscious-processing purpose.

Run `td inbox --json` (via the `todoist` skill). Sort by added-date,
oldest first. Show the count and start:

> You have N tasks in your Todoist Inbox. Let's process them one at a
> time, starting with the oldest.
>
> 1. "Email Brandon about v2 review" — added 8 days ago
>    What's the next action: assign to a project, defer, delete, or
>    leave?

For each item, four action paths:

- **Assign to project** — ask which Todoist project (or which vault
  project, if it should become a `- [ ]` line in a project note instead).
  Defer the move to the batched-writes step.
- **Defer** — ask when (e.g. "next week", "2026-05-15"). Confirm:
  "Defer '...' to YYYY-MM-DD? [y/n]" before the write.
- **Delete** — explicit confirmation per item: "Delete '...' from Todoist?
  [y/n]". Never bulk-delete.
- **Leave** — skip; the task stays in Inbox for the next sweep.

If 10+ items, offer: "That's a lot. Process the oldest 5 today and pick up
the rest next week?" Don't push to clear everything — that's the failure
mode that makes people abandon the review.

If empty:
> Todoist Inbox is empty. Nice work.

**Inbox must not contain vault-synced tasks.** Per `config.md`,
`lodestar-sync-todoist` writes to a dedicated `Lodestar` project, never
Inbox. If you find vault-synced tasks here (e.g. their text matches the
`<task> [<project name>]` format), do NOT auto-move them. Surface them and
ask: "These look like vault-synced tasks in Inbox — is your sync
destination misconfigured?"

### Step 2 — GitHub inbox sweep

Run `gh-inbox` in **non-mutating mode** so Paul's own `/gh-inbox` triage
state isn't consumed:

```bash
~/.claude/skills/gh-inbox/scripts/fetch.sh --since 7d
```

Parse the JSON. Group by category (Assigned / Body mentions / Comment
mentions) and surface as candidates for vault project tasks or new
opportunities:

> Last 7 days from your GitHub inbox: 4 assignments, 6 mentions. Any of
> these look like they should become a task in an existing project, or a
> new opportunity to capture?

For each item:
- **Task in existing project** → ask which; queue the edit for batched
  writes.
- **New opportunity** → suggest running `/lodestar-capture-opportunity`
  after the review (don't trigger QuickAdd mid-review).
- **Ignore** → no-op; item stays in the GitHub inbox for separate
  `/gh-inbox` flow.

If 0 items: "GitHub inbox is quiet for the last 7 days." Move on.

If `partial_failures` is non-empty: "Note: N gh-inbox queries failed —
sweep may be incomplete." Continue.

### Step 3 — Journal sweep for unrecorded commitments

Read the last 7 days of `journal`-tagged notes. Look for verb phrases that
suggest commitments not reflected in any project note: "I should...",
"need to...", "going to...", "Sarah suggested...". Surface 0-3 candidates:

> While reviewing journal entries, I noticed these don't appear in any
> project: [list]. Want to add any?

### Step 4 — Gmail sweep

Paul's email is messy. The skill has two modes:

#### First run — exploration

If `~/Git/Projects/lodestar/references/gmail-rules.md` does not exist,
enter **exploration mode** before doing any sweep. The point: figure out
together what's worth surfacing in weekly reviews and what to ignore.
Capture the decisions in `gmail-rules.md` so future reviews apply them
automatically.

Open with:

> Your email is new territory for me. Let's spend a few minutes figuring
> out what's worth surfacing in weekly reviews and what to ignore — your
> email is messy enough that I shouldn't guess. I'll search a few
> categories; we'll decide together what stays and what goes.

Run a sequence of probe searches via the Gmail connector's
`search_threads` (newest first, modest result count per probe):

1. `is:unread newer_than:7d in:inbox` — unread last week
2. `is:starred is:unread newer_than:30d` — starred-but-unhandled
3. `(feedback OR review OR "pull request" OR PR) newer_than:7d in:inbox -from:noreply -from:no-reply` — likely action requests
4. `to:paul.jacobson@a8c.com newer_than:7d -list:* -from:noreply` — direct mail (not list traffic)
5. `subject:(invitation OR rescheduled OR canceled) newer_than:14d` — calendar churn (some overlaps with calendar but worth flagging)
6. `category:promotions OR list:* newer_than:7d` — newsletters / list traffic (likely exclude)

For each probe:
- Show the count and 3-5 sample subject lines (sender + subject, no body
  excerpts unless asked — keep things compact).
- Ask: "Worth surfacing in future weekly reviews, or skip?"
- For "worth surfacing": ask if there's a refinement (narrower query, time
  window) that would make it more signal-heavy.
- For "skip": confirm and move on.

After the probes, propose a draft `gmail-rules.md` and confirm before
writing:

```markdown
# Gmail rules for lodestar weekly review

Captured during the first weekly review on YYYY-MM-DD. Edit by hand to
refine over time.

## Include — surface in weekly reviews

### Direct PR/feedback requests
- Query: `(feedback OR "pull request" OR PR) newer_than:7d in:inbox -from:noreply`
- Why: real action items from people
- Surface as: candidates for project tasks

### Starred-but-unhandled
- Query: `is:starred is:unread newer_than:30d`
- Why: Paul flagged these as important but didn't act
- Surface as: triage list

## Exclude — never surface

### Newsletters and list traffic
- Query match: `category:promotions OR list:*`
- Why: marketing/list noise; Paul ignores during reviews
```

Once written, proceed with the regular sweep using the new rules.

#### Subsequent runs — apply rules

If `gmail-rules.md` exists, read it. Run each "Include" query via
`search_threads`. Surface results grouped by category, oldest-first per
category:

> Last 7 days from Gmail (3 categories, N total threads):
>
> **Direct PR/feedback requests** (4 threads)
> 1. <sender> — <subject>
> 2. ...
>
> **Starred-but-unhandled** (2 threads)
> 1. ...

For each thread Paul wants to act on, the same options as the GitHub
sweep:
- **Task in existing project** → defer to batched writes (Step 10).
- **New opportunity** → suggest `/lodestar-capture-opportunity` after the
  review.
- **Reply now / draft now** → suggest doing it after the review (don't
  draft mid-flow).
- **Ignore / archive** → no-op; the email stays in Gmail. Lodestar does
  not archive or delete email.

If a category returns 0: write one line "No new threads in <category>"
and move on.

**Read-only**: do NOT use `create_draft`, `create_label`, or any other
mutating Gmail tool from this skill.

---

## Stage 2 — Get Current

With inboxes clear, now look at calendar, projects, and waiting-for.

### Step 5 — Calendar review (past + upcoming)

Read directly from Paul's **Google Calendar connector** (MCP). Pull two
windows:

1. **Past 7 days** — completed/past events. Surface anything that may have
   created a commitment not yet captured (e.g. "Sarah suggested X in the
   1:1 on Tuesday").
2. **Upcoming 7 days** — scheduled events. Surface deadlines, meetings,
   blocks of focus time that should shape this week's Top 3.

Use the calendar connector's `list_events` tool. Default to Paul's primary
calendar; if he has multiple work calendars and an item should obviously
come from a different one (e.g. an Automattic-specific calendar), pull
that too. Use `list_calendars` once if you need to discover what's
available.

**Read-only**: do NOT create, modify, or delete events in this step.
Calendar writes are out of scope for the weekly review.

Surface a compact summary, not a wall of detail:

```
Past 7 days:
- Mon — 1:1 with Sarah (1h)
- Tue — Team meetup planning (45m)
- Thu — Codex office hours (30m)

Upcoming 7 days:
- Tue 16:00 — 1:1 with Sarah
- Wed all-day — Team workshop
- Fri — Public holiday
```

Then prompt:

> Anything from the past week that surfaced a commitment we haven't
> captured yet?
> Anything in the upcoming week that should shape this week's Top 3?

Capture any items raised; defer writes to Step 10 (batched writes).

**If the calendar connector is unavailable or errors:** fall back to a
manual prompt — ask Paul to scan his own calendar and answer the same
two questions. Note the failure briefly: "Couldn't reach the calendar
connector — let's do this from your calendar app instead." Don't block
the review on it.

### Step 5b — Frontmatter audit

Before the per-project pass, sweep every project-tagged note's
frontmatter and surface drift. The point: in the 2026-05-02 first-run
review, 5 projects with non-canonical statuses + empty `reviewed:`
slipped past the "To review" query because malformed YAML tripped the
parser, and 21 of 89 projects (24%) used non-canonical `status:` values
overall. The skill never reported any of this — it just silently
under-counted the queue.

This step makes drift visible up front and offers normalisation in a
single batched pass, so the per-project pass that follows operates on a
clean queue.

#### Probe

Query the vault for all notes with `tags: [projects]` (use
`obsidian-cli` — read-only at this stage). For each, parse the
frontmatter and check three failure modes:

1. **Non-canonical `status:`** — the value is set but not in the
   canonical list from `config.md` (`inprogress` / `pending` /
   `complete` / `completed` / `shelved` / `someday_maybe` / empty).
   Read the canonical list from `config.md` at runtime; do not hardcode
   it here — if the canonical list changes, this audit follows
   automatically.

2. **Malformed YAML** — frontmatter doesn't parse cleanly. Common shape
   from the 2026-05-02 retro: collapsed `reviewed:`/`status:` lines
   like `status: priority:` (one line instead of two), stale `due:`
   values formatted as bare strings, or template placeholders never
   filled in.

3. **Zombie notes** — *all* of: empty `status:`, empty `reviewed:`, and
   `due:` either empty or older than 365 days from today. Empty status
   alone is canonical (config.md allows it), but the combination signals
   a project note that was created and never iterated on.

Each project may hit zero, one, or multiple categories; tag them
distinctly.

#### Surface

Render a compact summary, not a wall of detail:

```
### Frontmatter audit
89 project notes scanned.

Non-canonical status (21 notes, 5 distinct values):
  • `active` (14 notes)        → suggested map: `inprogress`
  • `done` (4 notes)            → suggested map: `completed`
  • `paused` (1 note)           → ambiguous: `pending` or `shelved`?
  • `idea` (1 note)             → suggested map: `someday_maybe`
  • `dropped` (1 note)          → suggested map: `shelved`

Malformed YAML (3 notes):
  • Improve the UI for the Linear SoT API interface
  • Personal Diabetes Web App
  • Lead 1 on 1 preparation workflow

Zombie notes (4 notes — empty status + empty reviewed + stale/empty due):
  • Old project A (due: empty, no edits since 2024-09-26)
  • Old project B (due: 2024-01-15, ~16 months stale)
  • ...
```

If all three categories are empty, write one line:
> Frontmatter audit clean — 89 projects, no drift detected.

…and proceed straight to Step 6.

#### Normalise non-canonical statuses (batched)

If the audit found non-canonical values, propose a mapping table. For
unambiguous mappings, suggest the canonical target. For ambiguous ones,
ask:

> `paused` (1 note: "Hebrew Calendar plugin") — should this map to
> `pending` (waiting on something), `shelved` (set aside intentionally),
> or stay as-is?

Once Paul has confirmed the mapping for all distinct values, present
the **single batched confirmation**:

```
About to normalise statuses:
  • 14 notes: `active` → `inprogress`
  • 4 notes:  `done`   → `completed`
  • 1 note:   `paused` → `shelved`
  • 1 note:   `idea`   → `someday_maybe`
  • 1 note:   `dropped` → `shelved`
Total: 21 notes.

Proceed?
```

Only on explicit "yes" / "proceed" / equivalent, run the writes via
`obsidian property:set status <value>` per note (per the
`feedback_obsidian-cli-for-frontmatter` memory: prefer
`obsidian property:set` over regex/Edit — preserves YAML and is
Obsidian-aware). Never partial-write.

If Paul wants to override individual notes (e.g. "actually, the one
`paused` note should stay as `pending`"), accept the override and
re-render the confirmation with the change.

#### Repair malformed YAML (per-file with diff)

For each note with malformed YAML, treat it individually — these are
not safe to batch. For each:

1. Read the current frontmatter as raw text.
2. Propose a repaired version, preserving every value the malformed
   form was carrying (don't silently drop fields).
3. Show a diff:

```
Personal Diabetes Web App — frontmatter repair

  -reviewed: status: inprogress
  +reviewed: 2026-04-12
  +status: inprogress
   due: 2024-09-26  ← stale; clear?
   priority:
```

4. Ask: *"Apply this repair? Optionally clear the stale due: too?"*
5. On yes, write via `obsidian property:set` (one property at a time
   — `obsidian property:set` doesn't take whole-frontmatter writes; if
   the malform is severe enough that property-set won't reach into it,
   fall back to a single full-frontmatter rewrite using `Edit`, but
   show the full diff first).

Don't insist on repair — if Paul wants to skip a file, skip it. The
audit will surface it again next week.

#### Surface zombies for review (no auto-mutation)

Zombies don't get an auto-mapping. Surface them as a brief list and
ask: *"Any of these you want to handle now (shelve, complete, delete),
or leave for the per-project pass?"*

Each can branch:
- **Shelve / complete / mark someday_maybe** → defer to batched writes
  (Step 10).
- **Delete the note entirely** → propose, but require explicit
  per-file confirmation. Use `trash` rather than `rm`.
- **Leave for later** → skip; the note will re-surface next audit.

#### Record what happened

If any normalisations, repairs, or zombie-decisions happened, the
session summary (Step 11) records them under "Frontmatter audit". If
the audit was clean, the line is omitted from the summary.

### Step 5c — Mode selection (when N > 8)

Count items matching the **"To review"** filter (`Bases/Projects.base`
lines 25-48). If `N ≤ 8`, skip this step — go straight to Step 6 in
deep-dive mode (the card flow). If `N > 8`, present the three modes
explicitly **before** any per-project work begins.

The 60-90s-per-project budget for the card flow caps at ~8 projects in
a 60-min slot; beyond that, scope has to be chosen consciously, not
discovered mid-flow. The 2026-05-02 first-run review had 36 in queue
and the triage/deep-dive split was decided mid-pass — that decision
belongs at the top of Stage 2.

#### Compute time estimates

For the prompt below, compute estimates from N at runtime:

- **Triage**: ~15 seconds per project. Round to nearest 5 min.
- **Deep-dive**: ~75 seconds per project (midpoint of 60-90s budget).
  Round to nearest 5 min.
- **Triage-then-deep-dive**: triage time + deep-dive time on roughly
  20-30% of N (calibrated from 2026-05-02: 36 → 6 active = 17%; for
  better-maintained queues expect higher retention). Use **30%** as the
  default deep-dive subset for the estimate; note in the prompt that
  the actual subset depends on triage outcomes.

#### Present the choice

```
N projects in "To review" — beyond the ~8 you can deep-dive in a
60-90s-per-project budget. How do you want to scope today?

A. Triage only (~<estimate> min)
   Per project: keep / shelve / someday / pending / complete. No card,
   no GTD prompts. Best when the queue's been ignored a long time and
   most can be cleared with a quick judgement.

B. Deep-dive only (~<estimate> min)
   Full card per project with the GTD prompts. Best when you want to
   think carefully about each one. Will run long if N is large.

C. Triage then deep-dive (~<estimate> min — roughly <ceil(N×0.3)>
   deep-dives after triage)
   Triage all N first; deep-dive only what stays active. Best when you
   suspect most can be cleared but want to think harder about the
   active subset. (This is what worked on 2026-05-02 — 36 → 6 active.)

Which?
```

Capture the choice as `stage2_mode` in working memory:

- `triage_only` → Step 6 runs in triage form for all N, then jumps
  past the deep-dive flow.
- `deep_dive_only` → Step 6 runs the existing card flow for all N. If
  Paul still wants to cap (e.g. "do the most-overdue 4"), offer that
  *after* the mode is locked, not before.
- `triage_then_deep_dive` → Step 6 runs triage form for all N first;
  on completion, computes the still-active subset (status =
  `inprogress` or empty after triage) and re-runs the card flow on
  just those.

#### Triage-form per-project flow

Render a **minimal card** — title, status, last-reviewed date. No top
tasks, no notes excerpt:

```
[3 of 36] — Block plugins workflow
Status: inprogress  ·  Last reviewed: 2026-04-12 (18 days ago)

Keep / shelve / someday / pending / complete?  (default: keep)
```

Recognised responses (case-insensitive, single-letter shortcuts):

- empty / `k` / `keep` / Enter → no status change; mark reviewed; advance
- `s` / `shelve` → status → `shelved`; mark reviewed; advance
- `m` / `maybe` / `someday` → status → `someday_maybe`; mark reviewed; advance
- `p` / `pending` → status → `pending`; mark reviewed; advance.
  Optionally ask: *"What are you waiting on?"* — if Paul answers, capture
  to the project's `## Notes` section (deferred to Step 10 batched writes).
- `c` / `complete` → status → `completed`; mark reviewed; advance
- `?` or any longer free text → **escape into deep-dive form for this
  one project**. Render the full Step 6 card; complete the deep-dive flow
  for it; then return to triage at the next item. This means triage mode
  doesn't lock you in — if a project needs more thought, you dip into
  the card just for that one without abandoning the run.

The pace target is ~10-15 seconds per project. If Paul stalls on a
single item, *don't prompt the GTD questions* — that's what the escape
is for. Stay in triage until he uses it.

All status changes from triage are deferred to Step 10 batched writes,
same as the card flow. The session summary (Step 11) records the mode
used and the triage outcomes (how many cleared, how many stayed
active).

#### Triage-then-deep-dive transition

After the triage pass completes, before the deep-dive pass starts,
surface a one-line summary:

> Triage complete: 36 → 8 still active (3 shelved, 18 someday, 7
> completed). Now deep-diving the 8.

Then proceed to Step 6's card flow on the 8.

If the still-active subset is 0 (everything cleared), output:

> Triage complete: nothing left active. Skipping deep-dive.

…and proceed to Step 7.

### Step 6 — Per-project pass ("To review" projects)

This step runs in one of two forms based on `stage2_mode` set in Step 5c:

- **Triage form** — see Step 5c. Skip the card-flow detail below.
- **Deep-dive form** (default when N ≤ 8 or `stage2_mode = deep_dive_only`,
  and the deep-dive portion of `triage_then_deep_dive`) — the card flow
  documented below.

For triage-then-deep-dive, when this step runs, it operates on the
post-triage active subset, not the original N.

#### Deep-dive form

Query the vault for projects matching the **"To review"** filter (defined
in `Bases/Projects.base` lines 25-48). Sort by `reviewed:` ascending
(oldest first). Group by `status:` per the base view definition.

> N projects need review, sorted by oldest review first. Most-overdue is
> "X" (last reviewed YYYY-MM-DD). Let's start there.

(The "N > 8 — that's a lot, do the most-overdue 4?" prompt is no longer
needed here — Step 5c handles upfront mode selection. If Paul is in
deep-dive mode and *still* wants a cap mid-flow, he can say so and the
remaining will defer to a future review.)

**Flow modelled on OmniFocus's "Review" perspective**: one project at a
time, full context visible, default closing action is "review and
advance." The point is to remove clicks and decision overhead — Paul
sees a project, makes 1-3 small decisions, hits "next."

For each project, in order, render a **compact card** in this shape:

```
[3 of 8] — Block plugins workflow
Status: inprogress  ·  Due: 2026-05-15  ·  Last reviewed: 2026-04-12 (18 days ago)

Top tasks (3 of 7 unchecked):
  • Email Brandon about v2 review timeline
  • Test the localisation fix on staging
  • Draft P2 post about the workflow

Notes (last entry, truncated):
  > Brandon happy with v1 output; v2 should narrow scope to migration cases…
```

Then ask **one consolidated GTD question** — pick 1-3 prompts based on
project state, but ask them as a single block, not sequentially:

> Quick check on Block plugins workflow:
> - Still active, or shelf/someday?
> - The top task ("Email Brandon...") — is that the very next action?
> - Anything to capture or change?
>
> Or just say "next" to mark reviewed and move on.

Default action is **"next"** = mark reviewed (today's date) and advance.
That's the OmniFocus one-tap pattern: most projects don't need
discussion, they just need a quick scan and a forward motion.

Other concise responses to recognise:
- `next` / `n` / `done` / `looks good` → mark reviewed, advance
- `skip` / `s` → advance WITHOUT updating `reviewed:` (e.g. Paul wants
  to come back to this one later)
- `<free text>` → treat as notes/decisions; capture, then ask "anything
  else, or next?"
- `defer` → ask when, then advance without review-mark
- `shelf` / `someday` → propose status change, advance

Per-project rules of thumb:
- If untouched 4+ weeks → suggest shelved/someday_maybe in the prompt
- If the top task looks vague → flag it ("'Look into Brandon stuff' —
  want a concrete next step?")
- If pending status implies waiting on someone → confirm who/what

If Paul wants to promote the next action to Todoist now, ask which
Todoist project (never default to Inbox). Defer the actual write to the
batched-writes step (Step 10).

**Don't go deep by default.** 60-90 seconds per project is the target;
if Paul wants to think out loud on one, give him room, but the prompt
shape should make "next" feel as natural as "stop and discuss."

**Record decisions in memory** (don't write to files yet — batch in
Step 10).

**Locate the `## Tasks` section using the rule below**, not the naive
"first match," so projects that embed the template inside a code fence
(e.g. the lodestar project note) don't trip you up.

### Step 7 — Waiting-For check

Query the vault for projects with `status: pending` (use the **"Pending"**
view in `Bases/Projects.base`).

> N projects are waiting on someone or something. Let's check them
> quickly:
>
> 1. "Block plugins workflow" — pending since YYYY-MM-DD, waiting on
>    Brandon (per Notes section). Still waiting? Need to follow up?

For each, two options:
- **Still waiting** → no change.
- **Follow up needed** → capture a task to ping the person (defer to
  batched writes); optionally update the project's Notes section with the
  follow-up date.
- **No longer pending** → ask whether to move back to active or to
  shelved/complete.

If 0 pending projects: skip silently. No pep talk required.

---

## Stage 3 — Get Creative

### Step 8 — Someday/Maybe review

Query the vault for projects with `status: someday_maybe` (use the
**"Someday/Maybe"** view in `Bases/Projects.base`).

> N projects are on Someday/Maybe. Let me read the titles —
> anything calling out for activation, given what's coming up this week?

List the titles. For each one Paul flags:

- **Activate now** → propose status change from `someday_maybe` to
  `inprogress` or empty (defer to batched writes). Ask for the first next
  action.
- **Keep dormant** → no change.
- **Drop entirely** → propose status change to `shelved` (or whatever
  Paul's "this is dead now" convention is) with confirmation.

Also a fresh prompt:

> Anything new on your mind that doesn't have a project yet — areas of
> focus, things you've been thinking about but haven't captured?

For yes responses, defer to `/lodestar-capture-opportunity` after the
review (don't trigger QuickAdd mid-review).

If 0 Someday/Maybe projects and nothing new on Paul's mind, skip silently.

---

## Stage 4 — Wrap-Up

### Step 9 — Top 3 next actions for the week ahead

This is the whole point — leaving the review with clarity on what to do
next.

#### Two forms recognised

A Top 3 slot is allowed to be **either**:

1. **A discrete physical task** — one specific next action ("Send blood
   test results to Dr Perl"). The classic Top 3 shape.
2. **A joint area priority** — one slot covering multiple issues or
   tasks under one focus area, typically tracked across one or more
   vault project notes ("Lodestar work — issues #2, #7, #8 + capture
   Dotcom Bug Blitz"). The 2026-05-02 retro showed this form is real
   and useful — sometimes the Top-3 priority is "make progress on this
   focus area" rather than a single physical task.

The prompt explicitly invites both:

> Given everything we've just looked at: what are your **Top 3
> priorities** for this week? Each can be either a single physical
> next action, or a joint area priority covering multiple issues under
> one focus.

For joint-area items, ask Paul to name the area and list the
constituent issues / tasks / project notes:

> "Lodestar work" — what's covered? (e.g. specific issues, project
> notes, or task lines)

Capture both shapes uniformly: each Top 3 entry has a title and an
optional list of constituents.

#### Per-item destination

For **each** of the 3 items, ask the destination separately — these
choices vary item-by-item:

> "Send blood test results to Dr Perl" — push to Todoist (which
> project?) or track in a vault project note?

Three destinations:

- **Todoist** — Paul names the destination project (per `config.md` §
  Todoist: never Inbox; never default; the choice depends on work area).
  Add a `@top3` label or due-today flag if Paul has a convention.
  Defer the actual write to Step 10 batched writes.

- **Vault project note** — when the work is already tracked in a vault
  project note + GitHub issues and Paul will action it via Claude Code
  or directly in Obsidian, no Todoist mirror is needed. This is the
  natural form for joint-area priorities, but a discrete task can also
  go here if Paul prefers.

  For this destination:
  1. Confirm the destination project note exists (read its title).
  2. For joint-area items, list the constituent issues/tasks Paul
     named. Check whether each is already represented in the project
     note's `## Tasks` section. For any that aren't, offer to add them
     as `- [ ]` entries (deferred to Step 10 batched writes).
  3. For discrete tasks going vault-only: same — confirm the line is
     in `## Tasks` or offer to add it.
  4. **No Todoist write.** Do not duplicate; the vault is the canonical
     record for these.

- **Both** — track in vault *and* push a single representative line to
  Todoist (e.g. for joint-area items, push the area title only as a
  visibility nudge; the constituents stay in the vault). Use sparingly
  — duplication is the Todoist anti-pattern lodestar exists to avoid.
  Confirm explicitly: *"Push 'Lodestar work' as a single Todoist line
  for visibility, with constituents tracked in the vault?"*

#### Defaults and prompts

- **Default destination is "ask"** — don't auto-push. The 2026-05-02
  retro found the auto-push prompt clumsy when the natural answer was
  vault-only.
- **Discrete physical tasks** lean toward Todoist (matches the
  next-action ergonomics).
- **Joint area priorities** lean toward vault-only (the issues are
  already tracked there; mirroring would create the parallel-list
  problem `lodestar-sync-todoist` is designed to avoid).
- **Don't pre-decide.** Ask per item, even when the lean is obvious.

The Top 3 get recorded in the session summary (Step 11) showing both
title and destination per item.

### Step 10 — Batched writes (with confirmation)

After all stages, summarise pending writes as a single confirmation
block:

```
About to update:
- 6 projects: reviewed → 2026-04-30
- "Block plugins workflow": status → pending (waiting on Brandon)
- "Training Simulator": new task added — "Scan TS issues this week"
- Todoist: 2 Inbox items deferred, 1 deleted, 3 assigned to projects
- Top 3 added to Todoist

Proceed?
```

Only write on explicit "yes" / "proceed" / equivalent. If Paul says no or
modifies, redo the confirmation. **Never partial-write.**

### Step 10b — "To review" invariant check

After batched writes commit, re-query the **"To review"** filter (same
filter Step 6 used; defined in `Bases/Projects.base` lines 25-48).
**The invariant**: a completed weekly review leaves "To review" empty.
This step verifies the invariant before the session summary records
results.

In the 2026-05-02 first-run review, 5 projects remained in "To review"
after the entire flow finished — discovered only as a follow-up
question. The skill should catch this before Step 12.

#### Probe

Re-run the "To review" filter against the vault. Capture:

- `to_review_remaining` — count of items still matching
- For each remaining item, attempt to detect the cause (best-effort —
  these are heuristics, not certainties):
  - **Non-canonical status** — Step 5b audit either wasn't run or Paul
    declined the mapping; the project's status still doesn't match the
    canonical list.
  - **Empty `reviewed:`** — a Stage 2 write didn't land. Possible
    reasons: malformed YAML still present, the property write returned
    an error that wasn't caught, the project was added to the queue
    *after* the per-project pass started.
  - **New project** — note's `created` (filesystem mtime or frontmatter
    `created:` if present) is during today's session window. Likely
    captured via `lodestar-capture-opportunity` or manually in Obsidian
    while the review was running.
  - **Cause unclear** — none of the above heuristics fit; flag and
    surface anyway.

#### If invariant holds (count = 0)

Capture `to_review_invariant_held = True` in working memory. Proceed
silently to Step 11. The session summary records "To review queue:
empty ✓" on a single line.

#### If invariant doesn't hold (count > 0)

Surface the remaining items with the detected cause:

```
"To review" still has 5 items after batched writes:

Likely cause: empty reviewed:
  • Personal Diabetes Web App — reviewed write didn't land (YAML still
    has malformed `reviewed:` after Step 5b repair was declined)
  • Lead 1 on 1 preparation workflow — same shape

Likely cause: new project (added during this session)
  • Dotcom Bug Blitz — note created today at 10:42

Likely cause: non-canonical status
  • Hebrew Calendar plugin — status `paused` (declined mapping in
    Step 5b)

Cause unclear:
  • Improve my GTD-productivity process — has reviewed: 2026-05-02 but
    still matches the filter

Process these now as a final batch (triage form), or leave for next
review?
```

For each branch:

- **"process now" / "yes" / "do them"** → run the triage form (Step 5c)
  on the remaining items. After Paul makes status decisions, treat
  those as a fresh round of pending writes — return to Step 10 for a
  batched-writes confirmation, then re-enter Step 10b at the top.

- **"leave for next review" / "skip" / "no"** → capture
  `to_review_invariant_held = False` with the count and reasons.
  Proceed to Step 11. The session summary records the residual count
  and the reasons.

- **Per-item disposition** (e.g. "process these 3, leave 2") → run
  triage on the 3, leave the 2; capture both outcomes.

#### Loop guard

If Step 10b runs more than **twice in a single session** and the
invariant still doesn't hold, stop looping. Surface:

> The invariant didn't hold after retry — N items still in queue.
> Possible causes: a malformed YAML write that's failing silently, or a
> filter condition that's tripping after each write. Leaving these for
> next review; the session summary will note the issue.

…and proceed to Step 11. This prevents an infinite loop on edge cases
where a write keeps failing for a reason the skill can't auto-resolve
(e.g. an Obsidian sync conflict mid-session).

#### Cross-references

- The **non-canonical-status** branch is connected to Step 5b
  (Frontmatter audit). If items are showing up here with non-canonical
  statuses that *weren't* surfaced in Step 5b, that's a Step 5b bug —
  log it as a "Lodestar's note" in the session summary.
- The **new project** branch is the cleanest case: a project created
  mid-session is legitimately *not* part of today's review. Default
  disposition for new projects is "leave for next review" unless Paul
  wants to triage them now.

### Step 11 — Session summary

Write to `~/Git/Projects/lodestar/reviews/YYYY-MM-DD.md`:

```markdown
### Weekly review — YYYY-MM-DD

**Reviewed**: N projects (Stage 2). Mode: <triage-only / deep-dive /
triage-then-deep-dive: M triaged → K deep-dived>
**Inbox processing**: Todoist N → 0/M, GitHub N items, journal N
commitments captured (Stage 1)
**Sources skipped this session** (omit line if none): GitHub inbox (gh
not authenticated), Gmail (no rules file yet)
**Frontmatter audit** (omit line if clean): N statuses normalised, N
YAML repairs, N zombie decisions
**"To review" queue**: empty ✓  *or*  N remaining (reasons: <reasons>)

#### Top 3 for the week ahead
Each item: title — destination (Todoist project / vault project / both).
Joint-area items list constituents inline.

1. <title> — <destination>
   *(if joint-area: constituents: <issues / project notes / task lines>)*
2. ...
3. ...

#### Status changes
- ProjectName: oldStatus → newStatus (reason)

#### New tasks captured
- ProjectName: "physical action phrase"

#### Waiting-For follow-ups
- ProjectName: ping Person about X

#### Someday/Maybe activations
- ProjectName: activated; first action: "..."

#### Flagged for follow-up
- (anything Paul wanted to revisit later)

#### Goal signal this week
One paragraph distilled from the **most recent Performance Highlights
note** (generated by `personal-weekly-update`; see `config.md` →
"Performance Highlights notes"). Surface which goals had strong signal
this week (highlighted journal entries, kudos, CSAT) and which were
quiet. Do not regenerate the aggregation here — `personal-weekly-update`
owns that workflow; lodestar only reads and summarises.

Lookup procedure:
1. List files in `Notes Hub/knowledgemattic/` matching the pattern
   `* Performance Highlights.md` (filename sort descending).
2. If none found: write *"No Performance Highlights note found yet —
   run /weekly-generate in personal-weekly-update to start producing
   them."* and skip the paragraph. Don't block.
3. If the most recent note is older than 14 days, prefix the paragraph
   with the age: *"Most recent Performance Highlights note is from
   YYYY-MM-DD (N days ago)."*
4. Read the note. Distil 2-4 sentences, mapped to current goals from
   `~/Git/a8c/lead-calls-prep/references/goals.md` (the canonical goals
   source). Do not paraphrase the entire note — keep it tight.

#### Lodestar's note
Optional: any meta-observation worth Paul seeing later (e.g. "third
week in a row Goal 1 has been quiet; might be worth raising with
Sarah").
```

### Step 12 — Close the session

End with a one-line affirmation, calibrated to actual progress. The GTD
note frames this well: "feel ready and clear for the upcoming week."

> Reviewed 6, processed 4 Inbox items, captured 3 new tasks, Top 3
> locked in. Good session — see you next Monday.

If Paul cut the review short:

> We did Stage 1 + half of Stage 2. The rest is in the queue for next
> time. No guilt.

---

## Finding the real Tasks section

Some project notes embed the bare `_Templates/project_template.md` inside
a fenced code block as documentation, so the file contains *two* `## Tasks`
headings — one inside the fence (template doc) and one as the actual
section. Naive "first match" picks the wrong one.

Rule: scan line by line, toggle an `in_fence` flag on every line starting
with three backticks, and only count `## Tasks` headings where `in_fence`
is false. Take the **last** such heading as the real one.

If no real `## Tasks` heading is found, treat the project as having no
tasks — don't error.

## Rules

- **Confirm before mutating any vault frontmatter.** Always batch.
- **Inbox sweeps are one-at-a-time.** Never bulk-process. The whole point
  is conscious decisions.
- **Don't push vault tasks to Todoist from this skill.** That's
  `lodestar-sync-todoist`'s job. Mention as a follow-up if appropriate.
- **The Top 3 are Paul's call.** Don't pre-pick them or pressure a
  particular set. Ask, capture, confirm.
- **Don't lecture about cadence.** If the last review was 4 weeks ago,
  just do today's review. The session summary may note it; the chat tone
  stays neutral.
- **Match Paul's energy.** If he's terse, be terse. If he wants to think
  out loud, give him room.
- **Don't auto-trigger other skills mid-review.** Capture-opportunity,
  sync-todoist, and similar mutating flows happen *after* the review,
  not during. Mention them as follow-ups when relevant.
