# Rules in Claude Code — What They Are and How to Use Them

A plain-English guide to understanding rules, how they relate to CLAUDE.md, and when to use them.

---

## What Are Rules?

"Rules" are not a separate feature — they are just **CLAUDE.md instructions organized into multiple files** inside a `.claude/rules/` directory.

Think of it this way:

| Approach | What it is |
|---|---|
| Single `CLAUDE.md` | All instructions in one file |
| `.claude/rules/` folder | Same instructions, split into topic-focused files |

Both do the exact same thing. "Rules" is just what you call instructions when they live in that folder.

---

## How Rules Work

Create a `.claude/rules/` directory and add markdown files inside it. Claude reads all of them automatically — same as CLAUDE.md.

```
.claude/
  rules/
    code-style.md     ← formatting and naming conventions
    testing.md        ← how tests should be written
    api.md            ← API route conventions
    git.md            ← commit message rules, branch naming
```

Each file is plain markdown. No special syntax needed — just write instructions the same way you would in CLAUDE.md.

---

## Path-Scoped Rules

One thing rules files can do that a plain CLAUDE.md cannot: **apply only to specific file paths** using frontmatter.

```markdown
---
paths:
  - "src/api/**/*.ts"
---

## API Rules
- Always validate input with Zod
- Return errors in { error: string } shape
```

Claude only loads this file when working on files that match the path pattern. This keeps context focused and avoids loading irrelevant instructions.

---

## Project Rules vs Global Rules

| | Project rules | Global rules |
|---|---|---|
| **Location** | `.claude/rules/` (project root) | `~/.claude/rules/` (home dir) |
| **Scope** | This project only | Every project on your machine |
| **Shared with team?** | Yes — committed to git | No — personal only |

Use **global rules** for your personal preferences (response style, language choice, habits).
Use **project rules** for team-shared conventions that apply to this codebase.

---

## Do You Actually Need Rules?

Not necessarily. Here's a simple decision guide:

**Stick with CLAUDE.md if:**
- Your instructions fit in one readable file (~100 lines or less)
- You're working alone or on a small project
- You don't need path-specific loading

**Move to `.claude/rules/` if:**
- Your CLAUDE.md is growing too long to maintain
- Different areas (auth, DB, UI, testing) have unrelated conventions
- Multiple teammates are editing the instructions file
- You want some rules to only load for certain file paths

---

## CLAUDE.md vs Rules — Summary

| | `CLAUDE.md` | `.claude/rules/` |
|---|---|---|
| **Format** | Single markdown file | Multiple markdown files |
| **Path scoping** | No | Yes (via frontmatter) |
| **Best for** | Simple projects | Larger or team projects |
| **Same behavior?** | Yes | Yes |

They are two ways to organize the same thing. Start with `CLAUDE.md`. Split into rules when it gets unwieldy.

---

## Quick Example

### Before (one big CLAUDE.md)
```markdown
## Code style
- Use 2-space indent
- Prefer const over let

## Testing
- Always write unit tests for services
- Mock external APIs

## API conventions
- Validate input with Zod
- Use 400 for validation errors
```

### After (split into rules)
```
.claude/rules/
  code-style.md    ← indent, naming, formatting
  testing.md       ← test patterns and mocking rules
  api.md           ← validation, error shapes
```

Each file is shorter, focused, and easier to update. Claude loads all of them — same result, better organization.

---

## Do Rules Need to Be Imported in CLAUDE.md?

No. Rules in `.claude/rules/` are **loaded automatically** — no `@` import needed in CLAUDE.md.

Claude scans the `.claude/rules/` directory on its own. If a rules file has `paths:` frontmatter, it loads conditionally. If it has no frontmatter, it loads always — just like CLAUDE.md.

| File | Needs manual import? |
|---|---|
| `.claude/rules/*.md` (no frontmatter) | No — always auto-loaded |
| `.claude/rules/*.md` (with `paths:`) | No — auto-loaded when path matches |
| `docs/claude/something.md` (outside `.claude/rules/`) | Yes — needs `@` in CLAUDE.md |

Only use `@` when the file lives **outside** the `.claude/rules/` folder.
