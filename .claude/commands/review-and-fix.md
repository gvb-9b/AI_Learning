---
description: Run a code review on the current uncommitted changes, then apply the fixes the user approves.
---

Run a review-and-fix loop on the current code change.

**Step 1 — Review**

Delegate to the `code-reviewer` subagent. Ask it to review the current uncommitted changes:
- If this is a git repo (`git rev-parse --git-dir` succeeds), use `git diff` and `git status`.
- Otherwise, review the files most recently edited (use `ls -lt` to identify them, or ask the user which files to review).

**Step 2 — Present findings**

Show the user the numbered list of findings exactly as the reviewer returned them. Then ask:

> Which findings should I apply? (e.g. "1, 3" or "all" or "none")

**Step 3 — Apply approved fixes**

For each approved finding:
- Open the file at the indicated line.
- Apply the suggested fix using `Edit`.
- Show a one-line confirmation: `Fixed: path/file.ext:LN — <what changed>`.

**Step 4 — Offer to learn (optional)**

If the user rejected a finding because the reviewer was wrong, OR if the user pointed out an issue the reviewer missed, offer:

> Want me to `/learn` the rule "<concise version>" so future reviews catch this?

This is how the system improves. Only offer once per session, and only when there's a concrete rule to capture.
