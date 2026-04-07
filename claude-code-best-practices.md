# Claude Code — Best Practices

Simple, practical habits that make working with Claude Code faster and less frustrating.

---

## 1. Start Every Project with `/init`

Run `/init` once when you open a new project. Claude scans the codebase and generates a `CLAUDE.md` with the stack, commands, and conventions. Edit it to match reality, then leave it.

---

## 2. Keep CLAUDE.md Short and Honest

Claude reads CLAUDE.md every session. A bloated file dilutes the signal.

**Write in it:**
- Stack and versions
- Build/test/lint commands
- Non-obvious conventions and decisions

**Don't write in it:**
- Things obvious from the file structure
- Secrets or API keys
- Architecture that's already clear from the code

---

## 3. Use `.claude/rules/` for Specific Areas

When CLAUDE.md gets long, split it. Put area-specific rules in `.claude/rules/` — Claude loads them automatically, no import needed.

Use `paths:` frontmatter to load rules only when working in a specific area:

```markdown
---
paths:
  - "src/api/**/*.ts"
---
- Validate all input with Zod
- Return errors as { error: string }
```

This keeps Claude's context lean and relevant.

---

## 4. Be Specific in Your Prompts

Vague prompts get vague results. Tell Claude exactly what you want.

| Instead of | Say |
|---|---|
| "fix the bug" | "the login form throws on empty email — fix the validation in `src/components/LoginForm.tsx`" |
| "add a feature" | "add a delete button to the user card in `src/components/UserCard.tsx` that calls `DELETE /api/users/:id`" |
| "clean this up" | "refactor `getUserById` in `src/services/user.ts` to use async/await instead of .then()" |

---

## 5. Ask Claude to Plan Before It Codes

For anything non-trivial, ask Claude to explain its approach first before writing code. This catches wrong assumptions early.

```
Before writing any code, explain how you'd approach adding Stripe webhooks to this project.
```

Review the plan, correct it if needed, then say "go ahead."

---

## 6. Load Context with `@` for One-Off Tasks

You don't have to put everything in CLAUDE.md. For a task that needs specific context just once, load it inline:

```
@docs/claude/database.md — add a migration for a new `payments` table
```

The file loads only for that prompt. Nothing permanent changes.

---

## 7. Break Big Tasks Into Small Steps

Don't ask Claude to "build the whole auth system." Ask one step at a time:

1. "Create the user model with Prisma"
2. "Add a registration endpoint"
3. "Add login with JWT"
4. "Add a middleware to protect routes"

Smaller steps = easier to review, easier to fix if something goes wrong.

---

## 8. Always Review Before Accepting

Claude can be confidently wrong. Before you accept any change:

- Read the diff
- Check that it doesn't touch things you didn't ask about
- Run the tests or the app

Don't let Claude commit or push unless you've reviewed.

---

## 9. Commit Often So You Can Revert

Before asking Claude to make a large change, commit your current work. If Claude's output is wrong or breaks something, you can roll back cleanly.

```bash
git add -p && git commit -m "checkpoint before refactor"
```

---

## 10. Use `/clear` When Switching Tasks

Claude carries conversation context. When you switch to a different task or file area, run `/clear` to reset. A fresh context avoids Claude confusing the old task with the new one.

---

## 11. Use Global Rules for Personal Preferences

Put things that are *your* preferences (not the project's) in `~/.claude/rules/`:

```markdown
- Always respond concisely
- Prefer TypeScript over JavaScript
- Never add emojis unless I ask
- My preferred package manager is pnpm
```

These apply to every project without polluting the project's CLAUDE.md.

---

## 12. Don't Over-Prompt

You don't need to explain every detail. Claude understands codebases well. Trust it to read the files and figure out the context — point it at the right file and describe what you want changed.

Over-explaining wastes your time and adds noise to the conversation.

---

## Quick Reference

| Habit | Why |
|---|---|
| Run `/init` on new projects | Auto-generates CLAUDE.md |
| Keep CLAUDE.md short | Bloated files reduce quality |
| Use `.claude/rules/` with `paths:` | Context loads only when relevant |
| Be specific in prompts | Precise input = precise output |
| Ask for a plan first | Catch wrong assumptions early |
| Break tasks into steps | Easier to review and correct |
| Commit before big changes | Easy rollback if things go wrong |
| Use `/clear` between tasks | Prevents context bleed |
