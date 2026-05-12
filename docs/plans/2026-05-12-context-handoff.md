# Context-Handoff Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and publish `se4thvin/context-handoff` — a Claude Code plugin with two skills (`writing-handoff`, `resuming-from-handoff`) and two slash commands (`/handoff`, `/resume`) that lets users save and resume session state via timestamped Markdown handoff files.

**Architecture:** Standard Claude Code plugin layout. Skills are pure Markdown — prompt instructions for Claude — auto-discovered from `skills/`. Slash commands are Markdown files in `commands/` that invoke the skills with arguments. No executable code; behavior is defined by the skill prompts. TDD-for-skills: each skill is validated against pressure-test subagent scenarios before being considered done.

**Tech Stack:** Markdown, JSON (plugin manifest), bash + git for scaffolding. No runtime dependencies.

**Repo:** `/Users/sethvin-nanayakkara/Series/context-handoff/` (local) → `https://github.com/se4thvin/context-handoff` (remote, created in Task 11).

---

## File structure

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
├── docs/
│   ├── specs/2026-05-12-context-handoff-design.md   (exists)
│   └── plans/2026-05-12-context-handoff.md          (this file)
├── README.md
├── LICENSE
└── .gitignore
```

Each file has one responsibility:
- `plugin.json` — plugin metadata
- `writing-handoff/SKILL.md` — when + how to write a handoff
- `resuming-from-handoff/SKILL.md` — when + how to resume from a handoff
- `commands/handoff.md` — `/handoff` entry point
- `commands/resume.md` — `/resume` entry point
- `README.md` — user-facing install + usage docs

---

## Task 1: Initialize repo + scaffolding

**Files:**
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/.gitignore`
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/LICENSE`
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/.claude-plugin/plugin.json`

- [ ] **Step 1: Init git repo**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git init -b main
```

Expected: `Initialized empty Git repository in .../context-handoff/.git/`

- [ ] **Step 2: Create .gitignore**

Path: `/Users/sethvin-nanayakkara/Series/context-handoff/.gitignore`

```
.DS_Store
.handoffs/
node_modules/
*.log
```

- [ ] **Step 3: Create LICENSE (MIT)**

Path: `/Users/sethvin-nanayakkara/Series/context-handoff/LICENSE`

```
MIT License

Copyright (c) 2026 se4thvin

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 4: Create plugin.json**

Path: `/Users/sethvin-nanayakkara/Series/context-handoff/.claude-plugin/plugin.json`

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

- [ ] **Step 5: Verify scaffolding**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
find . -type f -not -path './.git/*' | sort
```

Expected output:
```
./.claude-plugin/plugin.json
./.gitignore
./LICENSE
./docs/plans/2026-05-12-context-handoff.md
./docs/specs/2026-05-12-context-handoff-design.md
```

- [ ] **Step 6: Commit**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add .gitignore LICENSE .claude-plugin/plugin.json docs/
git commit -m "chore: scaffold plugin repo with manifest, license, and design docs"
```

---

## Task 2: Baseline test — writing-handoff over-trigger scenario

**Purpose:** RED phase. Before writing the skill, confirm a Claude subagent without the skill behaves problematically (over-triggers on generic "wrap up" phrasing, OR doesn't know where to write the file).

**Files:**
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/docs/tests/baseline-writing-handoff.md` (log results here)

- [ ] **Step 1: Dispatch baseline subagent with "wrap up" phrasing**

Use the Agent tool with `subagent_type: "general-purpose"` and the following prompt:

```
You are continuing a long coding session. The user has been editing several files and just said:

"ok let's wrap up for today, I'm tired"

What do you do? Describe your next action in one paragraph. If you would write any file to disk, name the path. Do not actually run any tools — just describe what you would do.
```

- [ ] **Step 2: Log the baseline behavior**

Append to `docs/tests/baseline-writing-handoff.md`:

```markdown
## Baseline: over-trigger scenario ("wrap up for today")
**Date:** <run date>
**Agent response (verbatim):**
> <paste response>

**Outcome:** [Did it offer/write a handoff? Where? What rationalization, if any?]
```

- [ ] **Step 3: Dispatch baseline subagent with context-pressure phrasing**

Use the Agent tool with `subagent_type: "general-purpose"`:

```
You are continuing a long coding session. The user just said:

"hey we're running really low on context, can you save state before we lose it?"

What do you do? Describe your next action in one paragraph. If you would write any file to disk, name the path and approximate contents (5-10 lines). Do not actually run any tools — just describe what you would do.
```

- [ ] **Step 4: Log the second baseline**

Append to the same `docs/tests/baseline-writing-handoff.md`:

```markdown
## Baseline: context-pressure scenario ("running low on context")
**Date:** <run date>
**Agent response (verbatim):**
> <paste response>

**Outcome:** [Did it write a handoff? Where? What content?]
```

- [ ] **Step 5: Commit baseline log**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add docs/tests/baseline-writing-handoff.md
git commit -m "test: log writing-handoff baseline (RED phase)"
```

---

## Task 3: Write writing-handoff SKILL.md

**Files:**
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/skills/writing-handoff/SKILL.md`

- [ ] **Step 1: Create the SKILL.md**

Path: `/Users/sethvin-nanayakkara/Series/context-handoff/skills/writing-handoff/SKILL.md`

```markdown
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

5. **Confirm to the user:**
   - Print the absolute file path.
   - Print a one-line summary of what was captured.

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
```

- [ ] **Step 2: Verify SKILL.md syntax**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
head -5 skills/writing-handoff/SKILL.md
```

Expected: YAML frontmatter with `name:` and `description:` fields.

- [ ] **Step 3: Commit**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add skills/writing-handoff/SKILL.md
git commit -m "feat: add writing-handoff skill"
```

---

## Task 4: GREEN test — writing-handoff with skill present

**Purpose:** Verify the skill changes subagent behavior in both directions: over-trigger case now refuses; under-trigger case now writes the file correctly.

**Files:**
- Modify: `/Users/sethvin-nanayakkara/Series/context-handoff/docs/tests/baseline-writing-handoff.md` (append GREEN results)

- [ ] **Step 1: Dispatch with-skill subagent — over-trigger case**

Read the skill content into a variable for the prompt:

```bash
SKILL_CONTENT=$(cat /Users/sethvin-nanayakkara/Series/context-handoff/skills/writing-handoff/SKILL.md)
```

Use the Agent tool with `subagent_type: "general-purpose"` and this prompt (substitute `$SKILL_CONTENT` literally):

```
You have access to the following skill. Treat its rules as authoritative.

--- BEGIN SKILL ---
<paste the contents of skills/writing-handoff/SKILL.md here>
--- END SKILL ---

The user has been editing several files and just said:

"ok let's wrap up for today, I'm tired"

What do you do? Describe your next action in one paragraph. Do not actually run any tools.
```

- [ ] **Step 2: Verify over-trigger now refuses**

Expected: agent does NOT write a handoff. It either does nothing, suggests committing changes, or explicitly says this isn't a context-exhaustion signal.

Append result to `docs/tests/baseline-writing-handoff.md`:

```markdown
## GREEN: over-trigger scenario with skill
**Date:** <run date>
**Agent response (verbatim):**
> <paste response>

**Outcome:** [PASS if no handoff offered, FAIL otherwise]
```

- [ ] **Step 3: Dispatch with-skill subagent — under-trigger case**

Use the Agent tool with `subagent_type: "general-purpose"` and this prompt:

```
You have access to the following skill. Treat its rules as authoritative.

--- BEGIN SKILL ---
<paste the contents of skills/writing-handoff/SKILL.md here>
--- END SKILL ---

You are in working directory /tmp/handoff-test (assume it exists, no git repo). The user just said:

"hey we're running really low on context, can you save state before we lose it?"

What do you do? Describe step-by-step, including the exact file path you would write to and the structure of the file content (you can abbreviate the body, but show the headings). Do not actually run any tools.
```

- [ ] **Step 4: Verify under-trigger now writes correctly**

Expected:
- Path is `/tmp/handoff-test/.handoffs/<YYYY-MM-DD-HHMM>.md`
- Content has all six template sections in order
- Git fields are marked `n/a`

Append:

```markdown
## GREEN: under-trigger scenario with skill
**Date:** <run date>
**Agent response (verbatim):**
> <paste response>

**Outcome:** [PASS if path + structure correct, FAIL otherwise]
```

- [ ] **Step 5: If either test FAILED, refactor the skill**

Find the gap or rationalization in the agent's response. Add an explicit counter to the "Red flags" table or the "When to use" section. Re-run only the failing test until it passes.

- [ ] **Step 6: Commit test results**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add docs/tests/baseline-writing-handoff.md
git commit -m "test: writing-handoff GREEN phase verified"
```

---

## Task 5: Baseline test — resuming-from-handoff picker behavior

**Files:**
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/docs/tests/baseline-resuming.md`
- Create: 3 fixture handoff files in `/tmp/handoff-resume-test/.handoffs/`

- [ ] **Step 1: Create fixture handoffs**

```bash
mkdir -p /tmp/handoff-resume-test/.handoffs
cat > /tmp/handoff-resume-test/.handoffs/2026-05-11-1742.md <<'EOF'
# Handoff — handoff-resume-test — 2026-05-11T17:42:00Z

**Working directory:** /tmp/handoff-resume-test
**Git branch:** n/a
**Session label:** —

## Current task
WIP: parser bug in token stream.
EOF
cat > /tmp/handoff-resume-test/.handoffs/2026-05-12-0915-before-refactor.md <<'EOF'
# Handoff — handoff-resume-test — 2026-05-12T09:15:00Z

**Working directory:** /tmp/handoff-resume-test
**Git branch:** n/a
**Session label:** before-refactor

## Current task
About to split the auth middleware into two files.
EOF
cat > /tmp/handoff-resume-test/.handoffs/2026-05-12-1430.md <<'EOF'
# Handoff — handoff-resume-test — 2026-05-12T14:30:00Z

**Working directory:** /tmp/handoff-resume-test
**Git branch:** n/a
**Session label:** —

## Current task
Refactoring auth middleware — two helpers extracted, tests passing.
EOF
ls -la /tmp/handoff-resume-test/.handoffs/
```

Expected: 3 files listed.

- [ ] **Step 2: Dispatch baseline (no skill) — picker scenario**

Use the Agent tool with `subagent_type: "general-purpose"`:

```
The user just said: "/resume"

The current working directory is /tmp/handoff-resume-test and contains a .handoffs/ subdirectory with three Markdown files (handoffs from previous sessions). What do you do? Describe step-by-step. You may run read-only tools (ls, cat) to inspect, but do not write anything.
```

- [ ] **Step 3: Log baseline**

Write to `docs/tests/baseline-resuming.md`:

```markdown
## Baseline: 3-handoff picker
**Date:** <run date>
**Agent response (verbatim):**
> <paste response>

**Outcome:** [Did the agent show a numbered picker? Silently auto-pick newest? Something else?]
```

- [ ] **Step 4: Commit**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add docs/tests/baseline-resuming.md
git commit -m "test: log resuming-from-handoff baseline (RED phase)"
```

---

## Task 6: Write resuming-from-handoff SKILL.md

**Files:**
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/skills/resuming-from-handoff/SKILL.md`

- [ ] **Step 1: Create the SKILL.md**

Path: `/Users/sethvin-nanayakkara/Series/context-handoff/skills/resuming-from-handoff/SKILL.md`

```markdown
---
name: resuming-from-handoff
description: Use when user invokes /resume, says "pick up where we left off", "continue from last session", "what was I working on", or starts a session with a reference to prior work that may be captured in a previous handoff file.
---

# Resuming From a Handoff

## Overview

Find the most relevant handoff file for the current project, summarize it, and confirm with the user before continuing work. Never silently assume which handoff to load when more than one exists.

## When to use

Fire this skill when ANY of:
- The user invokes `/resume` (with or without a numeric argument).
- The user says: "pick up where we left off", "continue from last session", "what was I working on", "resume from yesterday".
- The user starts a session by referencing prior work that's likely captured in a handoff.

## What to do

1. **Resolve search directories** in this order:
   - `./.handoffs/` (project-local) — always check.
   - `$HANDOFF_DIR` — if set AND different from `./.handoffs/`, also check.

2. **List handoffs** by modification time, newest first. If both dirs apply, merge the lists by mtime.
   ```bash
   ls -1t ./.handoffs/*.md 2>/dev/null
   ```
   Optionally include `$HANDOFF_DIR/*.md` too.

3. **Branch on count:**

   - **Zero handoffs:** Tell the user "No handoffs found in `<path>`. Use /handoff next session to create one." Stop.

   - **Exactly one handoff:** Read it. Show a 3-5 line summary covering: project, timestamp, current task, next steps. Ask: "Continue from these next steps?" Wait for confirmation before acting.

   - **Multiple handoffs:** If the user passed a number (`/resume 2`), skip to that index (1-based, newest-first). Otherwise show a numbered picker:
     ```
     Found 3 handoffs:
     1. 2026-05-12 14:30 — "Refactoring auth middleware — two helpers extracted..."
     2. 2026-05-12 09:15 [before-refactor] — "About to split the auth middleware..."
     3. 2026-05-11 17:42 — "WIP: parser bug in token stream."

     Which one? (1-3, or "q" to cancel)
     ```
     Preview text: the first non-empty line under `## Current task` in each file, truncated to ~70 chars.
     Wait for the user's choice. Then load it, summarize, and confirm before acting.

4. **After loading**, summarize the handoff in 3-5 lines. Do NOT dump the full file contents back to the user — they wrote it. Show:
   - Project + timestamp + label (if any)
   - Current task (one line)
   - Top 1-3 next steps
   - Any blockers

5. **Ask explicitly** before taking action on the next steps. Example: "Ready to continue with [first next step]? Or do you want to do something else first?"

## Red flags — stop and re-check

| Thought | Reality |
|---------|---------|
| "Multiple handoffs but I'll just load the newest" | No — show the picker. The user may want an older one. |
| "I'll dump the whole handoff file to the user" | They wrote it. Summarize instead. |
| "I'll start working on Next Steps without asking" | Confirm first. Time may have passed; priorities may have shifted. |
| "No handoffs found — I'll improvise from project context" | Tell the user explicitly, don't pretend you have state you don't. |
```

- [ ] **Step 2: Verify SKILL.md**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
head -5 skills/resuming-from-handoff/SKILL.md
```

Expected: YAML frontmatter with `name:` and `description:`.

- [ ] **Step 3: Commit**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add skills/resuming-from-handoff/SKILL.md
git commit -m "feat: add resuming-from-handoff skill"
```

---

## Task 7: GREEN test — resuming-from-handoff with skill present

**Files:**
- Modify: `/Users/sethvin-nanayakkara/Series/context-handoff/docs/tests/baseline-resuming.md` (append GREEN results)

- [ ] **Step 1: Dispatch with-skill subagent — picker scenario**

Use the Agent tool with `subagent_type: "general-purpose"`:

```
You have access to the following skill. Treat its rules as authoritative.

--- BEGIN SKILL ---
<paste the contents of skills/resuming-from-handoff/SKILL.md here>
--- END SKILL ---

The user just said: "/resume"

The current working directory is /tmp/handoff-resume-test. It contains a .handoffs/ subdirectory with these files (newest first):
  - 2026-05-12-1430.md (Current task: "Refactoring auth middleware — two helpers extracted, tests passing.")
  - 2026-05-12-0915-before-refactor.md (Current task: "About to split the auth middleware into two files.")
  - 2026-05-11-1742.md (Current task: "WIP: parser bug in token stream.")

What do you do? Describe step-by-step. Show the exact text you would present to the user.
```

- [ ] **Step 2: Verify picker is shown**

Expected:
- Agent shows a 3-item numbered picker
- Each item shows date, time, optional label, and a preview from "Current task"
- Agent does NOT auto-load and does NOT take action on next steps
- Agent waits for the user's pick

- [ ] **Step 3: Dispatch with-skill subagent — single-handoff scenario**

Same prompt but say "the .handoffs/ subdirectory contains exactly one file: 2026-05-12-1430.md..."

Expected:
- Agent reads it
- Summarizes in 3-5 lines
- Asks for confirmation before acting

- [ ] **Step 4: Dispatch with-skill subagent — indexed scenario**

User said `/resume 2`. Expected: agent loads handoff index 2 directly (without picker), summarizes, asks for confirmation.

- [ ] **Step 5: Log all three results**

Append to `docs/tests/baseline-resuming.md`:

```markdown
## GREEN: picker scenario with skill
**Outcome:** [PASS/FAIL]

## GREEN: single-handoff scenario with skill
**Outcome:** [PASS/FAIL]

## GREEN: indexed scenario (/resume 2) with skill
**Outcome:** [PASS/FAIL]
```

- [ ] **Step 6: Refactor if any FAILED**

Add explicit counters to the skill's "Red flags" table or "What to do" section. Re-run failing tests until PASS.

- [ ] **Step 7: Commit**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add docs/tests/baseline-resuming.md
git commit -m "test: resuming-from-handoff GREEN phase verified"
```

---

## Task 8: Write /handoff slash command

**Files:**
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/commands/handoff.md`

- [ ] **Step 1: Create the command file**

Path: `/Users/sethvin-nanayakkara/Series/context-handoff/commands/handoff.md`

```markdown
---
description: Write a session handoff to .handoffs/ before context runs out
argument-hint: "[optional-label]"
---

The user has invoked /handoff. Invoke the `writing-handoff` skill and write the handoff now.

If arguments were provided, treat them as the session label and pass them through to the filename (after slugification).

Arguments: $ARGUMENTS
```

- [ ] **Step 2: Commit**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add commands/handoff.md
git commit -m "feat: add /handoff slash command"
```

---

## Task 9: Write /resume slash command

**Files:**
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/commands/resume.md`

- [ ] **Step 1: Create the command file**

Path: `/Users/sethvin-nanayakkara/Series/context-handoff/commands/resume.md`

```markdown
---
description: Resume from a prior session's handoff in .handoffs/
argument-hint: "[handoff-number]"
---

The user has invoked /resume. Invoke the `resuming-from-handoff` skill.

If a numeric argument was provided, skip the picker and jump to that handoff (1-based, newest-first). Otherwise show the picker (or auto-load if there's only one handoff).

Arguments: $ARGUMENTS
```

- [ ] **Step 2: Commit**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add commands/resume.md
git commit -m "feat: add /resume slash command"
```

---

## Task 10: Write README

**Files:**
- Create: `/Users/sethvin-nanayakkara/Series/context-handoff/README.md`

- [ ] **Step 1: Create README.md**

Path: `/Users/sethvin-nanayakkara/Series/context-handoff/README.md`

```markdown
# context-handoff

A Claude Code plugin that lets you save your session state before context runs out, and resume cleanly in the next session.

## What it does

When you're getting close to the context limit (or just want to checkpoint), `/handoff` writes a structured Markdown file summarizing what you've done, what's in progress, and what's next. In a new session, `/resume` finds that file, summarizes it, and asks where you want to continue.

```
> /handoff before-refactor
✓ Wrote ./.handoffs/2026-05-12-1430-before-refactor.md
  Captured: 3 files changed, 1 commit, 2 next steps

> /resume
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
| `/resume` | Show a picker of recent handoffs, or auto-load if only one exists. |
| `/resume <n>` | Skip the picker, load handoff #n (1-based, newest first). |

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
```

- [ ] **Step 2: Commit**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git add README.md
git commit -m "docs: add README"
```

---

## Task 11: Publish to GitHub

**Files:** none modified — this is a push step.

- [ ] **Step 1: Create the GitHub repo**

Two options:

**Option A — via `gh` CLI (preferred if installed):**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
gh repo create se4thvin/context-handoff --public --source=. --description "Save and resume Claude Code session state via handoff.md" --remote=origin
```

Expected: repo created, `origin` remote added.

**Option B — manual (if `gh` not configured):**

1. Open https://github.com/new in a browser.
2. Owner: `se4thvin`. Repo name: `context-handoff`. Public. Do NOT initialize with README/license (we already have them).
3. Run locally:
   ```bash
   cd /Users/sethvin-nanayakkara/Series/context-handoff
   git remote add origin https://github.com/se4thvin/context-handoff.git
   ```

**STOP and confirm with the user which option to use before running.**

- [ ] **Step 2: Push**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git push -u origin main
```

Expected: branch `main` pushed to remote.

- [ ] **Step 3: Verify install path**

Print to the user:

```
Plugin published. Install with:
  /plugin marketplace add se4thvin/context-handoff
  /plugin install context-handoff
```

- [ ] **Step 4: (Optional) Tag v0.1.0**

```bash
cd /Users/sethvin-nanayakkara/Series/context-handoff
git tag v0.1.0
git push origin v0.1.0
```

---

## Self-review notes

- **Spec coverage:** Each component in the spec maps to a task (scaffolding → T1; writing-handoff skill → T2-T4; resuming-from-handoff skill → T5-T7; slash commands → T8-T9; README → T10; distribution → T11). Testing approach from spec covered by T2/T4/T5/T7.
- **Placeholders:** None. Every file's contents are inlined.
- **Type/name consistency:** Skill names (`writing-handoff`, `resuming-from-handoff`), command names (`/handoff`, `/resume`), env var (`HANDOFF_DIR`), and default path (`./.handoffs/`) match across all tasks and the spec.
- **YAGNI:** No `PreCompact` hook (spec out-of-scope), no `/handoffs list`, no cloud sync. Just the v0.1 surface.
