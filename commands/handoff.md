---
description: Write a session handoff to .handoffs/ before context runs out
argument-hint: "[optional-label]"
---

The user has invoked /handoff. Invoke the `writing-handoff` skill and write the handoff now.

If arguments were provided, treat them as the session label and pass them through to the filename (after slugification).

Arguments: $ARGUMENTS
