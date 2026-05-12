---
name: writing-handoff
description: Use when an auto-compaction system reminder appears, when the user explicitly invokes /handoff or asks to "write a handoff" / "save state", or when the user signals they're near the context limit specifically ("running low on context", "almost out of context", "context is getting full"). Do NOT fire on general session-ending phrases like "wrap up" or "let's stop for today" unless they're tied to context exhaustion.
---

# Writing a Handoff

## Overview

Save the current session's state to a Markdown file so a future session (or another person) can pick up cleanly. This skill is intentionally conservative about triggering: it fires on explicit invocation or real context-pressure signals, never on generic "let's wrap up" phrasing.

## When to use

Fire this skill when ANY of:
- The user invokes `/handoff` (with or without a label argument).
- The user explicitly asks: "write a handoff", "save state", "save session", "make a handoff".
- The user signals context exhaustion specifically: "running low on context", "almost out of context", "context is getting full", "we're near the limit".
- A system reminder about auto-compaction appears in the conversation.

Do NOT fire on:
- Generic session-ending phrases: "wrap up", "let's stop", "call it a day", "I'm tired", "let's call it here" — unless the user explicitly ties them to context limits.
- Mid-task pauses or breaks.

If unsure, ASK the user before writing — a one-line confirmation is cheaper than a wrong handoff.

## What to do

1. **Resolve the output directory:**
   - If env var `HANDOFF_DIR` is set, use it.
   - Otherwise, use `./.handoffs/` relative to the current working directory.
   - Create the directory if it does not exist: `mkdir -p <dir>`.

2. **Collect context (run these in parallel where possible):**
   - Project name: `basename "$(pwd)"`
   - Current ISO timestamp: `date -u +"%Y-%m-%dT%H:%M:%SZ"`
   - Filename timestamp: `date +"%Y-%m-%d-%H%M"`
   - If `.git` exists in cwd or any parent:
     - Branch: `git rev-parse --abbrev-ref HEAD`
     - Uncommitted: `git status --short`
     - Recent commits: `git log -5 --oneline`
   - Otherwise mark git fields as `n/a`.

3. **Build the filename:**
   - Base: `<filename-timestamp>.md` (e.g., `2026-05-12-1430.md`)
   - If a label was provided (via `/handoff <label>` or the user's phrasing), slugify it (lowercase, replace non-alphanumerics with `-`, collapse multiple dashes, trim leading/trailing dashes) and append: `2026-05-12-1430-before-refactor.md`.

4. **Write the file using the template below**, populated from the actual conversation so far. Be specific — the next reader has no access to this conversation.

5. **Confirm to the user** using exactly this two-line format:
   ```
   ✓ Wrote <absolute-path-to-handoff-file>
     Captured: <N files changed>, <N commits>, <N next steps>
   ```
   The summary line should count concrete items from the handoff body. If a category has zero items, write `0 files changed`, `0 commits`, etc. — don't omit.

## Template

```markdown
# Handoff — <project> — <ISO timestamp>

**Working directory:** <cwd>
**Git branch:** <branch or "n/a">
**Session label:** <label or "—">

## Current task
The "you are here" pointer — what we were doing and where we stopped.

## Completed this session
- Decisions made
- Files changed (with paths)
- Commits (hashes + subjects)

## In progress
- Uncommitted changes (file paths + brief description)
- Half-written code, partial implementations

## Next steps
Immediate TODOs, in priority order.

## Blockers / open questions
Things waiting on user input or external decisions.

## Key context
- Relevant file paths and what's in them
- Important function/symbol names
- Conventions or patterns learned this session
- Surprises or gotchas worth remembering
```

## Quality bar for the content

- Every section must have real content from this conversation, not placeholder text. If a section truly has nothing (e.g., no blockers), write "None." — not the example bullets from the template.
- File paths must be absolute or unambiguous relative paths.
- "Current task" must be a single concrete sentence, not a vague topic.
- "Next steps" must be actionable, in priority order.

## Red flags — stop and re-check

| Thought | Reality |
|---------|---------|
| "User said 'let's wrap up' so I'll write a handoff" | Generic session-ending ≠ context exhaustion. Ask first. |
| "I'll fill in the template with example values" | Examples are not state. Use real conversation content. |
| "I don't have git info, I'll skip the git section" | Mark it `n/a`, don't silently drop it. |
| "I'll write to /tmp" | No. Use `HANDOFF_DIR` or `./.handoffs/`. |
| "I'll skip the filename label" | If the user gave one, include it. Slugify, don't drop. |
