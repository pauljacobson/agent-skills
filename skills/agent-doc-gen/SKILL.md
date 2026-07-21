---
name: agent-doc-gen
description: Use whenever you are about to produce a written deliverable — a report, a draft reply to a user or customer, an internal note, a P2 post, an issue write-up, or any similar document. Triggers on phrasings like "write a report", "draft a reply", "write up an internal note", "document this", "create a note", "draft a response", or any request that results in a saved document. Files each document as its own Markdown file in a per-site subdirectory of the working directory, and enforces that the document's content follows whatever more specific skill or instruction governs that document type.
---

# Agent Document Generation & Filing

This skill governs **where** generated documents are saved and **how they are named** — nothing more. It deliberately does **not** invent content formats. The body of every document must follow whatever more specific skill or agent instruction applies to that document type (see [Content defers to the governing skill](#content-defers-to-the-governing-skill)).

Think of it as the filing layer that sits on top of every other document-producing skill.

## When this applies

Fire this skill whenever the task results in a **saved written deliverable**, including:

- Reports
- Draft replies to users or customers
- Internal notes (e.g. Zendesk internal notes — see [[zendesk-internal-note]])
- Working notes, handoff notes
- P2 posts, issue write-ups, and similar documents

Do **not** fire it when no written deliverable is being produced (e.g. a conversational answer, a code change with no accompanying document).

## What to do

### 1. Identify the site

Determine which site the document relates to, from whatever context is available — a URL, a Zendesk ticket, a Linear issue, or the conversation itself.

- The subdirectory is named after the site's bare domain, **exactly as written**: a site at `blkbrd.film` → a folder named `blkbrd.film`.
- Do not add `https://`, `www.`, trailing slashes, or otherwise normalise the name beyond taking the domain as given.

**If you cannot determine the site, stop and ask** before writing anything:

> Which site do these relate to? (e.g. `blkbrd.film`)

Do not guess silently and do not fall back to a default folder — ask.

### 2. Write one Markdown file per document

- Create the subdirectory `./<site>/` (relative to the current working directory) if it does not already exist, then write the document there.
- **One file per document.** If a task produces several deliverables (e.g. a report *and* a draft reply), write each as its own file in the same site folder.
- **Reports get two files.** For any document of type *report*, always produce **both** a Markdown file **and** an HTML slide-deck companion with the same base name (see [Reports also get an HTML slide deck](#reports-also-get-an-html-slide-deck)). This applies wherever the report is saved — site folder or working-directory root.

### 3. Name each file

Use the standard report filename convention for **every** document type:

```
YYYYMMDD Title of the document.md
```

- `YYYYMMDD` — ISO date at the time of generation.
- Title — concise and subject-based, describing what the document is.

Examples inside `blkbrd.film/`:

```
blkbrd.film/
  20260708 Migration status report.md
  20260708 Reply re broken checkout.md
  20260708 Zendesk note - DNS handoff.md
```

### 4. Content defers to the governing skill

This skill owns **location and filename only**. The **body** of each document must follow whatever more specific skill or agent instruction governs that document type. When more than one could apply, **the most specific one wins.**

| Document type | Content governed by |
|---|---|
| Zendesk internal note | [[zendesk-internal-note]] skill — use its exact template (📌 previous note / 📐 Progress / ✅ Next steps) |
| Report | The user's CLAUDE.md report format (level-3 top heading, proceeding down from there) |
| Issue / bug write-up | `write-effective-issues` skill |
| Support outputs (P2 post, working note, Linear issue) | `support-review` skill templates |
| Anything else | The most relevant skill or agent instruction for that document type |

If a governing skill exists for the document type, invoke or follow it for the content, then apply this skill's filing rules to save the result. If none applies, write clear, well-structured Markdown.

### 5. Report where you saved each file

After writing, tell the user the path(s) of every file created, e.g.:

> Saved to `blkbrd.film/20260708 Migration status report.md`

## Relationship to the CLAUDE.md report rule

The user's global CLAUDE.md says reports go in the **root** of the working directory. This skill **overrides that rule only when the document relates to a site** — in that case the report (and every other document) goes in the site subdirectory instead.

- Document relates to a site → `./<site>/YYYYMMDD Title.md`
- No associated site → the existing CLAUDE.md rule stands (report in the working-directory root).

## Out of scope

- Inventing document content formats of its own — content always defers to the governing skill.
- Reorganising, renaming, or moving pre-existing files.
- Firing when no written deliverable is being produced.
