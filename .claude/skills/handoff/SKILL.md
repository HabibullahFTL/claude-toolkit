---
name: handoff
description: A slash command for Claude Code that generates a session handoff document when ending work. Use when the user wants to create a summary of work completed, document key decisions made during a session, capture pending issues and next steps, prepare context for future sessions or other developers, or update CLAUDE.md with learnings. Triggers on "/handoff" or phrases like "write a handoff", "create a session summary", "document what we did", "end session with notes".
argument-hint: '[--update-claude-md] [--dated-folder] [--feedback]'
---

Generate a handoff document summarizing the current session's work, decisions, and pending items.

## Parse Arguments

Check `$ARGUMENTS` for these flags:

- `--update-claude`: Append key decisions/learnings to CLAUDE.md
- `--dated-folder`: Save to `docs/handoffs/{YYYY-MM-DD}/` instead of root
- `--feedback`: Collect session rating and improvement notes using the `AskUserQuestion` tool before generating

## Gather Information

1. Review conversation history for:
   - Completed tasks and implementations
   - Key decisions and their rationale
   - Failed approaches (to avoid repeating)
   - External context (deadlines, stakeholder requirements)

2. Check git status if available:

   ```bash
   git status --short
   git diff --stat
   ```

3. Look for TODOs/FIXMEs mentioned in session

## Output File

Before writing the file, run this command to get the current local date and time:

```bash
date +"%Y-%m-%d %H%M"
```

Use the output to construct the filename and ISO timestamp. Never guess or hardcode the time.

**Default location**: `handoffs/HANDOFF-{YYYY-MM-DD-HHMM}.md`
**With `--dated-folder`**: `handoffs/{YYYY-MM-DD}/HANDOFF-{HHMM}.md`

## Document Template

```markdown
---
title: Session Handoff
date: { ISO timestamp from `date -Iseconds` }
session_id: { conversation ID if available }
project: { project name from package.json, Cargo.toml, or directory name }
---

# Session Handoff — {Date}

## Summary

{2-3 sentences: what's different now vs. session start}

## Work Completed

{Grouped by category if multiple areas. Specific file:function references.}

## Dead Ends — Do Not Retry

{Approaches that were tried and failed. Critical for avoiding wasted effort in future sessions.}

### {Problem/Goal attempted}

| Approach Tried       | Why It Failed                                                | Time Spent       |
| -------------------- | ------------------------------------------------------------ | ---------------- |
| {What was attempted} | {Specific reason: error, incompatibility, performance, etc.} | {Rough estimate} |

**What to try instead**: {If known, suggest alternative direction}

### {Another problem if applicable}

...

## Key Decisions

{Technical choices with rationale. Format: **Decision**: Rationale. Trade-off: X.}

## Pending Issues

- [ ] {Issue} — {severity: HIGH/MED/LOW} {optional workaround note}

## Next Steps

1. {Actionable item with file:line reference if applicable}

## Files Modified

{Significant changes only. Skip formatting-only edits.}

## Context for Future Sessions

{Non-obvious context: env quirks, stakeholder requirements}

## Session Feedback

- **Rating**: {answer from AskUserQuestion 1}
- **What worked well**: {answer from AskUserQuestion 2}
- **What to improve**: {answer from AskUserQuestion 3}
```

## Dead Ends Section Guidelines

This section is critical for preventing context rot across sessions. Be specific:

**Bad (too vague):**

> - Tried using library X, didn't work

**Good (actionable):**

> ### Fixing the race condition in SessionStore
>
> | Approach Tried        | Why It Failed                                               |
> | --------------------- | ----------------------------------------------------------- |
> | `async-mutex` package | Deadlock when nested calls to `getSession()`                |
> | Redis WATCH/MULTI     | Our Redis 6.x cluster doesn't support WATCH in cluster mode |
> | In-memory lock Map    | Works single-node but breaks in horizontal scaling          |
>
> **What to try instead**: Upgrade to Redis 7.x which supports WATCH in cluster mode, or use Redlock algorithm

**Capture these dead ends:**

- Packages/libraries that had incompatibilities
- Approaches that caused new bugs or regressions
- Solutions that worked locally but failed in CI/staging/prod
- Configurations that conflicted with existing setup
- Rabbit holes that consumed significant time without progress

## Example Output

```markdown
---
title: Session Handoff
date: 2025-01-14T14:30:00Z
session_id: conv_abc123
project: myapp-api
---

# Session Handoff — 2025-01-14

## Summary

Implemented user authentication flow and fixed critical session race condition. Auth endpoints functional but missing rate limiting.

## Work Completed

**Authentication**

- Added `/auth/login` and `/auth/logout` endpoints (`src/routes/auth.ts`)
- Created `AuthService` class with JWT generation (`src/services/AuthService.ts`)
- Unit tests at 85% coverage (`tests/auth.test.ts`)

**Bug Fixes**

- Fixed race condition in `SessionStore.getSession():42` causing duplicate sessions

## Dead Ends — Do Not Retry

### Fixing the SessionStore race condition

| Approach Tried            | Why It Failed                                                             | Time Spent |
| ------------------------- | ------------------------------------------------------------------------- | ---------- |
| `async-mutex` package     | Deadlock on nested `getSession()` calls from middleware chain             | ~45 min    |
| Redis WATCH/MULTI         | Cluster mode (Redis 6.x) doesn't support WATCH — throws `CROSSSLOT` error | ~30 min    |
| Simple in-memory Map lock | Works locally but race condition returns when running multiple pods       | ~20 min    |

**What to try instead**: Either upgrade to Redis 7.x (supports WATCH in cluster) or implement Redlock. Current fix is a workaround using optimistic locking with retry.

### JWT token validation

| Approach Tried             | Why It Failed                                                   | Time Spent |
| -------------------------- | --------------------------------------------------------------- | ---------- |
| `jsonwebtoken` sync verify | Blocks event loop on high traffic — p99 latency spiked to 800ms | ~15 min    |

**What to try instead**: Already switched to `jose` library with async verify — this is resolved.

## Key Decisions

- **JWT over session cookies**: Stateless for horizontal scaling. Trade-off: no server-side invalidation without blacklist.
- **bcrypt cost factor 12**: ~300ms hash time balances security vs. latency.
- **Refresh token rotation**: Limits exposure window of compromised tokens.

## Pending Issues

- [ ] Rate limiting not implemented — HIGH — vulnerable to brute force
- [ ] Password reset flow incomplete — MED — email service not configured
- [ ] `validateToken()` edge case near midnight UTC — LOW

## Next Steps

1. Add rate limiting to `/auth/login` (suggest: express-rate-limit, 5/min) — `src/routes/auth.ts:15`
2. Configure SendGrid for password reset — needs `SENDGRID_API_KEY` in env
3. Integration tests for token refresh flow

## Files Modified

- `src/services/AuthService.ts` — created
- `src/routes/auth.ts` — created
- `src/stores/SessionStore.ts:42` — fixed race condition
- `tests/auth.test.ts` — created

## Context for Future Sessions

- Client expects OAuth by sprint end — may need `AuthService` refactor
- Staging uses different JWT secret (check `.env.staging`)
- SessionStore fix is a workaround; proper fix needs Redis 7.x upgrade
```

## Behavior Details

### Information Gathering

1. Review conversation history for completed tasks, decisions, failed approaches
2. Run `git status` and `git diff --stat` if available
3. Check for TODO/FIXME comments mentioned in session
4. Note external context (deadlines, stakeholder requirements, constraints)
5. **Actively scan for failed attempts** — look for patterns like "that didn't work", "let's try something else", reverted changes, error messages that led to pivots

### `--update-claude` Behavior

Appends to CLAUDE.md (creates if missing):

```markdown
## Session Learnings — {date}

- {Key decision or pattern that should persist}
```

Also consider adding critical dead ends to CLAUDE.md if they're likely to be re-attempted:

```markdown
## Known Dead Ends

- Don't use `async-mutex` with nested SessionStore calls — causes deadlock
- Redis WATCH doesn't work in cluster mode on 6.x
```

### `--feedback` Behavior

When `--feedback` is present, use the `AskUserQuestion` tool **three times sequentially** before generating the document — one question per call so each answer is captured cleanly.

> **Constraint**: `AskUserQuestion` accepts a maximum of **4 options** per question. Never pass 5 or more — it will throw an `InputValidationError`.

Use exactly these calls:

**Call 1 — Rating**
```
question: "How would you rate this session?"
header: "Rating"
options (4 max):
  - "5 — Excellent"  → Everything went smoothly, very productive
  - "4 — Good"       → Mostly smooth with minor friction
  - "3 — Average"    → Got things done but had some issues
  - "1–2 — Poor"     → Significant friction or confusion
```

**Call 2 — What worked well**
```
question: "What worked well this session?"
header: "Went well"
options (4 max, 3 dynamic + 1 fixed):
  - Derive 3 options from the session — pick moments that went smoothly,
    e.g. a task completed in one shot, a decision made quickly, a fix that
    worked first try. Be specific to what actually happened, not generic.
    Example: "add-dir guide created cleanly" or "AskUserQuestion fix landed first try"
  - Always include as the last option:
    "Other" → Something else — describe below
```

**Call 3 — What to improve**
```
question: "What could be improved for next time?"
header: "Improve"
options (4 max, 3 dynamic + 1 fixed):
  - Derive 3 options from the session — pick friction points, retries,
    misunderstandings, or approaches that needed correction. Be specific
    to what actually happened, not generic.
    Example: "wrong file edited first (SKILL.md vs new guide)" or
    "5-option AskUserQuestion caused a retry"
  - Always include as the last option:
    "Other" → Something else — describe below
```

Store the three answers and write them into the **Session Feedback** section of the handoff document. Ask one at a time — wait for each response before asking the next.

### Post-Generation Output

After creating file, output:

```
✓ Created handoffs/HANDOFF-2025-01-14-1430.md

To continue next session:
  "Load handoffs/HANDOFF-2025-01-14-1430.md and continue from next steps"

⚠️  {N} dead ends documented — avoid re-attempting these approaches

{if --update-claude}
✓ Updated CLAUDE.md with session learnings
{/if}
```

## Integration Notes

- Handoff files are designed to be loaded at session start for continuity. Write in a way a new version of you would understand.
- YAML frontmatter enables scripted indexing/search across handoffs
- Severity levels (HIGH/MED/LOW) help prioritize when resuming
- **Dead Ends section is the highest-value part for long debugging sessions** — prioritize capturing these thoroughly
