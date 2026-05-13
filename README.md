# context-handoff

[![Version](https://img.shields.io/github/v/tag/se4thvin/context-handoff?label=version&sort=semver)](https://github.com/se4thvin/context-handoff/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-orange)](https://claude.com/claude-code)

A Claude Code plugin that lets you save your session state before context runs out, and resume cleanly in the next session.

## What it does

When you're getting close to the context limit (or just want to checkpoint), `/handoff` writes a structured Markdown file summarizing what you've done, what's in progress, and what's next. In a new session, `/handoff-resume` finds that file, summarizes it, and asks where you want to continue.

```
> /handoff before-refactor
✓ Wrote ./.handoffs/2026-05-12-1430-before-refactor.md
  Captured: 3 files changed, 1 commit, 2 next steps

> /handoff-resume
Found 3 handoffs:
1. 2026-05-12 14:30 — "Refactoring auth middleware..."
2. 2026-05-12 09:15 [before-refactor] — "About to split auth..."
3. 2026-05-11 17:42 — "WIP: parser bug..."

Which one? (1-3)
```

## Install

```
/plugin marketplace add se4thvin/context-handoff
/plugin install context-handoff
```

## Usage

| Command | What it does |
|---------|--------------|
| `/handoff` | Write a handoff with a timestamp-only filename. |
| `/handoff <label>` | Write a handoff with `<label>` appended to the filename (slugified). |
| `/handoff-resume` | Show a picker of recent handoffs, or auto-summarize if only one exists (then asks before continuing). |
| `/handoff-resume <n>` | Skip the picker, load handoff #n (1-based, newest first). |

You can also just say "write a handoff" or "pick up where we left off" — the skills fire on those phrases too.

## When does it trigger automatically?

The `writing-handoff` skill is intentionally conservative. It triggers on:
- Explicit `/handoff` invocation.
- Phrases like "write a handoff", "save state", "save session".
- Context-exhaustion signals: "running low on context", "almost out of context".
- Auto-compaction system reminders.

It does NOT trigger on generic session-ending phrases like "wrap up" or "let's stop for today" — those don't mean context is the problem.

## Configuration

| Env var | Default | Effect |
|---------|---------|--------|
| `HANDOFF_DIR` | `./.handoffs/` (per-project) | Override the directory where handoffs are written and searched. Useful for centralizing: `export HANDOFF_DIR=~/.claude/handoffs`. |

## What gets captured

The handoff file has these sections:

- **Current task** — where you left off
- **Completed this session** — decisions, file changes, commits
- **In progress** — uncommitted changes, partial work
- **Next steps** — prioritized TODOs
- **Blockers / open questions** — things waiting on input
- **Key context** — file paths, function names, conventions, gotchas

Plus a header with project name, working directory, git branch, and timestamp.

## License

MIT — see [LICENSE](LICENSE).
