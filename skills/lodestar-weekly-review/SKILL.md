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

A ~30-second pass before Stage 1 that probes every external source, reports green/yellow/red per source, and lets Paul opt skip / fix / abort per non-green source. Captures `preflight_skip[<source>]` and `preflight_calendar_scope` flags that downstream stages honour silently.

**Read `references/preflight.md` before executing this stage** — it has the full probe table (7 sources), the surface format, the skip/fix/abort prompt shape, the Calendar scope handling, and the rules for honouring skip flags downstream. Don't proceed without reading it; the details matter.

After preflight, proceed to Stage 1.

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

### Step 1b — Stale `#next_action` tag cleanup

The `#next_action` tag flags candidate next actions in vault project
notes (see `lodestar-sync-todoist` for the convention). The tag stays
on the task until the work is done. This step sweeps for tags that
have gone stale via either of the two completion signals.

Skip this step silently if `~/Git/Projects/lodestar/todoist-sync/synced.jsonl`
does not exist *and* a vault grep for `#next_action` returns zero hits
— there's nothing to clean.

#### Part A — Vault-side: ticked tasks that still carry the tag

Grep the project-notes directory (per `config.md`) for lines matching
`^- \[x\] .*#next_action`. For each match, surface it:

> "Email Brandon about v2 review timeline" was ticked in the vault
> (in `Block plugins workflow.md`) but still carries `#next_action`.
> Remove the tag?

Capture the decision; defer the edit to Step 10 batched writes. The
edit is purely textual — strip the inline `#next_action` token from
the task line, leave the rest of the line (including the `[x]` and
any other content) untouched.

#### Part B — Todoist-side: completed promotions

Note on `td` capability (verified 2026-05-04): `td task view id:X
--json` returns identical JSON for active and completed tasks (no
`completed` / `isCompleted` field) and HTTP 400 for unknown IDs. There
is no per-ID completion-status query. The working path is to **sweep
the completion window** via `td completed --since <date> --json`,
build a set of completed IDs, and cross-reference.

Procedure:

1. Read `~/Git/Projects/lodestar/todoist-sync/synced.jsonl`.
2. For each entry, look up the vault task line. If it no longer
   contains `#next_action` (already removed by an earlier sweep, or by
   Paul directly), drop the entry from the working set — nothing to do.
3. If the working set is empty, skip the rest of Part B.
4. Compute the completion-window `since` date:
   - **Default**: the date of the most recent prior session summary in
     `~/Git/Projects/lodestar/reviews/` (filename pattern
     `YYYY-MM-DD.md`).
   - **Fallback** (no prior reviews, or prior review is more than 30
     days old): 30 days ago.
5. Run `td completed --since <since> --json --all` (the `--all` flag
   paginates fully so no `nextCursor` handling is needed). Collect IDs
   into a set `completed_ids`.
6. For each working-set entry, partition by signal:
   - **`todoist_id ∈ completed_ids`** → completed within window. Surface:
     > Todoist task "Email Brandon about v2 review timeline" (Fission
     > project) was marked complete. Remove `#next_action` from the
     > vault task in `Block plugins workflow.md`?
   - **`td task view id:X --json` exits 0, but ID not in
     `completed_ids`** → still active. No surface, no action.
   - **`td task view id:X --json` exits non-zero (HTTP 400)** →
     deleted, or completed before the window. Ask:
     > Can't find Todoist task <id> ("Email Brandon..."). It was
     > either completed before <since> or deleted. If completed,
     > remove the `#next_action` tag. If you might re-promote it,
     > leave the tag.

Rate-limit consideration: step 6's per-entry `td task view` calls only
run for entries *not* in `completed_ids`. In practice the working set
is small (only entries whose vault task still carries the tag), so
this is bounded.

#### Output

If both Part A and Part B return zero items: write one line "No stale
`#next_action` tags found." and proceed to Step 2.

Otherwise, summarise the captured decisions (count of tags pending
removal, broken down by signal) and proceed to Step 2. The actual
removals land in Step 10 batched writes.

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

### Step 2b — Linear tracked-issues sweep

Lodestar tracks specific Linear items (read-only) as an 8th signal source — both
whole **projects** (`kind: project`) and single tracking **issues**
(`kind: issue`). See `config.md` § "Linear (tracked issues)". This sweep
surfaces each tracked entry's open gaps that are NOT already represented in its
vault note, so the work keeps moving without lodestar maintaining a parallel
list.

If `preflight_skip["linear"]` is True, silently skip to Step 3.

The deterministic parts (parse the table, filter open gaps, flag new-since,
dedup against the vault note) live in `scripts/linear_tracked.py`; the live
Linear fetch is done here via the MCP. **Read-only** — never resolve, edit, or
create anything in Linear.

Procedure:

1. Read the tracked list:
   ```bash
   python3 ~/Git/Projects/lodestar/scripts/linear_tracked.py config
   ```
   Each row gives `ref`, `kind`, `vault_note`, etc. If the list is empty, write
   one line ("No tracked Linear items configured.") and move to Step 3.

2. Compute the `since` date — the most recent prior session-summary date in
   `~/Git/Projects/lodestar/reviews/` (filename `YYYY-MM-DD.md`); 30-day
   fallback if none (or the latest is >30 days old).

3. For each tracked entry, fetch its live state via the **standalone Linear
   MCP** (read-only), falling back to the ContextA8C `linear` provider if the
   MCP is unavailable; write the result JSON to a temp file:
   - `kind: project` → `list_issues` with `project: <ref>` → `/tmp/lt_<n>.json`
     (shape: `{"issues": [...]}` with `statusType`, `createdAt`, `url`).
   - `kind: issue` → `list_comments` with `issueId: <ref>` → `/tmp/lt_<n>.json`
     (shape: `{"comments": [...]}` with `parentId`, `resolvedAt`, `createdAt`).
   - Graceful degradation: if the fallback can't supply `resolvedAt`, treat all
     comments as open and lean on the `createdAt` watermark.

4. Run the dedup / new-flag filter (handles both kinds):
   ```bash
   python3 ~/Git/Projects/lodestar/scripts/linear_tracked.py surface \
     --kind <kind> --json-file /tmp/lt_<n>.json \
     --vault-note "<vault root>/<vault_note>.md" --since <since>
   ```

5. Surface the results per tracked entry in **two tiers** (see `config.md`
   § Linear → Read semantics):

   - **New since last review** (`is_new: true` in the `surface` output) → offer
     as **candidates**, same three options as the GitHub sweep:

     > Linear tracked — <title> (<N> open gaps, <K> new since last review):
     > 1. ★ HAP-2923 — Generate Linear templates… (Backlog) [new]
     > 2. "<gap summary>" [new]
     > Any of these become a task in the vault project note, a new opportunity, or ignore?

     - **Task in existing project** → which (default: the entry's vault note from
       the config row); queue the `- [ ]` edit for Step 10 batched writes.
       Adding it also makes the next sweep dedup it.
     - **New opportunity** → suggest `/lodestar-capture-opportunity` after the
       review.
     - **Ignore** → no-op; it stays in Linear (lodestar never mutates Linear).

   - **All open gaps** → show only as a **count + link**, do **not** re-pitch
     them individually (this is what keeps already-captured gaps from being
     re-offered every week):

     > <title> has 9 open gaps total — <issue/project url>

6. If a comment/issue looks already addressed (e.g. Paul says a fix shipped),
   you may **remind** him to resolve it in Linear himself — but never resolve it
   for him.

If `surface` returns `[]` for an entry: "No new gaps on <title> since last
review." If both Linear paths are unreachable and the source wasn't pre-skipped,
note it in one line ("Linear unavailable — skipping the tracked-issues sweep")
and continue. Never block the review on Linear.

**Read-only**: never resolve, edit, or create Linear comments or issues.

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

Before the per-project pass, sweep every project-tagged note's frontmatter and surface drift across three categories: non-canonical `status:` values (read canonical list from `config.md` at runtime), malformed YAML, and zombie notes (empty status + empty reviewed + stale/empty due > 365 days). Offer batched normalisation for non-canonical statuses; per-file repair with diff for malformed YAML; per-zombie disposition with no auto-mutation.

**Read `references/frontmatter-audit.md` before executing this step** — it has the full probe rules, the compact surface format, the batched-normalisation confirmation flow, the per-file YAML repair procedure (using `obsidian property:set` per the `feedback_obsidian-cli-for-frontmatter` memory), and the zombie handling.

If the audit is clean (no drift in any category), output one line and proceed to Step 6. Any normalisations / repairs / zombie decisions get logged under "Frontmatter audit" in the Step 11 session summary.

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

#### Triage UI generation (triage modes only)

When `stage2_mode` is `triage_only` or `triage_then_deep_dive`, generate
the interactive triage UI **before** the per-project pass begins:

```bash
cd ~/Git/Projects/lodestar
python3 scripts/generate-triage-ui.py [--exclude "Title 1,Title 2"]
```

The `--exclude` flag accepts comma-separated project titles already
handled earlier in the session (e.g. Lodestar from a Stage 0 pre-pass).
The script calls `scripts/to-review.py --json` internally to get the
live queue and writes a fresh `triage-ui/index.html` — so the UI always
reflects the actual current queue, not stale data.

Open the UI via the Claude Preview MCP (`preview_start`) pointing at
`~/Git/Projects/lodestar/triage-ui/index.html`. Paul uses the per-card
buttons to triage each project (Keep / Shelve / Someday / Pending /
Complete / Deep-dive). When finished, he clicks **"Finish & export → JSON"**
to copy the JSON decisions block, then pastes it into the conversation.

Process the pasted JSON as the triage decisions for Step 10's batched
writes — no further per-project prompting needed for simple triage
actions. Projects Paul marked "Deep-dive" become the active subset for
the card-flow pass (triage-then-deep-dive) or are flagged as needing
a follow-up session (triage-only).

If `generate-triage-ui.py` fails (script missing, `to-review.py` errors,
Preview MCP unavailable), fall back to the conversational triage-form
flow documented below.

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

Top tasks (3 of 7 unchecked, ★ = #next_action):
  • ★ Email Brandon about v2 review timeline
  • Test the localisation fix on staging
  • Draft P2 post about the workflow

Notes (last entry, truncated):
  > Brandon happy with v1 output; v2 should narrow scope to migration cases…
```

Mark each unchecked task carrying the inline `#next_action` tag with
a leading `★`. The legend in the section header reminds Paul what the
star means.

Then ask **one consolidated GTD question** — pick 1-3 prompts based on
project state, but ask them as a single block, not sequentially. The
shape of the next-action prompt depends on how many tasks carry the
`#next_action` tag:

- **1 tagged**: cite it as the candidate.
  > Quick check on Block plugins workflow:
  > - Still active, or shelf/someday?
  > - The `#next_action`-tagged task ("Email Brandon...") — is that
  >   still the very next action?
  > - Anything to capture or change?
  >
  > Or just say "next" to mark reviewed and move on.

- **0 tagged** (any active project should have one): flag the gap.
  > Quick check on Block plugins workflow:
  > - Still active, or shelf/someday?
  > - **No task tagged `#next_action`.** What's the very next physical
  >   action? (Tag it `#next_action` in Obsidian to mark it; a Top 3
  >   slot or a `lodestar-sync-todoist` push is optional.)
  > - Anything to capture or change?

- **2+ tagged**: sanity check.
  > Quick check on Block plugins workflow:
  > - Still active, or shelf/someday?
  > - **N tasks tagged `#next_action`** — which is *the* next one?
  >   (Multiple is fine if the project has parallel workstreams; worth
  >   a quick sanity check.)
  > - Anything to capture or change?

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
batched-writes step (Step 10). Every `#next_action`-tagged vault task
pushed to Todoist **must** carry the `Next_Actions✅` label — see Step 10
for the three-step write pattern (add → move → label).

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

If the active projects collectively carry `#next_action`-tagged tasks,
offer them as a starting menu before Paul commits:

> The `#next_action` tags across active projects right now: <list,
> grouped by project>. Want to draw any of these into the Top 3, or
> propose something else?

Tagged tasks are natural Top 3 candidates because Paul has already
flagged them as next actions; the only question left is whether they
deserve attention *this week*. Don't push for selection from this list
— it's a menu, not a recommendation.

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
  Every task promoted from a vault `#next_action` tag **must** carry the
  `Next_Actions✅` label (Todoist label id: `2152094658`). Defer all
  Todoist writes to Step 10 batched writes; the three-step pattern
  documented there handles the label correctly.

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
- 2 stale #next_action tags removed (1 ticked in vault, 1 completed in Todoist)
- Todoist: 2 Inbox items deferred, 1 deleted, 3 assigned to projects
- Top 3 added to Todoist (all with Next_Actions✅ label)

Proceed?
```

Only write on explicit "yes" / "proceed" / equivalent. If Paul says no or
modifies, redo the confirmation. **Never partial-write.**

#### Todoist write pattern for promoted `#next_action` tasks

Every vault `#next_action` task pushed to Todoist in this step **must**
carry the `Next_Actions✅` label (id: `2152094658`). The `td task add`
command does not support `--json`, and multi-word project names do not
parse via NLP — use the three-step pattern:

```bash
# 1. Add to Inbox (NLP parses date/priority; project is handled in step 2)
td task add "Task text here"           # note the returned task ID

# 2. Move to the correct project
td task move "id:<id>" --project "Family financial"

# 3. Apply the Next_Actions✅ label (replaces ALL labels — include any
#    context labels too, comma-separated)
td task update "id:<id>" --labels "Next_Actions✅"
# or, if a context label also applies:
td task update "id:<id>" --labels "Next_Actions✅,Computer💻"
```

Omitting step 3 is the error that occurred on 2026-05-09. Never skip it.

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
**`#next_action` cleanup** (omit line if zero): N tags removed (M
ticked in vault, K completed in Todoist)
**Linear tracked issues** (omit line if skipped/none): N open gaps across M
issues (K new this week, surfaced as candidates)
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

#### Format learnings
Meta-observations about how the **review itself** went, separate from
per-project decisions. Three sub-sections; omit any that produced
nothing:

##### What worked
- (e.g. "Triage-then-deep-dive cleared 30 in 25 min where deep-dive
  alone would have taken 90 min")

##### What needs to change
- (e.g. "Stage 0 preflight should also probe the Performance Highlights
  notes — got caught off guard when none existed")

##### What surprised me
- (e.g. "Didn't expect 21 of 89 projects to use non-canonical statuses;
  that's a 24% drift rate")
```

#### Prompting for learnings

After drafting the rest of the summary, but before writing the file,
explicitly prompt Paul:

> Format learnings — anything worth flagging about how this review
> itself went?
> - What worked?
> - What needs to change about the skill?
> - What surprised you?
>
> Skip any that didn't produce something specific.

If Paul has examples from prior reviews to draw on, surface 1-2 from
recent `reviews/*.md` files as a kickstart — but only if the prior
review actually had a Format learnings section. Don't fabricate
examples; the section is descriptive, not aspirational.

If Paul writes "What needs to change" entries that look like concrete
**skill-spec changes** (mention specific Steps, name a behaviour that
should be different, suggest a new probe / step / handler), capture
them in working memory as **candidate-issues**. They get acted on in
Step 12.

### Step 12 — Close the session

#### Skill-spec issue offer

If Step 11's Format learnings captured **candidate-issues** (entries
in "What needs to change" that look like concrete skill-spec changes),
offer to create GitHub issues for them before the close affirmation:

> Two skill-spec items came up:
>   1. "Stage 0 should also probe Performance Highlights notes"
>   2. "Triage form needs a 'why' field for `pending` status changes"
>
> Want me to open lodestar issues for these?

If yes:
- For each candidate, draft a title and body using the standard
  weekly-review issue shape (What / Why / Spec / Related — referencing
  this review's session summary).
- Show all drafts together, ask once for confirmation, then create
  via `gh issue create --repo pauljacobson/lodestar` with appropriate
  labels (`enhancement`, `weekly-review`).
- Confirm with issue numbers/URLs after creation.

If no, leave the candidate-issues only in the session summary — Paul
can act on them later.

This is opt-in only — never auto-create issues. Format learnings is
descriptive; turning learnings into issues is a separate decision.

#### Close affirmation

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
