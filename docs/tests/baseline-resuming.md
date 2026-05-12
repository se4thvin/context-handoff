# RED baseline — resuming-from-handoff skill

Behavior of a vanilla Claude subagent (general-purpose, opus, no skill installed) when given `/resume` in a project with three existing handoff files. Captured before writing the skill so Task 7 (GREEN) has a baseline to measure against.

**Date:** 2026-05-12

## Fixture setup

Created `/tmp/handoff-resume-test/.handoffs/` with three files (newest first by mtime):
- `2026-05-12-1430.md` — "Refactoring auth middleware — two helpers extracted, tests passing."
- `2026-05-12-0915-before-refactor.md` — "About to split the auth middleware into two files."
- `2026-05-11-1742.md` — "WIP: parser bug in token stream."

---

## Baseline: 3-handoff picker scenario

**Prompt:**
> The user just said: "/resume"
>
> The current working directory is /tmp/handoff-resume-test and contains a .handoffs/ subdirectory with three Markdown files (handoffs from previous sessions). You may run read-only tools (ls, cat) to inspect, but do not write or modify anything.
>
> What do you do? Describe step-by-step what you would actually do next. If you would print or show anything to the user, show the exact text. If you would take action on the contents of a handoff (e.g., start editing files), say so.

**Agent response (verbatim):**
> ## Step 3: Decide what to do
>
> The three handoffs describe three different tasks (parser bug, then a checkpoint before auth refactor, then mid-refactor). The most recent is from 14:30 today, and it describes work clearly in flight: "two helpers extracted, tests passing." That's the natural resume point.
>
> A few things I would NOT do:
> - I would not start editing files. The handoffs are short status notes — they don't describe a concrete next action with enough specificity for me to just go execute. And the user only said `/resume`, not "/resume and keep going."
> - I would not silently pick the newest file and pretend the others don't exist — the `before-refactor` label suggests the user sometimes saves intentional checkpoints they may want to roll back to, so they should know all three are present.
> - I would not create or modify any files (per the instructions, and per `/resume` semantics — reading handoffs is read-only).
>
> What I'd do is surface the handoffs, recommend the most recent one as the resume point, and ask the user to confirm before doing anything.
>
> ## Step 4: What I'd print to the user
>
> Here's the exact text I'd show:
>
> ---
>
> Found 3 handoffs in `/tmp/handoff-resume-test/.handoffs/`:
>
> | When | Label | Task |
> |---|---|---|
> | 2026-05-11 17:42 | — | WIP: parser bug in token stream. |
> | 2026-05-12 09:15 | before-refactor | About to split the auth middleware into two files. |
> | **2026-05-12 14:30** | — | **Refactoring auth middleware — two helpers extracted, tests passing.** |
>
> The most recent (today, 14:30) looks like the natural resume point: an auth middleware refactor is in progress, two helpers have been extracted, and tests are passing. The 09:15 entry is a `before-refactor` checkpoint for that same work. The 05-11 entry is an unrelated parser bug that may still be open.
>
> A few things to flag before I dive in:
> - The handoff is a one-liner — it tells me where you stopped but not what the next step is. Do you want to (a) continue the refactor (e.g., extract more helpers, wire things up, update callers), (b) wrap it up (commit, open PR), or (c) something else?
> - Should I treat the 05-11 parser bug as still open, or has that been resolved?
> - Want me to look at the actual working tree to see what state the auth middleware is in right now?
>
> Tell me which thread to pick up and I'll go.
>
> ---
>
> Then I'd stop and wait for the user's answer before taking any action on the codebase.

**Outcome:** Vanilla Claude does NOT silently auto-pick the newest handoff — it surfaces all three and waits for confirmation. This is good (don't break this). However:
- Format is a table, not a numbered picker. The skill standardizes on a numbered list (1-N) so the user can answer `/resume 2` to skip the picker on subsequent invocations.
- The baseline sorts ascending by date (oldest first). The skill should sort newest-first.
- The baseline highlights the newest with bold and adds reasoning. The skill's picker is more minimal — just date, optional label, and preview.
- The baseline asks rich follow-up questions about "what should I do next." The skill should be more disciplined: load the chosen handoff, summarize in 3-5 lines, ask whether to continue from "Next steps".

The skill needs to enforce: numbered list (1-N, newest first), succinct preview (date + label + first-line of Current task), no auto-loading, no follow-up question fanout.

---

## GREEN: 3-handoff picker scenario with skill present
**Date:** 2026-05-12
**Setup:** Full SKILL.md prepended to the same prompt as RED baseline.

**Agent response (verbatim):**
> Found 3 handoffs:
> 1. 2026-05-12 14:30 — "Refactoring auth middleware — two helpers extracted, tests passing."
> 2. 2026-05-12 09:15 [before-refactor] — "About to split the auth middleware into two files."
> 3. 2026-05-11 17:42 — "WIP: parser bug in token stream."
>
> Which one? (1-3, or "q" to cancel)

**Outcome:** PASS. Output matches the skill's specification character-for-character — newest first, optional label in brackets, preview from "Current task", final prompt line. No auto-pick.

---

## GREEN: single-handoff scenario with skill present
**Date:** 2026-05-12
**Setup:** cwd `/tmp/handoff-single-test`, `.handoffs/` contains exactly one structured handoff file.

**Agent response (verbatim):**
> Found 1 handoff in `/tmp/handoff-single-test/.handoffs/`:
>
> **handoff-single-test** — 2026-05-12 14:30 UTC
> - **Current task:** Refactoring auth middleware — two helpers (`verifyToken()`, `loadUserProfile()`) extracted into `src/middleware/auth-helpers.ts`; all 12 unit tests passing; committed as `a1b2c3d`.
> - **Next steps:** (1) Wire helpers into `authMiddleware()` in `src/middleware/auth.ts`, (2) update integration tests, (3) remove now-redundant inline logic.
> - **Blockers:** None.
>
> Continue from these next steps?

**Outcome:** PASS. Clean 3-5 line summary (project, timestamp, current task, next steps, blockers). Did NOT dump full file. Asked the exact confirmation question the skill prescribes.

---

## GREEN: indexed `/resume 2` scenario with skill present
**Date:** 2026-05-12
**Setup:** cwd `/tmp/handoff-resume-test` with 3 handoffs. User says "/resume 2".

**Agent response (verbatim, abbreviated to the user-visible output block — full step-by-step rationale captured in controller transcript):**
> Loaded handoff 2 of 3: .handoffs/2026-05-12-0915-before-refactor.md
> (session label: before-refactor, saved 2026-05-12 09:15)
>
> Summary:
> - State at save: about to split the auth middleware into two files — pre-refactor checkpoint.
> - This is the snapshot taken BEFORE the work captured in today's later handoff (1430), which reports the two helpers already extracted and tests passing.
> - Picking this one means rewinding to the pre-split starting point rather than continuing from the completed refactor.
>
> Heads up: handoff 1 (2026-05-12-1430.md) is newer and shows the refactor as done. Confirm you want to resume from the pre-refactor checkpoint and not the newer one — want me to continue from this handoff's next steps?

**Outcome:** PASS. Correctly parsed the argument, skipped the picker, identified index 2 = `before-refactor`, summarized minimally, asked for confirmation. Bonus: thoughtful note that handoff 1 is newer.

---

## Summary

All three GREEN tests PASS. The `resuming-from-handoff` skill achieves its design goals:
1. Standardizes the picker format (numbered, newest-first, optional `[label]`, preview from Current task).
2. Handles the single-handoff case with a clean summary + the prescribed confirmation question.
3. Honors the indexed-resume optimization (`/resume N` skips the picker) and still asks for confirmation before acting.

No skill refactoring needed. Proceeding to Task 8.
