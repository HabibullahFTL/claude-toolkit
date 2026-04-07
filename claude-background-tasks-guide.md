# Background Tasks in Claude Code — When, Why, and How

---

## What Is a Background Task?

When you ask Claude to do something, normally it works on it and you wait — the conversation is blocked until it's done.

A **background task** lets Claude start a job and keep it running separately, so you (or Claude) can continue doing other things at the same time.

Think of it like this: instead of waiting for the oven timer, you start the oven and go do something else. The oven runs in the background.

---

## Why Use Background Tasks?

| Problem | Background task solves it |
|---|---|
| A task takes a long time (big test run, long build) | Start it in background, keep working |
| You want Claude to do two independent things at once | Run them in parallel as separate agents |
| Verbose output (logs, test results) clutters your chat | Isolate it to a background task |
| You want Claude to research multiple things simultaneously | Fan out to multiple background agents |

---

## How to Use Background Tasks

### Option 1 — Ask Claude directly

Just tell Claude to run something in the background:

```
Run the full test suite in the background while we continue working on the new feature.
```

Claude starts the task, keeps it running separately, and you can keep chatting.

### Option 2 — Press `Ctrl+B`

If a task is already running and taking too long, press **Ctrl+B** to push it to the background.

> Tmux users: press **Ctrl+B twice**.

---

## Running Claude Non-Interactively (Programmatic Mode)

You can run Claude Code without an interactive session using the `-p` flag. Useful in scripts, CI/CD pipelines, or automation:

```bash
# Run a prompt and print the result
claude -p "Review this PR and summarize the changes"

# Get output as JSON
claude -p "Find all TODO comments in src/" --output-format json

# Streaming JSON output
claude -p "Run the tests and report failures" --output-format stream-json --verbose

# Skip local config, hooks, and skills (clean run for CI)
claude -p "Run lint check" --bare
```

---

## Running Multiple Things in Parallel

Claude can spin up multiple subagents to work on independent tasks at the same time:

```
Research how to implement rate limiting, and at the same time look up 
how our current auth middleware works. Do both in parallel.
```

Claude runs two agents simultaneously and brings both results back to you — faster than doing them one after another.

**When to use parallel agents:**
- Two independent research tasks
- Running tests on separate modules
- Comparing two different approaches
- Any work where the tasks don't depend on each other

---

## Checking on Background Task Results

Background task output is written to files. When the task finishes, Claude reads the results and reports back to you. You don't have to do anything manually — Claude tracks it.

---

## When to Use Background Tasks

**Use background tasks when:**
- The task takes more than a few seconds and you want to keep working
- You have two or more independent tasks that can run at the same time
- The task produces a lot of output that would flood your chat (logs, test runs)
- You're scripting or automating Claude in CI/CD (use `-p` flag)

**Don't bother with background tasks when:**
- The task is quick (a few seconds)
- The next step depends on the result — just wait for it
- You're doing one simple thing

---

## Disable Background Tasks

If you don't want Claude to run anything in the background at all:

```bash
CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1 claude
```

---

## Quick Reference

| What you want | How |
|---|---|
| Run a task in background | Tell Claude: "run this in the background" |
| Push running task to background | Press `Ctrl+B` |
| Run Claude non-interactively | `claude -p "your prompt"` |
| Get structured output from a script | `claude -p "prompt" --output-format json` |
| Run multiple tasks in parallel | Ask Claude to do both "in parallel" |
| Skip local config in CI | Add `--bare` to the `-p` command |
