---
name: lodestar-capture-opportunity
description: >
  Conversational capture of a new work opportunity, idea, or commitment.
  Either appends a one-liner to opportunities/inbox.md (lightweight) or
  creates a new project note in the Obsidian vault using project_template.md
  (committed). Use when Paul says "track this", "new opportunity", "add this
  as a project", "lodestar capture", or runs /lodestar-capture-opportunity.
  Also triggers when Paul mentions a Sarah suggestion, a P2 idea, or
  something he wants to revisit.
---

# lodestar-capture-opportunity

Two-stage funnel: inbox first if uncommitted, vault project note when ready.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Use `obsidian:obsidian-cli` for vault writes.
3. Read `_Templates/project_template.md` so the generated note matches
   structure (frontmatter fields, headings).

## Procedure

### Step 1 — Capture the gist

Ask Paul one short question if he hasn't already given context:

> What's the opportunity in one sentence?

Then quickly determine commitment level. Choose based on Paul's wording:

- "I should look into..." / "maybe..." / "Sarah mentioned..." → **inbox**
- "I'm going to..." / "let's track..." / "this is a project" → **vault project**
- Ambiguous → ask: "Inbox for now (one line, decide later) or full project
  note?"

### Step 2a — Inbox path (lightweight)

Append to `~/Git/Projects/lodestar/opportunities/inbox.md`:

```
- YYYY-MM-DD — short title — context (origin, who suggested, when worth revisiting)
```

Keep oldest at top, newest at bottom. Never edit existing lines silently —
only append.

Confirm with Paul before writing.

### Step 2b — Vault project note path (committed)

Generate a filename from the title — kebab-case or use Paul's preferred
convention (check existing project notes in the vault for the pattern).
Confirm filename with Paul before creating.

Use `_Templates/project_template.md` as the structural blueprint. Fill in:

```markdown
---
aliases:
tags:
  - projects
cssclasses:
  - img-grid
due:
started:
completed:
reviewed:
status:
priority:
---

Main: 
Related: 
Created: [[YYYY-MM-DD]]

# <Project Title>

## What is this project about?

<one paragraph from Paul's description>

## Notes

<source/origin: e.g. "Suggested by Sarah in 2026-04-29 1:1">

## Tasks

- [ ] <first physical next action, if Paul knows it>

## Related notes

![[related_notes.base]]
```

Important overrides from the bare template:

- **Do NOT propagate the template's stale `due: 2024-09-26`.** Leave `due:`
  blank unless Paul specifies a target date.
- **`Created:`** uses today's date as a wikilink (matches template behaviour
  via Templater — replicate manually).
- **`status:`** leave blank or set to `inprogress` only if Paul explicitly
  says he's starting now.
- **`reviewed:`** always blank for a new project.

Confirm the full file content with Paul before writing.

### Step 3 — Promotion (inbox → vault)

If Paul invokes this skill referencing an existing inbox line ("promote that
Snyk thing from inbox"), the skill should:

1. Find the matching line in `inbox.md`.
2. Generate the vault project note as in Step 2b.
3. **After** vault note is confirmed and written, remove the line from
   `inbox.md` (separate confirmation: "Remove from inbox now? [y/n]").

### Step 4 — Offer follow-ups

After capture, optionally:

- "Want to add the first next-action to Todoist now?" → defer to
  `lodestar-sync-todoist`
- "Want me to flag this for the next 1:1 with Sarah?" → recommend running
  `lead-call-prep` next time

These are offers, not automatic — Paul says yes or this is the end.

## Rules

- **Always confirm before writing to the vault.** Show the full file
  content; only write on explicit yes.
- **Inbox writes only require confirmation of the line content**, not the
  whole file (since it's pure append).
- **Use the project template structure literally.** Don't add sections, don't
  rename headings — Paul's other tooling depends on the existing shape.
- **Don't fill in `status: inprogress` defensively.** A new opportunity
  isn't necessarily started. Blank is the safer default.
- **Don't promote inbox items unless Paul asks.** Inbox is allowed to sit.
