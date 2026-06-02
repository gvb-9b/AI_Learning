---
description: Record a lesson, decision, or convention into project memory so future sessions apply it.
argument-hint: <what to remember>
---

The user wants to save the following into project memory so future sessions can apply it:

> $ARGUMENTS

Steps:

1. Decide which memory file fits best:
   - `memory/lessons.md` → a mistake to avoid next time
   - `memory/decisions.md` → a structural choice with a reason
   - `memory/conventions.md` → a code style or organization rule

   If ambiguous, ask the user to confirm (one short question, two options max).

2. Delegate to the `memory-keeper` subagent. Pass it:
   - The exact content above.
   - Your chosen target file.
   - Today's date in YYYY-MM-DD format.

3. After `memory-keeper` returns:
   - Show the user one line confirming what was written and where (e.g. "Saved to memory/lessons.md → ## avoid-mock-database").
   - If `memory-keeper` reported updated line numbers, mention them.

Keep your own output to 3 lines max. The subagent does the writing.
