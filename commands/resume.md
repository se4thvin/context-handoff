---
description: Resume from a prior session's handoff in .handoffs/
argument-hint: "[handoff-number]"
---

The user has invoked /resume. Invoke the `resuming-from-handoff` skill.

If a numeric argument was provided, skip the picker and jump to that handoff (1-based, newest-first). Otherwise show the picker (or auto-load if there's only one handoff).

Arguments: $ARGUMENTS
