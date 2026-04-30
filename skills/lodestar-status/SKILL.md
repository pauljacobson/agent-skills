---
name: lodestar-status
description: >
  On-demand snapshot of Paul's lodestar view: active goals, projects in flight,
  recent nudges, projects due for review. Read-only — never mutates anything.
  Use when Paul says "lodestar status", "what's on my plate", "lodestar
  snapshot", or runs /lodestar-status. Pairs well at the start of a working
  session before deciding what to focus on.
---

# lodestar-status

A read-only snapshot. Use this as the validator that lodestar's paths and tools
are wired correctly — if this skill works end-to-end, the rest of the
constellation has the inputs it needs.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for canonical paths.
2. Verify the four key paths exist:
   - Vault root
   - `Bases/Projects.base`
   - `lead-calls-prep/references/goals.md`
   - The lodestar repo itself
3. If any are missing, surface that as the first item in the output and stop.

## Procedure

Generate a markdown summary with these sections, in order. Keep the whole
output under ~25 lines unless Paul asks for detail.

### 1. Active goals

Read `lead-calls-prep/references/goals.md`. Extract the goal headings (Goal 1,
2, 3) and their `Status:` values. One line per goal:

```
- Goal 1 — Cybersecurity Certificate — Paused (Mar 2026)
- Goal 2 — AI-Assisted Workflows — Active
- Goal 3 — Scripting / CLI — Active
```

### 2. Projects in flight

Use `obsidian-cli` (or read project notes directly) to list projects matching
the **"Working on"** view in `Bases/Projects.base` (status not in
complete/completed/shelved/someday_maybe). Output:

```
Working on (N): name1, name2, name3, ...
```

If N > 5, list the 5 with the oldest `started:` dates and append "+ N more".

### 3. Due for review

Same source, but the **"To review"** view filter (status active AND
(reviewed.isEmpty() OR reviewed < now() - "1 week")). Output the count and the
3 oldest:

```
Due for review (N): oldest3...
```

If N == 0, write "Caught up on reviews."

### 4. Recent nudges

Read the last 7 days of `~/Git/Projects/lodestar/nudges/log.jsonl`. Show each
on one line: `YYYY-MM-DD — goal — first 80 chars of message`. If empty, write
"No nudges fired recently."

### 5. Quiet goals

For each goal, count keyword matches in the last 7 days of journal entries
(use `goal-mapping.md` for keyword sets). If any goal has 0 matches, list it
as "Quiet for 7 days: Goal X". Don't generate a nudge here — that's
`lodestar-goals-nudge`'s job. Just observe.

### 6. GitHub inbox freshness

Read `~/.claude/skills/gh-inbox/state.json` (if present) and report the age
of `last_checked` in days. One line:

```
GitHub inbox: last checked N days ago.
```

If the file doesn't exist or is unparseable, write `GitHub inbox: not yet
initialised.` and move on. **Never call `gh-inbox/scripts/fetch.sh` from
this skill** — that script's default mode mutates state, and we'd consume
items the user hasn't seen yet through their own `/gh-inbox` flow.

### 7. Suggested next move

One sentence. Not a nudge — a routing suggestion. Examples:
- "Weekly review is overdue (last one was 11 days ago); consider /lodestar-weekly-review."
- "Three new tasks in vault project notes haven't been pushed; consider /lodestar-sync-todoist."
- "Nothing pressing — good time for queue work."

## Rules

- **Never write anything.** No nudge log entries. No frontmatter changes. No
  Todoist mutations. Status is observational only.
- **Don't editorialise.** Report counts and dates. Save commentary for the
  "Suggested next move" line.
- **Stay under ~25 lines.** This is a glance, not a report.
- **If Paul asks for detail**, follow up with a deeper read of the section
  they're asking about — but the default output is terse.
