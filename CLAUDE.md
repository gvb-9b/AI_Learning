# AI_learning — Claude Code learning sandbox

This is a sandbox project for learning how to structure a Claude Code project with specialized subagents, custom slash commands, and a project memory that grows over time. The HTML/CSS/JS files are toy code — the real value is in `.claude/` and `memory/`.

See `GUIDE.md` for the full tutorial in French.

## Project layout

- `index.html`, `index.css`, `script.js` — demo app (toy code)
- `.claude/agents/` — specialized subagents (code-reviewer, explorer, memory-keeper)
- `.claude/commands/` — custom slash commands (`/learn`, `/review-and-fix`)
- `.claude/settings.json` — minimal permissions for safe read commands
- `memory/` — project memory files, indexed by `memory/INDEX.md`
- `GUIDE.md` — the teaching guide

## Memory reading rule (apply to every memory file)

The project keeps long-term knowledge in `memory/*.md`. Each file starts with a **TABLE OF CONTENTS** that maps section titles to line numbers. Reading a full memory file every time wastes context.

When you need information from project memory:

1. Read `@memory/INDEX.md` (it's imported below — already in your context) to decide which file to open.
2. Open the file and read **only its TABLE OF CONTENTS** (first ~30 lines).
3. Use `Read` with `offset` to jump to the line number of the matching section. Read ~40 lines from there.
4. NEVER read a memory file from line 1 to the end. If the table of contents doesn't cover what you need, check another memory file's table of contents — don't dump full files.
5. If you add or modify a section in a memory file, the `memory-keeper` subagent updates the table of contents.

## Master memory index

@memory/INDEX.md

## Project conventions (loaded at startup)

@memory/conventions.md

## When to delegate to which subagent

- **explorer** — find files, search code, answer "where is X defined". Read-only. Use before any non-trivial change.
- **code-reviewer** — review uncommitted changes for bugs, security, style violations. Read-only. Use after writing code, before declaring done.
- **memory-keeper** — record a lesson, decision, or convention into `memory/*.md`. Re-indexes tables of contents. The `/learn` command routes here.

## When to use which slash command

- `/learn <what to remember>` — captures a lesson, decision, or convention into project memory. Routes to `memory-keeper`.
- `/review-and-fix` — runs `code-reviewer` on the current changes, then applies the fixes you approve.

## Standard workflow (per task)

After completing a task (writing code, creating files, etc.):

1. **Review (when needed)** — Run `/review-and-fix` to check for bugs, security issues, style violations, and convention misalignment.
   - **Always review** after significant changes, refactoring, or when you're unsure.
   - **Optional** for trivial edits (one-line fixes, small tweaks).
2. **Approve or adjust** — Select which findings to apply, or discuss if a finding is incorrect.
3. **Learn** — If the reviewer found a systemic issue, offer to `/learn` it so future sessions avoid it.

The more you review, the more the system learns and improves. Reviews are a good habit, not a requirement.

## Conventions

- Keep memory files under 500 lines. If one grows past that, ask whether to split it.
- Use `## ATX-style` headings inside memory files so the table of contents stays predictable.
- Never delete a memory entry without explicit user confirmation.
- This is not a git repository yet — `git diff` won't work. The `code-reviewer` subagent should review the most recently edited files instead.
