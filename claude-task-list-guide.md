# Claude Task List — What It Is, When, Why, and How to Use It

---

## What Is the Task List?

The **task list** is Claude Code's built-in system for breaking work into discrete, trackable steps within a conversation.

When Claude creates tasks, you get a live view of what it's doing, what's done, and what's coming next — instead of watching a wall of tool calls go by with no sense of progress.

Think of it like a shared to-do list between you and Claude. Claude writes the plan, you can watch it execute, and both of you always know where things stand.

---

## Why Use the Task List?

| Without task list | With task list |
|---|---|
| Claude works quietly — you don't know how far it's gotten | You see each step and its status in real time |
| You have to re-read the conversation to understand what was done | Tasks give a clean summary of the work structure |
| Large tasks feel opaque and hard to trust | You can verify each step before Claude moves on |
| Hard to resume work if context is lost | Tasks carry intent forward even across a compacted context |

The core benefit is **visibility and control** — especially on large, multi-step tasks where it's easy to lose track of what's in progress versus what's done.

---

## When to Use the Task List

**Good fits:**
- Multi-step implementation tasks (e.g., "Add auth to all API routes")
- Tasks with clear phases (plan → implement → test → verify)
- Anything you'd naturally write a checklist for before starting
- Work you want to be able to pause and resume
- Tasks where you want to verify each step before proceeding

**Not worth it:**
- Quick, single-step operations ("rename this variable")
- One-line answers or explanations
- Tasks where the steps aren't known upfront and would just be invented noise

---

## How the Task List Works

### Claude creates it automatically

On complex tasks, Claude creates tasks itself using the `TaskCreate` tool. You don't need to prompt it. If Claude doesn't create tasks and you want it to, just ask:

```
Break this into tasks so I can track progress.
```

### Tasks have statuses

Each task goes through these states:

| Status | Meaning |
|---|---|
| `pending` | Not started yet |
| `in_progress` | Claude is actively working on this |
| `completed` | Done |
| `cancelled` | Skipped or no longer needed |

Only one task is `in_progress` at a time. Claude marks a task completed before moving to the next.

### Tasks appear in the UI

As Claude works, you'll see the task list update in real time in the Claude Code interface — each step checks off as it's finished.

---

## The Task Tools (What Claude Uses Internally)

You don't call these yourself — Claude calls them. But knowing they exist helps you understand what's happening.

| Tool | What it does |
|---|---|
| `TaskCreate` | Creates a new task with a title, description, and initial status |
| `TaskUpdate` | Updates a task's status (e.g., marks it `completed`) |
| `TaskGet` | Reads a specific task's current state |
| `TaskList` | Lists all tasks in the current session |
| `TaskOutput` | Attaches output/results to a task for reference |
| `TaskStop` | Cancels or stops a running task |

---

## Examples

### Example 1 — Adding a feature

You ask:

```
Add email verification to the signup flow. 
Users should receive a link, click it, and their account gets activated.
```

Claude might create tasks like:

```
[ ] 1. Audit the current signup flow and database schema
[ ] 2. Add email_verified and verification_token columns to users table
[ ] 3. Create the send-verification-email utility
[ ] 4. Add POST /verify-email endpoint
[ ] 5. Integrate email send into the signup handler
[ ] 6. Write tests for the verification flow
[ ] 7. Verify end-to-end by running the test suite
```

As Claude works through them, you see each one flip from `pending` → `in_progress` → `completed`. You always know which step it's on.

---

### Example 2 — A multi-file refactor

You ask:

```
Refactor all database calls in the services layer to use the new query builder.
```

Claude creates:

```
[ ] 1. Identify all files in services/ that make raw DB calls
[ ] 2. Understand the new query builder API
[ ] 3. Refactor services/user.ts
[ ] 4. Refactor services/orders.ts
[ ] 5. Refactor services/payments.ts
[ ] 6. Run the test suite to verify no regressions
```

Without tasks, this looks like Claude reading and editing files for 10 minutes with no signal. With tasks, you can see exactly which file is being worked on at any moment.

---

### Example 3 — Asking Claude to be explicit

If Claude starts working on something large without creating tasks, you can always say:

```
Before you start — break this down into a task list so I can follow along.
```

Claude will pause, plan the steps, create the tasks, then start executing.

---

### Example 4 — Pausing between steps

If you want to review each step before Claude proceeds, ask upfront:

```
Add OAuth login. Create a task list and check in with me after each task 
completes before moving to the next.
```

Claude will complete a task, show you what it did, and wait for your go-ahead before continuing.

---

## Task List vs. Other Claude Features

| Feature | Purpose | Scope |
|---|---|---|
| **Task list** | Track progress on multi-step work in the current session | Current conversation only |
| **CLAUDE.md** | Store permanent project instructions and conventions | Persists across sessions |
| **Memory** | Store facts about the user, project, or workflow | Persists across sessions |
| **Sub-agents** | Delegate focused work to isolated specialist agents | Isolated context window |

The task list is a **session-scoped progress tracker**. It doesn't persist when you start a new conversation, and it's not a substitute for a real project management tool. Its job is to make the current session's work transparent and verifiable.

---

## Tips

**Ask for a plan first.** On big tasks, say "plan this out before doing anything." Claude creates the task list but doesn't start executing — you can review and adjust the plan before any changes are made.

**Use tasks to catch drift.** If Claude seems to be going off track, look at the task list. The current `in_progress` task tells you exactly what Claude thinks it's doing. If that doesn't match your intent, correct it early.

**Tasks keep context when things compress.** As conversations get long, Claude's context gets compressed. Tasks help preserve the structure of the work — what was done, what's next — so nothing gets lost.

**Don't ask for tasks on tiny things.** A task list for "fix this typo" is overhead with no benefit. Reserve it for work with genuine structure and multiple meaningful steps.

---

## Quick Reference

| You want to... | How |
|---|---|
| See tasks Claude created | They appear automatically in the UI as Claude works |
| Ask Claude to plan before starting | "Break this into tasks first, don't start yet" |
| Step-by-step execution with review | "Check in with me after each task before continuing" |
| Get a summary of what was done | "List what tasks you completed and what each one did" |
| Resume from where you left off | Reference the task list: "Continue from task 3 onward" |
