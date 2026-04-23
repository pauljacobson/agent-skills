---
name: gh-inbox
description: Use when the user wants to check GitHub mentions or assignments — e.g. "what's in my GitHub inbox", "any new mentions on GitHub", "show me PRs assigned to me", "what GitHub items have I missed since [date]". Surfaces both body and comment @mentions and assigned issues/PRs. Default invocation is stateful and reports only items new or changed since the last run.
---

# gh-inbox

Surface `@pauljacobson` mentions and assignments from GitHub. Default invocation is stateful: items already reported and unchanged since the last run are suppressed.

## How to invoke

Always run the script and parse its JSON output. Do not call `gh` directly.

```bash
~/.claude/skills/gh-inbox/scripts/fetch.sh [options]
```

### Options

- *(no args)* — Default. Reports items new or changed since last run. Updates `state.json`.
- `--since <7d|2026-04-15|yesterday|...>` — Ad-hoc time window. Ignores state.
- `--all` — Same window as default, but skips dedup. Does not modify state.
- `--reset --yes` — Clear state. Ask the user to confirm before passing `--yes`.
- `--repo owner/repo` — Narrow scope.
- `--org owner` — Narrow scope.
- `--mentions-only` / `--assignments-only` — Skip the other half.

### Conflict rules

- `--since` and `--all` cannot be combined.
- `--reset` is exclusive with all other flags.
- `--mentions-only` and `--assignments-only` are mutually exclusive.

## Output shape (stdout JSON)

```json
{
  "generated_at": "2026-04-23T10:30:00Z",
  "window": { "since": "...", "mode": "default-cursor", "deduped": true },
  "counts": {
    "assignments": 3, "body_mentions": 5, "comment_mentions": 7,
    "total_unique": 14, "suppressed_unchanged": 22
  },
  "items": [ { "url": "...", "repo": "...", "number": 1, "type": "issue|pr",
               "title": "...", "state": "open|closed",
               "categories": ["assignment", "body_mention", "comment_mention"],
               "sources": ["search", "notification"],
               "updated_at": "...", "latest_comment_url": "..."|null } ],
  "partial_failures": []
}
```

`mode` is one of `default-cursor`, `default-firstrun-14d`, `since-flag`, `all-flag`.

## How to render in chat

Group items by category and emit grouped markdown. If the user just asked "any new mentions", default to this layout:

```
## Assigned to you (N open)
- [owner/repo#123](url) — Title — updated <relative time>

## Body mentions (N since <date>)
- [owner/repo#456](url) — Title

## Comment mentions (N since <date>)
- [owner/repo#789](url) — Title
```

Always include a one-liner footer: `Tracking from <window.since>. Suppressed N unchanged items.` when `window.deduped` is true.

If `partial_failures` is non-empty, surface a warning *before* the list: "Note: <N> queries failed — results may be incomplete." Do not silently drop them.

## Hand-off to other skills

- **Todoist:** when the user says "add these to my tasks" or similar, pass each item's `url`, `repo`, `number`, `title` to the `todoist` skill.
- **Browser:** when the user asks to open something, use the `url` field directly.

## Failure handling

The script exits 0 even with `partial_failures`. Exit 1 means setup error (missing tool, auth, corrupt state). Exit 2 means usage error (bad flags).

If the script exits 1 because of corrupt state, ask the user to confirm before running `--reset --yes`.

## State

`~/.claude/skills/gh-inbox/state.json` (gitignored). Stores `last_checked` cursor and a `reported` map keyed by URL. Pruned automatically: closed items >30 days old, anything >90 days, soft cap 1000 entries.

## Reference

`references/gh-query-cookbook.md` documents the exact `gh` commands the skill runs and the qualifier syntax for ad-hoc queries.
