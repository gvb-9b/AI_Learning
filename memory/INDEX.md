# memory/INDEX.md — Master table of contents

> Read this **first** when you need information from project memory.
> It tells you which file to open. Don't open files at random.

## Files in this memory directory

| File | What it holds | Open when |
|------|---------------|-----------|
| [lessons.md](lessons.md) | Mistakes made in past sessions and the rule to avoid each one | Before starting any task — check whether a known mistake applies |
| [decisions.md](decisions.md) | Architectural choices and the reason behind each | Before changing project structure, tech stack, or core conventions |
| [conventions.md](conventions.md) | Code style, naming, file organization rules | Before writing or editing code |

## How to read a memory file (the rule)

1. Open the file and read ONLY its **TABLE OF CONTENTS** (the first ~30 lines).
2. Identify the section whose title + summary match your question.
3. Use `Read` with the `offset` parameter to jump straight to that section's line number. Read only ~40 lines from there.
4. NEVER read a memory file from line 1 to the end.
5. If you add or modify a section, the `memory-keeper` subagent re-indexes the file.

## How to grow this memory

- `/learn <lesson>` — appends a new entry. The `memory-keeper` subagent decides which file, writes the entry, and updates the table of contents.
- Direct edits are fine, but ask `memory-keeper` to re-index afterward so line numbers stay correct.

## How this differs from auto-memory

Claude Code also keeps a machine-local "auto-memory" in `~/.claude/projects/<project>/memory/MEMORY.md`. That memory is written by Claude itself, lives on your machine only, and is NOT committed to git. The `memory/` directory here is the **project** memory: written by you (via `/learn`), committed to git, shared with teammates.
