# /handoff — Session Handoff Guide

A plain-English guide to the `/handoff` skill: what it does, when to use it, and how the flags work.

---

## What Is `/handoff`?

`/handoff` generates a structured markdown document summarizing everything that happened in the current Claude Code session — what was built, what failed, key decisions made, and what comes next.

Think of it as a baton pass: the document is written so that a future session (or a teammate) can pick up exactly where you left off without needing to re-read the conversation.

---

## When to Use It

Run `/handoff` at the end of any session where:

- You completed non-trivial work you'll want to continue later
- You hit dead ends others (or future-you) might retry
- You made architectural or technical decisions worth recording
- You're handing work to another developer
- A session ran long and you want a clean entry point for next time

---

## Basic Usage

```
/handoff
```

Creates a file at `handoffs/HANDOFF-{YYYY-MM-DD-HHMM}.md` with a full session summary.

---

## Flags

### `--update-claude`

Appends key learnings and known dead ends to `CLAUDE.md` so they persist into every future session — not just the handoff document.

```
/handoff --update-claude
```

Use this when a decision or dead end is important enough that Claude should know about it automatically, without you needing to load the handoff file.

**What gets added to CLAUDE.md:**

```markdown
## Session Learnings — {date}

- {Key pattern or decision}

## Known Dead Ends

- Don't use X for Y — causes Z
```

---

### `--dated-folder`

Saves the handoff into a date-organized subfolder instead of the flat `handoffs/` root.

```
/handoff --dated-folder
```

| Without flag | With flag |
|---|---|
| `handoffs/HANDOFF-2025-01-14-1430.md` | `handoffs/2025-01-14/HANDOFF-1430.md` |

Use this when you run multiple handoffs per day and want them grouped by date.

---

### `--feedback`

Asks you three questions before generating the document and records your answers in a **Session Feedback** section.

```
/handoff --feedback
```

The three questions:
1. **Rating** — How would you rate this session? (1–5 scale)
2. **What worked well** — Options are derived from actual smooth moments in the session
3. **What to improve** — Options are derived from actual friction points in the session

Options are always specific to what happened — not generic labels. This makes the feedback useful for spotting patterns across sessions.

---

## What the Document Contains

| Section | What it captures |
|---|---|
| **Summary** | 2–3 sentences: what changed vs. session start |
| **Work Completed** | Specific tasks, grouped by area, with `file:line` references |
| **Dead Ends** | Approaches tried and why they failed — with a table per problem |
| **Key Decisions** | Technical choices and their rationale |
| **Pending Issues** | Open items with HIGH/MED/LOW severity |
| **Next Steps** | Ordered, actionable items |
| **Files Modified** | Significant changes only — formatting edits skipped |
| **Context for Future Sessions** | Non-obvious env quirks, stakeholder constraints |
| **Session Feedback** | Present only when `--feedback` flag is used |

---

## The Dead Ends Section

This is the highest-value part of the document for debugging-heavy sessions.

**Bad (too vague):**
> Tried library X, didn't work

**Good (actionable):**
> | Approach Tried | Why It Failed | Time Spent |
> |---|---|---|
> | `async-mutex` | Deadlock on nested `getSession()` calls | ~45 min |
> | Redis WATCH/MULTI | WATCH not supported in Redis 6.x cluster mode | ~30 min |
>
> **What to try instead**: Upgrade to Redis 7.x or use Redlock algorithm

The goal is to prevent the next session from burning time on paths that already failed.

---

## How to Resume from a Handoff

At the start of your next session, load the file:

```
Load handoffs/HANDOFF-2025-01-14-1430.md and continue from next steps
```

Claude will read the document and pick up where you left off — pending issues, next steps, and dead ends included.

---

## Combining Flags

Flags can be combined freely:

```
/handoff --update-claude --feedback
/handoff --dated-folder --update-claude
/handoff --dated-folder --update-claude --feedback
```

---

## Output After Generation

```
✓ Created handoffs/HANDOFF-2025-01-14-1430.md

To continue next session:
  "Load handoffs/HANDOFF-2025-01-14-1430.md and continue from next steps"

⚠️  3 dead ends documented — avoid re-attempting these approaches

✓ Updated CLAUDE.md with session learnings   ← only with --update-claude
```

---

## Quick Decision Guide

| Situation | Command |
|---|---|
| End of a normal session | `/handoff` |
| Decision worth persisting to all future sessions | `/handoff --update-claude` |
| Multiple handoffs per day, want them grouped | `/handoff --dated-folder` |
| Want to record session quality | `/handoff --feedback` |
| Long debugging session with many dead ends | `/handoff --update-claude` |
| Handing off to a teammate | `/handoff --dated-folder --update-claude` |
