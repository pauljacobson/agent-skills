---
name: support-review
description: >
  Review and work through WordPress.com support interactions. Use when the user provides
  Zendesk ticket URLs, Slack threads, or Linear issues for review — or says things like
  "let's review this ticket", "review these tickets", "support review", or any variation
  of wanting to analyze a support interaction. Covers the full lifecycle: structured review,
  iterative troubleshooting, and formatted outputs (Zendesk notes, P2 posts, Linear issues,
  working notes). Also trigger when the user explicitly invokes /support-review.
---

# Support Interaction Review

## Purpose

Review WordPress.com support interactions and work through them end-to-end: understand the
issue, investigate and troubleshoot, find solutions, and produce formatted outputs for
internal tools (Zendesk, P2, Linear).

This skill is a lightweight orchestrator. It owns the review framework and output templates
but delegates technical diagnosis to the `troubleshooting` skill and uses ContextA8C for
accessing internal resources.

## When to Use

Trigger this skill when:
- The user provides Zendesk ticket URLs for review
- The user says "review this ticket", "let's look at this issue", or similar
- The user invokes `/support-review`
- The user provides a mix of Zendesk, Slack, and Linear URLs for a support case

## Delegation

| Concern | Handled by |
|---|---|
| Review framework & output templates | This skill |
| Technical diagnosis & troubleshooting | `troubleshooting` skill (invoke automatically) |
| Slack, Zendesk, Linear data fetching | ContextA8C (use proactively) |
| Security/malware investigation | `security-file-analysis` skill (invoke when needed) |

**Do not ask before delegating.** When there is an identifiable issue to diagnose, invoke
the `troubleshooting` skill automatically. When URLs reference internal resources, fetch
them via ContextA8C immediately.

## Phase 1: Context Gathering

On invocation, immediately:

1. **Parse the user's message** for URLs:
   - Zendesk: `a8c.zendesk.com/agent/tickets/...`
   - Slack: `a8c.slack.com/archives/...`
   - Linear: `linear.app/a8c/issue/...`

2. **Fetch all referenced resources via ContextA8C**:
   - Load the appropriate ContextA8C provider
   - Retrieve ticket content, Slack thread messages, Linear issue details
   - Do not ask permission — fetch proactively

3. **Check the working directory** (`~/Downloads/Testing/Support interaction review/`)
   for a site-domain subdirectory containing supplementary files (logs, CSVs, exports).
   Read any relevant files found.

4. **Set up the working subdirectory** if one doesn't already exist:
   - **Preferred name**: the customer's site domain (e.g., `motorradundtouren.ch/`)
   - **Fallback**: `YYYYMMDD_HHMM Username` if no site domain is identifiable
     (e.g., `20260409_0924 Dominique/`)
   - If the subdirectory already exists, use it as-is

5. **Note any freeform context** the user included in their message.

## Phase 2: Initial Review

Produce a structured review using the 4-point framework. This is always the first output.

### 1. What is the issue the customer reported?
- What happened, when it occurred, specific details useful for troubleshooting
- Site domain, relevant URLs, customer details

### 2. What did the support assistant try?
- Actions taken chronologically
- What worked, what didn't, what was inconclusive

### 3. What progress did the support team and/or customer make?
- Current state of the issue
- Any partial resolutions or workarounds in place

### 4. What are the unresolved issues?
- What still needs to be addressed
- Blockers, open questions, dependencies

## Phase 3: Investigation & Troubleshooting

This phase is conversational and iterative. The user drives it.

- If there is an identifiable technical issue, **automatically invoke the `troubleshooting`
  skill** — do not ask first
- Use ContextA8C to pull additional context as needed during the conversation
- Review supplementary files the user adds to the working subdirectory
- Help identify root causes, correlate evidence, and develop hypotheses

## Phase 4: Resolution

Once the issue is understood:
- Help identify and evaluate potential solutions
- Assist with implementing fixes (scripts, configuration changes, WP-CLI commands, etc.)
- Validate fixes against evidence gathered during investigation

## Phase 5: On-Demand Outputs

The user requests these as needed. Use the formats below.

### Zendesk Internal Notes

- Concise progress summary for the ticket
- Markdown format
- Single-level bullet list (no nesting)
- Factual, not interpretive
- Drafted in English (Zendesk handles translation)

### P2 Escalation Posts

```markdown
#### Summary of the Issue
[What's happening, site context, timeline]

#### Current Behavior
[What the customer/support team observes now]

#### Expected Behavior
[What should happen instead]

#### Steps to Reproduce
[Numbered steps to see the issue]

#### UI Errors / Browser Console / CLI PHP-Errors
[Any error output, or note if none]

#### Additional Context
[Background, what's been tried, relevant findings, proposed approach]

---

#### Site and User Details:
**Slack at-help Ping:** [URL]
**Site Address:** [domain]
**Blog RC:** [URL]
**Follow-up Ticket:** [Zendesk URL]
**Store Admin:** [URL or N/A]
```

### Linear Issues

- Title, priority, labels
- Summary section with clear problem statement
- Root cause evidence (data, logs, specific findings)
- Impact assessment (who's affected, how widespread)
- Suggested improvements or fix approaches
- Related issues section with Linear/Zendesk links

### Working Notes

Structure:
- **Header**: ticket number, site, customer, date, who's working on it
- **Background**: how the issue came to be
- **What we tried**: chronological record with rationale and results for each step
- **What went wrong** (if applicable)
- **Resolution**
- **Key technical findings** (numbered, detailed)
- **Remaining issues**
- **Lessons learned**

## Formatting Conventions

These apply to all outputs produced by this skill:

- No horizontal rules (`---`) as section separators (except before Site and User Details
  in P2 posts, as shown in the template above)
- Wrap site domain names in backticks (e.g., `example.com`); full `https://` URLs don't
  need backticks
- Hyperlink relevant posts, pages, and resources rather than leaving them as plain text
- When listing affected posts/pages, use a bullet list with hyperlinks
- Draft customer-facing messages in English (Zendesk handles translation)
- Heading styles start at level 3 (`###`) for saved report files

## File Organization

- **Base directory**: `~/Downloads/Testing/Support interaction review/`
- **Subdirectories**: named by site domain (preferred) or `YYYYMMDD_HHMM Username` (fallback)
- **Filenames**: `Ticket [NUMBER] - [Type].md` (e.g., `Ticket 11037797 - Working Notes.md`)
- **Version control**: commit on creation, before edits, after edits (per project CLAUDE.md)

## Communication Style

- Respond informally but respectfully
- Be helpful, constructive, and curious
- Verify factual accuracy before asserting — distinguish clearly between speculation
  and evidence-based findings
- Give clear examples when explaining technical concepts
- Express opinions when thoughtful and evidence-based — don't sugar-coat
- Use emoji occasionally but not excessively

## Permissions

- **Read-only actions** (ContextA8C fetches, file reads, directory scans): proceed without asking
- **Write actions** (creating files, committing): follow normal permission prompts
- **Skill invocation** (troubleshooting, security-file-analysis): invoke automatically
  when the situation calls for it

## Out of Scope

This skill does NOT:
- Replace the `troubleshooting` skill's diagnostic methodology
- Replace the `security-file-analysis` skill's malware analysis capabilities
- Handle customer-facing replies directly (the user composes those)
- Manage ticket routing or assignment
