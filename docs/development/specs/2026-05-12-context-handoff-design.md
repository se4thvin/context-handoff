# Context Handoff — Design

**Date:** 2026-05-12
**Author:** se4thvin
**Status:** Approved (pending user re-review of this written spec)

## Summary

A Claude Code plugin that lets a user (or Claude itself, in narrow conditions) save a structured `handoff.md` before context runs out, and pick up cleanly from one in a new session. Distributed as `se4thvin/context-handoff` on GitHub.

## Goals

- Capture session state to a Markdown file in a known location, on demand.
- Let a new session find that file, summarize it, and continue the work.
- Trigger conservatively: explicit user invocation OR a real context-pressure signal — never on generic "let's wrap up" phrasing.
- Be installable as a standard Claude Code plugin from GitHub.

## Non-Goals

- Automatic compaction-time hooks (out of scope for v0.1; can be added later).
- Cross-machine sync of handoffs.
- Any automatic detection of context usage beyond the system reminders Claude already receives.

## Architecture

Two skills + two slash commands, packaged as one plugin.

```
context-handoff/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── writing-handoff/
│   │   └── SKILL.md
│   └── resuming-from-handoff/
│       └── SKILL.md
├── commands/
│   ├── handoff.md
│   └── resume.md
├── README.md
└── LICENSE
```

The skills hold the intelligence and templates. The slash commands are thin entry points that invoke the skills with optional arguments.

## Components

### Skill: `writing-handoff`

**Description (CSO):**
> Use when an auto-compaction system reminder appears, when the user explicitly invokes /handoff or asks to "write a handoff" / "save state", or when the user signals they're near the context limit specifically ("running low on context", "almost out of context", "context is getting full"). Do NOT fire on general session-ending phrases like "wrap up" or "let's stop for today" unless they're tied to context exhaustion.

**Behavior:**
1. Determine project name from `basename $(pwd)`.
2. If the cwd is a git repo, capture: current branch, `git status --short`, last 5 commit subjects.
3. Fill in the handoff template (below) using current conversation state.
4. Resolve output directory:
   - If `HANDOFF_DIR` env var is set → use it.
   - Otherwise → `./.handoffs/` relative to cwd.
5. Create the directory if missing.
6. Write file: `YYYY-MM-DD-HHMM[-<label>].md` where `<label>` is the optional `/handoff <label>` argument.
7. Print the absolute file path and a one-line summary.

**Handoff template:**

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

### Skill: `resuming-from-handoff`

**Description (CSO):**
> Use when user invokes /resume, says "pick up where we left off", "continue from last session", "what was I working on", or starts a session with a reference to prior work.

**Behavior:**
1. Resolve search directories:
   - Always check `./.handoffs/` (project-local).
   - Also check `$HANDOFF_DIR` if set and different.
2. List handoffs sorted by mtime, newest first.
3. Branch on count:
   - **Zero** → tell user, suggest `/handoff` next time. Stop.
   - **One** → read it, summarize state in 3-5 lines, ask whether to continue from "Next steps".
   - **Multiple** → present a numbered picker:
     ```
     1. 2026-05-12 14:30 — "Refactoring auth middleware"
     2. 2026-05-12 09:15 — "before-refactor"
     3. 2026-05-11 17:42 — "WIP: parser bug"
     ```
     The preview text is the first non-empty line under `## Current task`. User picks → load → summarize → confirm before acting.
4. `/resume <n>` skips the picker and jumps straight to handoff index `<n>` (1-based, newest-first).

### Slash command: `/handoff [label]`

Thin wrapper. Invokes the `writing-handoff` skill. If `[label]` is provided, it's appended to the filename (slugified: lowercase, hyphens for spaces).

### Slash command: `/resume [n]`

Thin wrapper. Invokes the `resuming-from-handoff` skill. If `[n]` is provided, skips the picker and loads handoff #n directly.

## Configuration

One env var:

| Variable | Default | Effect |
|----------|---------|--------|
| `HANDOFF_DIR` | unset → uses `./.handoffs/` | Override output (and additional search) directory. Useful for centralizing handoffs to e.g. `~/.claude/handoffs/`. |

## Data flow

```
User says "/handoff before-refactor"
  → /handoff slash command fires
  → invokes writing-handoff skill with label="before-refactor"
  → skill collects: cwd, git state, conversation summary
  → writes ./.handoffs/2026-05-12-1430-before-refactor.md
  → prints path

[new session, possibly weeks later]

User says "/resume"
  → /resume slash command fires
  → invokes resuming-from-handoff skill
  → skill lists ./.handoffs/*.md by mtime desc
  → shows picker or auto-loads single result
  → reads chosen file, summarizes state
  → asks user to confirm continuation
```

## plugin.json

```json
{
  "name": "context-handoff",
  "version": "0.1.0",
  "description": "Write a handoff.md before context runs out and resume cleanly in the next session",
  "author": {
    "name": "se4thvin",
    "url": "https://github.com/se4thvin"
  }
}
```

## README sections

1. **What it does** — one paragraph + a tiny example interaction.
2. **Install** — `/plugin marketplace add se4thvin/context-handoff` then `/plugin install context-handoff`.
3. **Usage** — `/handoff`, `/handoff <label>`, `/resume`, `/resume <n>`.
4. **Configuration** — `HANDOFF_DIR` env var with example.
5. **When it triggers** — the narrowed conditions from `writing-handoff`'s description.
6. **License** — MIT.

## Testing approach

Per the writing-skills TDD discipline, each skill needs pressure-tested baselines:

- **writing-handoff over-trigger test:** Subagent given a long session ending with "ok let's wrap up for today" should NOT write a handoff. With skill present, it should comply with the "do NOT fire on generic session-ending phrases" rule.
- **writing-handoff under-trigger test:** Subagent given an auto-compaction system reminder should write the handoff without further prompting.
- **resuming-from-handoff picker test:** Subagent in a dir with 3+ handoffs should present the picker, not silently load the newest.
- **End-to-end test:** Write a handoff in one session, start a new session in the same dir, run `/resume`, verify the summary matches.

## Open questions

None at design time. Implementation plan will surface any.

## Out of scope (for future versions)

- `PreCompact` hook that auto-triggers writing-handoff before compaction (Approach 3 from brainstorming).
- Cloud sync of handoffs.
- A `/handoffs list` command for browsing without resuming.
- Per-handoff tags / search.
