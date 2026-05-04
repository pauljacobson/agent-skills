---
name: lodestar-sync-todoist
description: >
  Interactive promotion of a single Obsidian project task to Todoist as a
  next action. Paul picks the task; Paul picks the destination Todoist
  project. Not a bulk sync — Obsidian holds the full project backlog,
  Todoist holds only the very next actions Paul has consciously elevated.
  Surfaces `#next_action`-tagged vault tasks as the strong candidates;
  applies the `Next_Actions✅` Todoist label on push. Use when Paul says
  "promote this to todoist", "add this as a next action", "push to
  todoist", or runs /lodestar-sync-todoist. Pairs with the `todoist`
  skill which wraps the `td` CLI.
---

# lodestar-sync-todoist

**Obsidian project notes are not mirrored to Todoist.** The vault holds
the full project backlog (every task that comes to mind, from broad ideas
to small chores). Todoist holds only the **very next actions** Paul has
consciously elevated. This skill facilitates that promotion — one task at
a time, with the Todoist destination project chosen per task.

If your mental model is "sync the lists," that model is wrong. The right
model is "elevate one task from backlog to current attention."

## When to use

- During a weekly review, when the per-project pass identifies a clear
  next action that should sit in front of Paul this week.
- Mid-week, when Paul realises a vault task should become the next action
  for that project — typically because the prior next action is done.
- After a 1:1 or capture moment that surfaced a concrete next step worth
  acting on this week.

**Don't use this skill for bulk operations.** If Paul wants to push three
items, run the skill three times. The friction is intentional —
promoting a task is a decision, not a sweep.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Verify `td` CLI is installed and authenticated — defer to the existing
   `todoist` skill for the auth check.
3. Use `obsidian:obsidian-cli` for vault reads.

## Procedure

### Step 1 — Identify the source

Paul can specify the task in three ways:

1. **Already named in conversation**: "promote 'Email Brandon about v2
   review' from the Block plugins workflow" → source is identified, skip
   to Step 2.
2. **Project named, task not specified**: "promote a task from Block
   plugins workflow" → read the project note, locate the real `## Tasks`
   section (rule below), and surface candidates per the **tag-driven
   surfacing** rule below. Ask which one.
3. **Nothing specified**: "I want to promote a task to Todoist" → list
   active projects (the **"Working on"** view in `Bases/Projects.base`)
   and ask which one. Then drill into its tasks as in path 2.

#### Finding the real Tasks section

Scan the file line by line, tracking whether you're inside a fenced code
block (toggle a `in_fence` flag on every line that starts with three
backticks). A `## Tasks` heading only counts when `in_fence` is false.
Take the **last** such heading in the file as the real one.

If no real `## Tasks` heading is found, or the section is empty / contains
only an empty placeholder, tell Paul there's nothing to promote and stop.

#### Tag-driven candidate surfacing

Within the unchecked tasks of the located `## Tasks` section, count how
many lines carry the inline `#next_action` tag. Three branches:

- **Exactly 1 tagged**: that's the strong candidate. Show it and ask
  "This task is tagged `#next_action`. Use it, or pick something else?"
  — accept silence/yes as confirmation; if Paul wants a different task,
  list the full set.
- **0 tagged**: no pre-flagged next action. Ask: "No task in this
  project carries `#next_action`. Pick one from the list below, or add
  the tag in Obsidian first so it's marked as the next action going
  forward?" — then list the full set.
- **2+ tagged**: list only the tagged tasks first, framed: "N tasks
  carry `#next_action` — which one?" If Paul wants a non-tagged task,
  he can ask for the full list.

The tag is a candidate marker, not an exclusivity claim — Paul may
genuinely have parallel next actions, especially in projects with
multiple workstreams. When unsure (0 or 2+), always ask.

### Step 2 — Confirm the task text

Strip `#next_action` (and any other inline `#tags`) from the task text
before showing it. Tags are vault metadata, not part of the Todoist
task. Then show what will become the Todoist task:

> Source: `Block plugins workflow.md`
> Task: "Email Brandon about v2 review timeline"
>
> Push this as-is, or edit the wording before sending?

Allow Paul to refine the wording. Todoist task text often benefits from
small tweaks (e.g. add a verb, drop project-internal jargon). The
Obsidian task text stays unchanged regardless — including the
`#next_action` tag, which only comes off when the work is done (see
"Tag lifecycle" below).

### Step 3 — Pick the destination Todoist project

Ask which Todoist project. Default suggestions, in order:

- The Todoist project Paul most recently used for promotions (track this
  in `todoist-sync/synced.jsonl` — most recent `todoist_project` field).
- A project name Paul says directly.
- "Pick from a list" → fetch via `td projects --json` and let Paul choose.

**Never default to Inbox.** Inbox is reserved for unprocessed user
inputs that the weekly review will triage. If Paul names Inbox: push
back gently — "Inbox is reserved for unprocessed inputs in your weekly
review. Suggest something else, like the project for this work area?"

### Step 4 — Confirm and push

Final confirmation:

```
About to add to Todoist:
  Project: <todoist project>
  Label:   Next_Actions✅
  Task:    "Email Brandon about v2 review timeline"
  Source:  Block plugins workflow.md (vault)

Proceed? [y/n]
```

On `y`, create the task via the `todoist` skill. Apply the
`Next_Actions✅` Todoist label automatically — this is the Todoist-side
counterpart of the vault `#next_action` tag, marking the task as a
next action across all of Paul's Todoist projects (it's a label, not
a project, so it composes with whatever project Paul picked in
Step 3). Optionally include a trailing reference like `(from: Block
plugins workflow)` in the task body so Paul can trace it back later,
but only if he wants — ask once and remember the preference.

### Step 5 — Record state

Append to `~/Git/Projects/lodestar/todoist-sync/synced.jsonl`:

```json
{"date":"2026-04-30","fingerprint":"a1b2c3d4e5f6","project_note":"Block plugins workflow.md","vault_task_text":"Email Brandon about v2 review timeline","todoist_task_text":"Email Brandon about v2 review timeline","todoist_project":"Fission","todoist_id":"123456789"}
```

The state file is for **observability**, not deduplication. Paul might
legitimately re-promote a task (e.g. after the prior Todoist task was
deleted, or to bump it back to attention) — the skill should not block
that. Just log the fact.

### Step 6 — Summarise

> Added "Email Brandon about v2 review timeline" to your Fission project
> in Todoist.

If creation fails, surface the error and don't write to `synced.jsonl`.

## Tag lifecycle

The vault `#next_action` tag stays on the task line through promotion.
It only comes off when the underlying work is done. The weekly review
sweeps for two completion signals (see `lodestar-weekly-review` Step
1b):

1. The Todoist task this tag is bound to (via `todoist-sync/synced.jsonl`'s
   `todoist_id`) is marked complete in Todoist.
2. The Obsidian task line is marked done (`- [x]`).

Either signal is a cue to remove the tag, with confirmation — this
skill never auto-edits the vault task. The `todoist_id` recorded in
Step 5 is what makes the Todoist-side check possible; don't drop it.

## Rules

- **One task per invocation.** This is the whole point. If Paul asks for
  three, run three times.
- **Always confirm before any Todoist write.**
- **Never auto-pick the destination project.** Always ask, even if the
  most recent promotion suggests an obvious choice — confirm.
- **Never write to Inbox.** Push back if Paul names it.
- **Don't tick or modify the Obsidian task.** The vault task stays as-is;
  Paul ticks it manually when the underlying work is done. Two-way sync
  (Todoist completion → vault tick) is out of scope.
- **Don't bulk.** No "find all unsynced and push them." That mental model
  is wrong for this system.

## Anti-patterns

- "Sync everything" — there is no "everything to sync." Obsidian's
  backlog is intentional; pushing it all to Todoist drowns the next-action
  list in noise.
- "Mirror Obsidian projects to Todoist projects" — Paul's Todoist
  projects are organised by area of work, not by Obsidian project. The
  mappings are 1:many and human-driven.
- "Push the first unchecked task from each project" — this was the old
  v1 design. It's wrong: the first unchecked task isn't necessarily the
  next action; it might be the largest or most distant. Paul decides
  what's next.
