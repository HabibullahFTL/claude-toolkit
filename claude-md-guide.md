# CLAUDE.md — Setup, Hierarchy & Best Practices

A concise reference for understanding CLAUDE.md, the `/init` command, file hierarchy, and how it compares to AGENTS.md.

---

## What is `/init`?

`/init` is a Claude Code slash command that **auto-generates a `CLAUDE.md` file** for your project.

- Scans your codebase (structure, stack, scripts, conventions)
- Writes a starter `CLAUDE.md` at the project root
- You review and refine it — Claude doesn't update it automatically afterward

```bash
# Run inside your project directory
/init
```

Use it once at the start of a project, then maintain the file manually.

---

## What is CLAUDE.md?

`CLAUDE.md` is a **persistent instruction file** that Claude Code reads automatically at the start of every session. Think of it as a project briefing — it tells Claude:

- What the project is and what stack it uses
- Coding conventions and style rules
- Commands to run (build, test, lint, etc.)
- Things to avoid or always do
- Key architectural decisions

Claude reads it silently — you don't need to paste it into chat.

---

## File Hierarchy

Claude merges CLAUDE.md files from multiple locations, reading all of them together:

| Location                   | Scope                             | Example path                   |
| -------------------------- | --------------------------------- | ------------------------------ |
| `~/.claude/CLAUDE.md`      | Global — applies to every project | `/Users/you/.claude/CLAUDE.md` |
| `<project-root>/CLAUDE.md` | Project-wide                      | `/my-app/CLAUDE.md`            |
| `<subdir>/CLAUDE.md`       | Subdirectory only                 | `/my-app/backend/CLAUDE.md`    |

**Priority:** More specific (deeper) files take precedence when instructions conflict.

### How to use each level

- **Global** — personal preferences, preferred languages, response style, tools you always use
- **Project root** — stack details, commands, conventions for the whole repo
- **Subdirectory** — module-specific rules (e.g. different lint rules for `frontend/` vs `backend/`)

---

## How It Works

1. You open a project in Claude Code
2. Claude automatically reads all CLAUDE.md files in the hierarchy
3. Those instructions shape every response in the session — no need to repeat yourself
4. When you run `/init`, Claude generates the project-level file based on what it finds in the codebase

---

## CLAUDE.md vs AGENTS.md

|                         | `CLAUDE.md`                              | `AGENTS.md`                               |
| ----------------------- | ---------------------------------------- | ----------------------------------------- |
| **For whom**            | Claude Code specifically                 | Any AI agent (OpenAI Codex, Cursor, etc.) |
| **Auto-read by Claude** | Yes, always                              | No — Claude ignores it                    |
| **Standard**            | Anthropic                                | OpenAI / community                        |
| **Purpose**             | Claude-specific instructions and context | Generic agent instructions                |
| **Use when**            | Your team uses Claude Code               | Your team uses multiple AI tools          |

> If your team uses multiple AI tools, you may want **both files** — `AGENTS.md` for shared conventions and `CLAUDE.md` for Claude-specific overrides.

---

## What to Put Where

### Global `~/.claude/CLAUDE.md`

```markdown
- Always respond concisely
- Prefer TypeScript over JavaScript
- Never add emojis unless asked
- My preferred package manager is pnpm
```

### Project `CLAUDE.md`

```markdown
## Stack

Next.js 14 (App Router), Prisma, PostgreSQL, Tailwind CSS

## Commands

- `pnpm dev` — start dev server
- `pnpm test` — run tests
- `pnpm lint` — ESLint check

## Conventions

- Use server components by default; client components only when needed
- API routes go in `src/app/api/`
- Never commit `.env.local`
```

### Subdirectory CLAUDE.md (e.g. `backend/CLAUDE.md`)

```markdown
## Backend-specific rules

- All DB queries must go through the service layer, not directly in route handlers
- Use Zod for all input validation
```

---

## Splitting a Large CLAUDE.md

As a project grows, `CLAUDE.md` can get long and hard to maintain. Claude Code lets you split it into focused files and import them with `@`.

### How it works

Use `@filename` in `CLAUDE.md` to import additional context files — Claude loads them automatically. Any line starting with `@` that points to a file will cause Claude to load that file too:

```markdown
# CLAUDE.md

@docs/claude/auth.md      # authentication conventions
@docs/claude/database.md  # ORM patterns, migration rules
@docs/claude/api.md       # route conventions, error handling
@docs/claude/ui.md        # component patterns, styling rules
```

Claude loads `CLAUDE.md` first, then follows each `@` import and reads those files as well. The imported files are full markdown — same format, same rules.

### Suggested structure

Keep `CLAUDE.md` as a short index; split conventions by area into `docs/claude/`:

```
CLAUDE.md                     ← short index: stack, key conventions, imports
docs/
  claude/
    auth.md                   ← auth conventions, session handling, providers
    database.md               ← ORM patterns, migration rules, query conventions
    api.md                    ← API route conventions, error handling, validation
    ui.md                     ← component patterns, styling rules, design tokens
```

`CLAUDE.md` stays short and readable. Each imported file covers one area in depth.

### When to split

Split `CLAUDE.md` when:

- It exceeds ~100 lines and keeps growing
- Different areas (auth, DB, UI) have unrelated conventions that don't need to be loaded together
- Multiple people are editing it and stepping on each other

Don't split prematurely — a single well-organized `CLAUDE.md` is fine for most projects.

### Conditional loading with `@`

You can also use `@` inline when prompting Claude directly, not just in `CLAUDE.md`:

```
@docs/claude/database.md — add a new migration for the payments table
```

This loads the database conventions only for that prompt, without making them a permanent import in `CLAUDE.md`. Useful for context that's only relevant to a specific task.

---

## Maintenance Tips

- **Keep it short** — Claude reads it every session; bloated files dilute signal
- **Update it when conventions change** — stale instructions cause confusion
- **Don't put secrets in it** — it may be committed to version control
- **Don't duplicate what's in code** — don't describe architecture that's obvious from the file structure; focus on non-obvious decisions
- **Re-run `/init` on a new branch** only if the project has changed significantly; otherwise edit manually
