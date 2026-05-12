# RED baseline — writing-handoff skill

Behavior of a vanilla Claude subagent (general-purpose, opus, no skill installed) when given each trigger scenario. Captured before writing the skill so Task 4 (GREEN) has something to measure against.

**Date:** 2026-05-12

---

## Baseline: over-trigger scenario ("wrap up for today")

**Prompt:**
> You are continuing a long coding session. The user has been editing several files and just said: "ok let's wrap up for today, I'm tired"
> What do you do? Describe your next action in one paragraph. If you would write any file to disk, name the path. Do not actually run any tools — just describe what you would do.

**Agent response (verbatim):**
> I would simply acknowledge the user's message and wrap up conversationally — something like "Sounds good, get some rest. Pick this back up whenever you're ready." I would not write any files to disk, would not create a summary or handoff document (no `SESSION_NOTES.md`, `PROGRESS.md`, `TODO.md`, or anything similar), would not commit anything to git, and would not call any tools. The user said they're tired and want to stop; that's not a request for me to be proactive about documenting state or tidying up. If they wanted a recap or a commit before stopping, they'd ask. Doing extra work here would ignore what they actually said and burn their attention when they've explicitly signaled they're done for the day.

**Outcome:** Vanilla Claude correctly does NOT write a handoff. This is desirable. The skill's job here is to PRESERVE this behavior — i.e., the skill must NOT cause an over-trigger when adding rules. Task 4 will verify the skill doesn't break this.

---

## Baseline: context-pressure scenario ("running low on context")

**Prompt:**
> You are continuing a long coding session. The user just said: "hey we're running really low on context, can you save state before we lose it?"
> What do you do? Describe your next action in one paragraph. If you would write any file to disk, name the path and approximate contents (5-10 lines). Do not actually run any tools — just describe what you would do.

**Agent response (verbatim):**
> I would save a state file capturing the current session context so we can resume after compaction. Specifically, I'd write to `/Users/sethvin-nanayakkara/Series/.claude-session-state.md` (project-local, easy to find on resume) containing: the current task and where we are in it, the files we've been editing with their absolute paths, any in-flight changes not yet committed, the next concrete step to take, and any decisions/constraints established earlier in the conversation that aren't obvious from the code. Approximate contents would be something like: a "Current task" line, a "Files in play" list with absolute paths, a "Status" line (e.g., "midway through refactoring X, tests passing for Y, Z still broken"), a "Next step" line with the exact next action, and a "Context/decisions" section noting things like chosen library versions, naming conventions agreed on, or paths we've ruled out. I would not run git commands, commit, or modify source files as part of this — the goal is purely to persist enough breadcrumbs that a fresh context can pick up without re-deriving everything. I'd then tell the user the file path so they can reference it (e.g., "read /Users/sethvin-nanayakkara/Series/.claude-session-state.md") when starting the next session.

**Outcome:** Vanilla Claude DOES recognize this as a context-save situation and improvises. But:
- Path is non-standard: `<cwd>/.claude-session-state.md` in project root rather than a `.handoffs/` subdirectory.
- Filename is a single fixed name — overwrites prior handoffs, no history.
- No timestamp in the file or filename.
- Structure is ad-hoc, not standardized.

The skill needs to standardize the path (`./.handoffs/YYYY-MM-DD-HHMM.md` or `$HANDOFF_DIR`), filename (timestamped, preserves history), and template (six fixed sections).
