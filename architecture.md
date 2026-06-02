# Claude Code Self-Learning Architecture — Installation Guide

## Overview

This document contains **everything you need to replicate the self-learning Claude Code setup in any existing project**. Copy-paste the prompt below into Claude Code, and it will bootstrap the entire system (CLAUDE.md, subagents, slash commands, project memory).

The system is designed so that:
- **Conventions are applied automatically** (loaded at startup)
- **Every task includes a mandatory code review** (100% coverage, zero silent errors)
- **Lessons are captured and reused** across sessions (learning loop)
- **Memory is indexed efficiently** (small tokens cost, no full-file reads)

---

## Prerequisites

- Claude Code running in your project directory (any existing project is fine)
- A `.git` repository (optional but recommended; if not present, the setup still works)
- ~5 minutes

---

## The Prompt: Copy & Paste Into Claude Code

```
I want to set up a self-learning Claude Code system in this project.

Here's what I need:

1. Create .claude/agents/ with three specialized subagents:
   - code-reviewer.md (read-only, reviews code for bugs/style/security)
   - explorer.md (read-only, searches code and finds definitions)
   - memory-keeper.md (read-write, maintains project memory files)

2. Create .claude/commands/ with two slash commands:
   - learn.md (user types /learn "what to remember" → memory-keeper records it)
   - review-and-fix.md (user types /review-and-fix → code-reviewer checks + apply fixes)

3. Create memory/ directory with indexed memory files:
   - INDEX.md (master table of contents)
   - lessons.md (mistakes to avoid)
   - decisions.md (architectural choices)
   - conventions.md (code style rules)

4. Create CLAUDE.md with:
   - Project description
   - Memory reading rule (read TABLE OF CONTENTS first, jump to section by line number)
   - Import of memory/INDEX.md and memory/conventions.md at startup
   - Standard workflow: task → /review-and-fix (mandatory) → /learn if needed

5. Create .claude/settings.json with minimal permissions (read-only bash commands)

6. Create GUIDE.md explaining the whole system

Start with step 1 (the subagents). For each file, show me the content before writing.
```

---

## What Gets Created

```
your-project/
├── CLAUDE.md                          # Project instructions + imports
├── GUIDE.md                           # Teaching guide (optional, but useful)
│
├── .claude/
│   ├── agents/
│   │   ├── code-reviewer.md           # Reviews code (read-only)
│   │   ├── explorer.md                # Searches code (read-only)
│   │   └── memory-keeper.md           # Writes memory (read-write)
│   ├── commands/
│   │   ├── learn.md                   # /learn command
│   │   └── review-and-fix.md          # /review-and-fix command
│   └── settings.json                  # Permissions
│
└── memory/
    ├── INDEX.md                       # Master table of contents
    ├── lessons.md                     # Lessons (empty initially)
    ├── decisions.md                   # Decisions (empty initially)
    └── conventions.md                 # Conventions (empty initially)
```

---

## After Setup: What Happens in Each Session

### Session startup:
```
✅ CLAUDE.md loaded
✅ memory/INDEX.md loaded (tells Claude where to look)
✅ memory/conventions.md loaded (code style rules, no need to re-read)
```

### Standard workflow (what the user should do):
```
1. User: "Add a validateEmail function"
2. Claude: [writes code, applies conventions automatically]
3. User: (optional) /review-and-fix
   → if the change is significant or you're unsure
   → code-reviewer checks for bugs + convention violations
   → shows findings
4. User: "Apply 1, 3" (or "all" or "none")
5. Claude: [applies approved fixes]
6. If systemic issue found:
   User: /learn "rule: always X because Y"
   → memory-keeper records it
   → future sessions know this rule
```

---

## How to Customize for Your Project

### Step 1: Edit CLAUDE.md

Replace the top section with your project description:

**Before:**
```markdown
# AI_learning — Claude Code learning sandbox

This is a sandbox project for learning how to structure...
```

**After (example for a web app):**
```markdown
# MyApp — Production web application

This is a React + Node.js web application. Key tools: Jest, ESLint, Prettier.
```

Keep everything else in CLAUDE.md unchanged (memory reading rule, subagents, commands, workflow are generic).

### Step 2: Clear the Memory Files

Each memory file (lessons.md, decisions.md, conventions.md) has a template with a TABLE OF CONTENTS and an empty `## Entries` section. Keep the template, delete any example entries, and start fresh via `/learn`.

### Step 3: Add First Conventions (Optional)

If your project already has code style rules, capture them now:

```
/learn "always use camelCase for variables. Apply consistently."
/learn "no console.log in production code — use logger.debug() instead"
/learn "max line length: 100 characters"
```

This seeds the system so future sessions apply them automatically.

---

## Testing the Setup

After Claude creates all files, run this test:

```
/learn "test: the architecture guide should be updated when the project structure changes"
```

You should see:
1. Claude delegates to memory-keeper
2. memory-keeper writes an entry to one of the memory files
3. memory-keeper updates the TABLE OF CONTENTS
4. You get back: "Saved to memory/X.md → ## <title> (line N)"

If you see this output, the system is working. You can delete the test entry afterward.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "404 on memory/INDEX.md" | Make sure INDEX.md exists (Claude should create it). If not, create it manually. |
| "/review-and-fix doesn't run" | Make sure `.claude/commands/review-and-fix.md` exists and has the correct description. |
| "Claude doesn't know conventions" | CLAUDE.md must have `@memory/conventions.md` import. If missing, add it. |
| "Can't find code to review" | code-reviewer uses `git diff` if it's a git repo, otherwise reads recently modified files. Make sure you've edited/created files. |

---

## What This Gives You

✅ **Automatic convention enforcement** — conventions applied from the start, no manual style checking  
✅ **On-demand code review** — when you need it most (significant changes, uncertain code)  
✅ **Self-improving system** — Claude learns from reviews and applies lessons automatically  
✅ **Efficient memory** — conventions loaded once at startup, lessons indexed by line number  
✅ **Reproducible workflow** — /learn and /review-and-fix are standardized across sessions  

---

## Next Steps (After Setup)

1. **Use the standard workflow** on your first real task (not a test)
2. **Capture lessons** as they come up (`/learn "rule: ..."`), don't do it manually
3. **Review the GUIDE.md** (if created) to understand the system deeply
4. **Iterate** — the more lessons you capture, the smarter Claude becomes

---

## Tips for Success

- **Don't skip reviews** — `/review-and-fix` is mandatory, not optional. It's where the learning loop closes.
- **Keep memory files under 500 lines each** — if one grows too big, ask memory-keeper to suggest a split.
- **Be specific in /learn** — "never mock the database" is better than "write good tests"
- **Use the reading rule** — don't read full memory files; use the table of contents and jump to sections.
- **Commit memory/ to git** — project memory is shared; personal auto-memory (`~/.claude/projects/<name>/memory/`) is not.

---

## One More Thing

If you're setting this up for a team, share this guide with them. The memory system is most powerful when multiple people contribute lessons. 

Happy building! 🚀
