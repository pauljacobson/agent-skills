---
name: lodestar-weekly-review
description: >
  Walks Paul through a guided GTD weekly review across the projects matching
  the "To review" view in Projects.base. Updates each project's `reviewed:`
  frontmatter (with confirmation) and saves a session summary to
  ~/Git/Projects/lodestar/reviews/YYYY-MM-DD.md. Use when Paul says "weekly
  review", "GTD review", "let's review my projects", or runs
  /lodestar-weekly-review. Typically Monday morning.
---

# lodestar-weekly-review

Facilitates Paul's weekly review. Paul drives — lodestar surfaces, prompts,
and records.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Read `~/Git/Projects/lodestar/references/weekly-review-procedure.md` for
   the GTD prompts adapted to this workflow.
3. Use `obsidian-cli` (skill: `obsidian:obsidian-cli`) for vault access.

## Procedure

### Step 1 — Surface the list

Query the vault for projects matching the **"To review"** filter (already
defined in `Bases/Projects.base` lines 25-48). Sort by `reviewed:` ascending
(oldest first). Group by `status:` per the base view definition.

Open with:

> We've got N projects to review, sorted by oldest review first. Most-overdue
> is "X" (last reviewed YYYY-MM-DD). Let's start there.

If N > 8, offer: "That's a lot. Want to do the most-overdue 4 today and pick
up the rest later?"

### Step 2 — Per-project pass

For each project, in order:

1. **Read the project note** — show the title, status, `reviewed:` date, and
   the current top unchecked task in the project's `## Tasks` section.
   **Locate the section using the rule below**, not the naive "first match,"
   so projects that embed the template inside a code fence (e.g. the
   lodestar project note itself) don't trip you up.
2. **Apply 2-3 of the GTD prompts** from `weekly-review-procedure.md`,
   choosing based on the project's state:
   - If untouched 4+ weeks → ask about shelved/someday_maybe
   - If next action is vague → ask for a concrete physical step
   - If waiting on someone → confirm pending status + capture who/what
3. **Record decisions in memory** (don't write files yet — batch at the end).
4. **Move on.**

Keep each per-project pass under 90 seconds of conversation. If Paul wants to
go deep on one, that's fine — but the skill should not solicit depth by
default.

### Step 3 — Sweep journal for unrecorded commitments

Read the last 7 days of `journal`-tagged notes. Look for verb phrases that
suggest commitments not reflected in any project note: "I should...", "need
to...", "going to...", "Sarah suggested...". Surface 0-3 candidates:

> While reviewing, I noticed these in your journal that don't appear in any
> project: [list]. Want to add any?

### Step 3.5 — GitHub inbox sweep

Run `gh-inbox` in **non-mutating mode** so the user's own `/gh-inbox`
triage state isn't consumed:

```bash
~/.claude/skills/gh-inbox/scripts/fetch.sh --since 7d
```

Parse the JSON. Group by category (Assigned / Body mentions / Comment
mentions) and surface as candidates for vault project tasks or new
opportunities:

> Last 7 days from your GitHub inbox: 4 assignments, 6 mentions. Any of
> these look like they should become a task in an existing project, or a
> new opportunity to capture?
>
> 1. owner/repo#123 — Title — assigned to you
> 2. owner/repo#456 — Title — comment mention
> ...

For each item the user wants to act on:
- "task in existing project" → ask which project; queue an edit to add the
  GitHub URL as a task line in that project's `## Tasks` section (defer to
  the batched writes in Step 4).
- "new opportunity" → suggest running `/lodestar-capture-opportunity` after
  the review (don't trigger QuickAdd mid-review).
- "ignore" → no-op; the item stays in the GitHub inbox for the user's
  separate `/gh-inbox` flow.

If gh-inbox returns 0 items in the window, output a single line:
"GitHub inbox is quiet for the last 7 days." Move on.

If the script's `partial_failures` is non-empty, surface a brief warning
("Note: N gh-inbox queries failed — sweep may be incomplete.") but
continue.

### Step 3.7 — Todoist Inbox sweep (process unprocessed items)

The Todoist Inbox is the GTD "stuff pile" — where new tasks land before
they're categorised. Paul typically won't have seen these unless he's been
deliberate about processing the Inbox. Surface each item and walk through
it consciously. **One at a time, not bulk** — bulk-processing defeats the
inbox-zero purpose.

Run `td inbox --json` (via the `todoist` skill). Sort by added-date,
oldest first. Show the count and start the walk:

> You have N tasks in your Todoist Inbox. Let's process them one at a
> time, starting with the oldest.
>
> 1. "Email Brandon about v2 review" — added 8 days ago
>    What's the next action: assign to a project, defer, delete, or
>    leave?

For each item, the four action paths:

- **Assign to project** — ask which Todoist project (or which vault
  project, if it should become a `- [ ]` line in a project note instead).
  If a Todoist project, defer the move to the batched writes step. If a
  vault project, defer to the batched writes (or suggest running
  `/lodestar-capture-opportunity` after the review for a brand-new
  project).
- **Defer** — ask when (e.g. "next week", "2026-05-15"). Reschedule via
  the todoist skill. Confirm: "Defer '...' to YYYY-MM-DD? [y/n]" before
  the write.
- **Delete** — explicit confirmation per item: "Delete '...' from Todoist?
  [y/n]". Never bulk-delete. Use the todoist skill's delete operation
  only after the user types yes.
- **Leave** — skip; the task stays in Inbox for the next sweep. No-op.

If there are 10+ Inbox items, offer: "That's a lot. Want to process the
oldest 5 today and pick up the rest next week?" Don't push to clear
everything in one session — that's the failure mode that makes people
abandon the review.

If Inbox is empty:

> Todoist Inbox is empty. Nice work.

**Inbox should not contain vault-synced tasks.** By policy,
`lodestar-sync-todoist` writes to a dedicated project (e.g. "Lodestar"),
never Inbox — see `config.md`. If you find vault-synced tasks here (e.g.
their text matches the `<task> [<project name>]` format used by sync), do
NOT auto-move them. Surface them and ask Paul: "These look like
vault-synced tasks in Inbox — is your sync destination misconfigured?"

### Step 4 — Batched writes (with confirmation)

After the pass, summarise pending writes as a single confirmation block:

```
About to update:
- 6 projects: reviewed → 2026-04-30
- "Block plugins workflow": status → pending (waiting on Brandon)
- "Training Simulator": new task added

Proceed?
```

Only write on explicit "yes" / "proceed" / equivalent. If Paul says no or
modifies, redo the confirmation. **Never partial-write.**

### Step 5 — Session summary

Write to `~/Git/Projects/lodestar/reviews/YYYY-MM-DD.md`:

```markdown
### Weekly review — YYYY-MM-DD

**Reviewed**: N projects

#### Status changes
- ProjectName: oldStatus → newStatus (reason)

#### New tasks captured
- ProjectName: "physical action phrase"

#### Flagged for follow-up
- (anything Paul wanted to revisit)

#### Goal signal this week
One paragraph from `goal-mapping.md` keyword sweep — which goals had strong
signal in journal entries, which were quiet.

#### Lodestar's note
Optional: any meta-observation worth Paul seeing later (e.g. "third week in
a row Goal 1 has been quiet; might be worth raising with Sarah").
```

### Step 6 — Close the session

End with a one-line affirmation, calibrated to actual progress:

> Reviewed 6, updated 4, captured 3 new tasks. Good session — see you next
> Monday.

If Paul cut the review short:

> We did 3 of 6. The remaining 3 are still in the queue for next time. No
> guilt.

## Finding the real Tasks section

Some project notes embed the bare `_Templates/project_template.md` inside a
fenced code block as documentation, so the file contains *two* `## Tasks`
headings — one inside the fence (template doc) and one as the actual
section. Naive "first match" picks the wrong one.

Rule: scan line by line, toggle an `in_fence` flag on every line starting
with three backticks, and only count `## Tasks` headings where `in_fence`
is false. Take the **last** such heading as the real one.

If no real `## Tasks` heading is found, treat the project as having no
tasks — don't error.

## Rules

- **Confirm before mutating any vault frontmatter.** Always batch.
- **Don't push to Todoist from here.** That's `lodestar-sync-todoist`'s job.
  Mention it as a follow-up if appropriate.
- **Don't lecture about cadence.** If the last review was 4 weeks ago, just
  do today's review. The session summary may note it, but the chat tone
  stays neutral.
- **Match Paul's energy.** If he's terse, be terse. If he wants to think out
  loud, give him room.
