---
name: lodestar-sync-todoist
description: >
  One-way push of unsynced project next-actions from Obsidian project notes
  into Todoist. Idempotent via a sync-state file. Always confirms before
  creating Todoist tasks. Use when Paul says "sync todoist", "push my tasks",
  "lodestar todoist", or runs /lodestar-sync-todoist. Pairs with the
  `todoist` skill which wraps the `td` CLI.
---

# lodestar-sync-todoist

Pushes the **first unchecked task** from each active project note in the
vault into Todoist, skipping anything already synced. One-way only in v1
(Obsidian → Todoist). Two-way is a future idea, not in scope.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Verify `td` CLI is installed and authenticated — defer to the existing
   `todoist` skill for the auth check.
3. Use `obsidian:obsidian-cli` for vault reads.

## Procedure

### Step 1 — Gather candidates

For each project matching the **"Working on"** view in `Bases/Projects.base`
(status active):

1. Read the project note.
2. Find the **first** unchecked `- [ ]` line in the `## Tasks` section.
3. Compute a stable fingerprint: `sha256(project_filename + ":" + task_text)`,
   first 12 hex chars.
4. Check `~/Git/Projects/lodestar/todoist-sync/synced.jsonl` — skip if the
   fingerprint already has an entry.

### Step 2 — Present batch for confirmation

Show Paul the full set of candidates:

```
Found 4 new next-actions across 4 projects:

1. [Block plugins workflow] Email Brandon about v2 review timeline
2. [Lodestar] Write the goals-nudge skill SKILL.md
3. [Hebrew Calendar plugin] Test the localisation fix on a staging blog
4. [Same-site migration tracking] Ping Marie re: PM ownership

Push all 4 to Todoist (default project: Inbox)?
[y/n/select]
```

Three response paths:

- `y` / `yes` / `proceed` → push all
- `n` / `cancel` → no writes; end the turn
- `select` → walk through one at a time, y/n per item

### Step 3 — Push

For each approved task, call the `todoist` skill / `td` CLI to create the
task. Recommended Todoist task body format:

```
<task text>  [<project name>]
```

Or `[<project name>] <task text>` — pick one and stick to it. The bracketed
project tag makes it easy for Paul to spot vault-sourced tasks in Todoist.

If Paul has a preferred Todoist project for vault-sourced tasks, ask once
on first run and remember (record in `config.md` or a small state file).

### Step 4 — Record sync state

For each successfully created Todoist task, append to
`~/Git/Projects/lodestar/todoist-sync/synced.jsonl`:

```json
{"date":"2026-04-30","fingerprint":"a1b2c3d4e5f6","project_note":"Block plugins workflow.md","task_text":"Email Brandon about v2 review timeline","todoist_id":"123456789"}
```

### Step 5 — Summarise

```
Pushed 4 tasks to Todoist. Skipped 0 (already synced).
```

Or if a task fails to create:

```
Pushed 3, failed 1: "Test the localisation fix" — error: <reason>. Retry next run.
```

Don't append to `synced.jsonl` for failures.

## Rules

- **One task per project per run.** Lodestar pushes the first unchecked task
  only. The rest stay in Obsidian — they're not "next" yet. This intentional
  throttle keeps Todoist from ballooning with backlog items.
- **Always confirm before any Todoist write.** Even on follow-up runs.
- **Idempotent.** Re-running with no new tasks is a clean no-op.
- **Don't tick off tasks in Obsidian.** Even when the corresponding Todoist
  task is completed — that's two-way sync, out of scope for v1.
- **Don't delete from Obsidian.** Ever. Tasks stay in the project note.
- **Vault is read-only from this skill.** Only state file
  (`synced.jsonl`) and Todoist see writes.

## Future ideas (do NOT implement now)

- Two-way: detect Todoist completions and tick the matching `- [ ]` in the
  vault.
- Project-to-project mapping: each Obsidian project gets its own Todoist
  project, not flat to Inbox.
- Due-date propagation from `due:` frontmatter to Todoist task due date.

These all add complexity and failure modes. Stay simple until the simple
version has been used for a couple of weeks.
