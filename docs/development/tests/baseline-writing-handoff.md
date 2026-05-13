# RED baseline — writing-handoff skill

Behavior of a vanilla Claude subagent (general-purpose, opus, no skill installed) when given each trigger scenario. Captured before writing the skill so Task 4 (GREEN) has something to measure against.

**Date:** 2026-05-12

> Note: absolute paths in the verbatim baseline response below have been redacted to <cwd>/.

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
> I would save a state file capturing the current session context so we can resume after compaction. Specifically, I'd write to `<cwd>/.claude-session-state.md` (project-local, easy to find on resume) containing: the current task and where we are in it, the files we've been editing with their absolute paths, any in-flight changes not yet committed, the next concrete step to take, and any decisions/constraints established earlier in the conversation that aren't obvious from the code. Approximate contents would be something like: a "Current task" line, a "Files in play" list with absolute paths, a "Status" line (e.g., "midway through refactoring X, tests passing for Y, Z still broken"), a "Next step" line with the exact next action, and a "Context/decisions" section noting things like chosen library versions, naming conventions agreed on, or paths we've ruled out. I would not run git commands, commit, or modify source files as part of this — the goal is purely to persist enough breadcrumbs that a fresh context can pick up without re-deriving everything. I'd then tell the user the file path so they can reference it (e.g., "read <cwd>/.claude-session-state.md") when starting the next session.

**Outcome:** Vanilla Claude DOES recognize this as a context-save situation and improvises. But:
- Path is non-standard: `<cwd>/.claude-session-state.md` in project root rather than a `.handoffs/` subdirectory.
- Filename is a single fixed name — overwrites prior handoffs, no history.
- No timestamp in the file or filename.
- Structure is ad-hoc, not standardized.

The skill needs to standardize the path (`./.handoffs/YYYY-MM-DD-HHMM.md` or `$HANDOFF_DIR`), filename (timestamped, preserves history), and template (six fixed sections).

---

## GREEN: over-trigger scenario with skill present
**Date:** 2026-05-12
**Setup:** Full SKILL.md prepended to the same prompt as RED baseline 1.

**Agent response (verbatim):**
> I would not fire the writing-handoff skill. The user's message ("ok let's wrap up for today, I'm tired") is a generic session-ending phrase, which the skill explicitly lists as a red flag and a do-NOT-fire condition unless tied to context exhaustion. There's no mention of context limits, no `/handoff` invocation, no auto-compaction reminder, and no explicit request to save state. Per the skill's "If unsure, ASK the user before writing" guidance, my next action would be to simply acknowledge the user's wrap-up in plain text and offer a one-line confirmation question — something like: "Sounds good. Want me to write a handoff file capturing today's progress before you go, or just leave things as-is?" — and wait for their answer before doing anything else.

**Outcome:** PASS. Skill correctly refuses to fire. Bonus: the agent offered a one-line confirmation question, improving on the baseline's passive acknowledgement.

---

## GREEN: under-trigger scenario with skill present
**Date:** 2026-05-12
**Setup:** Full SKILL.md prepended. cwd is `/tmp/handoff-test` (no git, no HANDOFF_DIR). Prompt same as RED baseline 2.

**Agent response (verbatim):**
> # What I Do
>
> The user's message ("running really low on context") is an explicit context-exhaustion signal — that's a clear trigger for the writing-handoff skill. No need to ask for confirmation.
>
> ## Step-by-step
>
> 1. **Resolve output directory:**
>    - `HANDOFF_DIR` is unset, so I use `./.handoffs/` relative to cwd → `/tmp/handoff-test/.handoffs/`.
>    - Run `mkdir -p /tmp/handoff-test/.handoffs` to ensure it exists.
>
> 2. **Collect context:**
>    - Project name: `basename "$(pwd)"` → `handoff-test`
>    - ISO timestamp: `date -u +"%Y-%m-%dT%H:%M:%SZ"` → e.g. `2026-05-12T14:30:00Z`
>    - Filename timestamp: `date +"%Y-%m-%d-%H%M"` → e.g. `2026-05-12-1430`
>    - Git info: no `.git` directory, so branch is `n/a`
>    - Session label: none provided by user → `—`
>
> 3. **Build the filename:**
>    - No label was given, so just the base: `2026-05-12-1430.md`
>    - Full path: `/tmp/handoff-test/.handoffs/2026-05-12-1430.md`
>
> 4. **Write the file** with real session content (not template examples).
>
> 5. **Confirm to the user** with the absolute path and a one-line summary.
>
> (... full headings template followed; abbreviated here for the log. Full response captured in commit-time controller transcript.)

**Outcome:** PASS. Behavior changed from baseline:
- Path: `./.handoffs/2026-05-12-1430.md` (vs baseline's `<cwd>/.claude-session-state.md`).
- Timestamped filename preserves history (vs baseline's fixed name that overwrites).
- All 6 template sections present in order.
- Git fields marked `n/a` (not silently dropped).
- Quality bar honored ("None." for empty sections rather than template examples).

---

## Summary

Both GREEN tests PASS. The writing-handoff skill achieves its design goals:
1. Refuses to over-trigger on generic wrap-up phrasing while still being helpful (offers confirmation).
2. Correctly triggers on context-pressure phrasing AND produces standardized output (path, filename format, template, git handling).

No skill refactoring needed. Proceeding to Task 5.
