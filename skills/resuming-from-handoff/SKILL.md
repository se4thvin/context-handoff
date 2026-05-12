---
name: resuming-from-handoff
description: Use when user invokes /recap, says "pick up where we left off", "continue from last session", "what was I working on", or starts a session with a reference to prior work that may be captured in a previous handoff file.
---

# Resuming From a Handoff

## Overview

Find the most relevant handoff file for the current project, summarize it, and confirm with the user before continuing work. Never silently assume which handoff to load when more than one exists.

## When to use

Fire this skill when ANY of:
- The user invokes `/recap` (with or without a numeric argument).
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

   - **Multiple handoffs:** If the user passed a number (`/recap 2`), skip to that index (1-based, newest-first). Otherwise show a numbered picker:
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
