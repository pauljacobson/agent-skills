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

### Step 6 — Per-project pass ("To review" projects)

Query the vault for projects matching the **"To review"** filter (defined
in `Bases/Projects.base` lines 25-48). Sort by `reviewed:` ascending
(oldest first). Group by `status:` per the base view definition.

> N projects need review, sorted by oldest review first. Most-overdue is
> "X" (last reviewed YYYY-MM-DD). Let's start there.

If N > 8: "That's a lot. Want to do the most-overdue 4 today and pick up
the rest later?"

For each project, in order:

1. **Read the project note** — show title, status, `reviewed:` date, and
   the unchecked tasks in the project's `## Tasks` section (cap at 5
   shown; mention if there are more). The vault is the project backlog,
   so multiple open tasks is normal — that's not a problem.
   **Locate the section using the rule below**, not the naive "first
   match," so projects that embed the template inside a code fence
   (e.g. the lodestar project note) don't trip you up.
2. **Apply 2-3 of the GTD prompts** from `weekly-review-procedure.md`,
   choosing based on project state:
   - If untouched 4+ weeks → ask about shelved/someday_maybe
   - If next action is vague → ask for a concrete physical step
   - If waiting on someone → confirm pending status + capture who/what
3. **Identify the next action.** From the unchecked tasks shown, ask
   Paul which (if any) is the **very next action** — the one that
   should sit in front of him this week. Often this is the top task; it
   doesn't have to be. Paul might also name a next action that isn't
   in the vault yet — capture it as a new task.
4. **Offer to promote the next action to Todoist.** "Want to push
   '<task text>' to Todoist now?" If yes, ask which Todoist project
   (don't default to Inbox). Defer the actual write to the batched
   writes step (Step 10). If no, that's fine — it stays in the vault
   backlog only.
5. **Record decisions in memory** (don't write files yet — batch in
   Step 10).
6. **Move on.**

Keep each project under 90 seconds of conversation by default. If Paul
wants to go deep on one, give him room — but don't solicit depth.

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

> Given everything we've just looked at: what are your **Top 3 next
> actions** for this week? Not goals, not areas — concrete physical
> next steps that, if done, would make this week feel productive.

Capture them. Offer:

> Want me to push these to the top of your Todoist for this week?

If yes, add to Todoist via the `todoist` skill (with confirmation per
task). Add a `@top3` label or due-today flag if Paul has a convention.
Otherwise, drop them in his preferred project (NOT Inbox).

The Top 3 also get recorded in the session summary (Step 11).

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

### Step 11 — Session summary

Write to `~/Git/Projects/lodestar/reviews/YYYY-MM-DD.md`:

```markdown
### Weekly review — YYYY-MM-DD

**Reviewed**: N projects (Stage 2)
**Inbox processing**: Todoist N → 0/M, GitHub N items, journal N
commitments captured (Stage 1)

#### Top 3 for the week ahead
1. ...
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
One paragraph from `goal-mapping.md` keyword sweep — which goals had
strong signal in journal entries, which were quiet.

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
