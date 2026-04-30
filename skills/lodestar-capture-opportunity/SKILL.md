---
name: lodestar-capture-opportunity
description: >
  Conversational capture of a new work opportunity, idea, or commitment.
  Either appends a one-liner to opportunities/inbox.md (lightweight) or
  triggers Paul's existing QuickAdd "New Project" choice in Obsidian
  (committed) — which fills _Templates/project_template.md with the title
  Paul provides. Use when Paul says "track this", "new opportunity", "add
  this as a project", "lodestar capture", or runs
  /lodestar-capture-opportunity. Also triggers when Paul mentions a Sarah
  suggestion, a P2 idea, or something he wants to revisit.
allowed-tools: Bash, Read, Edit
---

# lodestar-capture-opportunity

Two-stage funnel: inbox first if uncommitted, vault project note (via
QuickAdd) when ready.

## Prerequisites

1. Read `~/Git/Projects/lodestar/config.md` for paths.
2. Verify Obsidian is installed (the URI scheme requires the desktop app).
3. The vault has the **QuickAdd** plugin configured with a Choice named
   **"New Project"** (template: `_Templates/project_template.md`,
   filename = `{{value}}`). This is Paul's existing setup; lodestar
   delegates to it rather than rolling its own template fill.

## Procedure

### Step 1 — Capture the gist

Ask Paul one short question if he hasn't already given context:

> What's the opportunity in one sentence?

Then quickly determine commitment level. Choose based on Paul's wording:

- "I should look into..." / "maybe..." / "Sarah mentioned..." → **inbox**
- "I'm going to..." / "let's track..." / "this is a project" → **vault project (QuickAdd)**
- Ambiguous → ask: "Inbox for now (one line, decide later) or full project
  note?"

### Step 2a — Inbox path (lightweight)

Append to `~/Git/Projects/lodestar/opportunities/inbox.md`:

```
- YYYY-MM-DD — short title — context (origin, who suggested, when worth revisiting)
```

Keep oldest at top, newest at bottom. Never edit existing lines silently —
only append. Confirm the line with Paul before writing.

### Step 2b — Vault project note path (via QuickAdd)

Use Paul's existing **QuickAdd "New Project"** flow rather than fabricating
the file content. This keeps a single source of truth: any change to the
QuickAdd config or the template is automatically picked up by lodestar.

**Procedure:**

1. **Decide the project title** (used as the filename — `{{value}}` in the
   QuickAdd config). Confirm with Paul before triggering.

2. **Trigger QuickAdd via the Obsidian URI scheme:**

   ```bash
   open "obsidian://quickadd?vault=Notes%20Hub&choice=New%20Project&value=$(printf '%s' "$TITLE" | jq -sRr @uri)"
   ```

   - `vault=Notes%20Hub` — the vault name (the folder name, not the full
     path). URL-encoded.
   - `choice=New%20Project` — the QuickAdd Choice name. URL-encoded.
   - `value=...` — the project title, URL-encoded. Use `jq -sRr @uri` (or
     equivalent) to encode safely.

   This brings Obsidian to focus, runs QuickAdd's "New Project" choice,
   creates the file from `_Templates/project_template.md`, fills `{{value}}`
   into the filename, runs Templater (so `Created:` gets today's date), and
   opens the new note.

3. **Confirm the note was created.** Briefly: "Obsidian should be opening
   the new project note. Did it appear?" If yes, proceed to Step 2c. If no,
   fall back to manual create (see "Fallback" below).

### Step 2c — Post-creation enrichment (optional)

The QuickAdd "New Project" choice only fills the filename. Lodestar can
optionally enrich the new file by editing it after creation. Offer Paul:

> Want me to fill in any of these now: due date, status, priority, the
> "What is this project about?" paragraph, or a first task?

If yes, edit the file at `<vault>/<title>.md` to add the requested fields.
Use `obsidian:obsidian-cli` or direct file write — but **always show the
proposed edit and confirm before writing**.

**Important — the stale-due bug:** the project template ships with
`due: 2024-09-26`. After QuickAdd creates the note, lodestar should always
ask:

> The template has a stale due date (`2024-09-26`). Want to clear it, set
> a real target, or leave it?

Default: clear it. Don't propagate the bug.

### Fallback — direct file write (only if QuickAdd fails)

If the URI invocation doesn't work (Obsidian not running, URI scheme
disabled, vault name mismatch), fall back to:

1. Read `_Templates/project_template.md` directly.
2. Compose the note content with `{{value}}` replaced by the title and
   `<% tp.date.now("YYYY-MM-DD") %>` replaced by today's date (Templater
   syntax that won't run if we bypass QuickAdd).
3. Confirm the full content with Paul.
4. Write to `<vault>/<title>.md` via `obsidian-cli` or direct write.
5. Note in the response: "QuickAdd didn't trigger — used direct write
   instead. You may want to check Obsidian is running before next time."

### Step 3 — Promotion (inbox → vault)

If Paul invokes this skill referencing an existing inbox line ("promote
that Snyk thing from inbox"), the skill should:

1. Find the matching line in `inbox.md`.
2. Trigger the QuickAdd flow as in Step 2b, with the title from the inbox
   line.
3. **After** vault note creation is confirmed, remove the line from
   `inbox.md` (separate confirmation: "Remove from inbox now? [y/n]").

### Step 4 — Offer follow-ups

After capture, optionally:

- "Want to add the first next-action to Todoist now?" → defer to
  `lodestar-sync-todoist`
- "Want me to flag this for the next 1:1 with Sarah?" → recommend running
  `lead-call-prep` next time

These are offers, not automatic — Paul says yes or this is the end.

## Rules

- **Delegate to QuickAdd by default.** Lodestar does not duplicate
  QuickAdd's template-fill logic. Single source of truth.
- **Confirm the title before triggering QuickAdd.** The title becomes the
  filename and is hard to change after.
- **Inbox writes only require confirmation of the line content**, not the
  whole file (since it's pure append).
- **Always offer to clear the stale `due:` field** after QuickAdd creates a
  new note. Default: clear.
- **Don't fill in `status: inprogress` defensively.** A new opportunity
  isn't necessarily started. Blank is the safer default — only set if Paul
  says he's starting now.
- **Don't promote inbox items unless Paul asks.** Inbox is allowed to sit.

## Reference — the URI scheme

QuickAdd registers the `obsidian://quickadd` URI handler. Parameters:

- `vault` — vault name (folder name, URL-encoded)
- `choice` — Choice name as defined in QuickAdd settings (URL-encoded)
- `value` — substituted into `{{value}}` placeholders in the choice
  (URL-encoded)

If Paul renames the Choice or the vault, update this skill — or, better,
add the Choice name and vault name as fields in `~/Git/Projects/lodestar/config.md`
so this skill reads them at runtime.
