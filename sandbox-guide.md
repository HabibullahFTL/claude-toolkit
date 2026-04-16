# Claude Code Sandbox — A Short Guide

## What is Sandbox?

Sandbox is an **OS-level security boundary** that restricts what Claude Code can do when it runs shell commands on your machine.

When sandbox is **on**, Claude's subprocesses are locked down:
- Cannot write to most parts of the filesystem
- Cannot make network requests
- Cannot spawn arbitrary processes

When sandbox is **off** (the default), Claude can do everything your user account can do.

---

## Why Use It?

| Reason | Explanation |
|---|---|
| **Safety** | Prevents accidental deletion, overwriting, or corruption of files outside the project |
| **Security** | Stops malicious code in dependencies or LLM responses from touching your system |
| **Auditing** | Forces explicit permission grants — you know exactly what Claude is allowed to touch |
| **CI/CD** | Adds a hard boundary in automated pipelines where human review is absent |

---

## How to Use It

### 1. Start Claude with sandbox enabled

```bash
claude --sandbox
```

That's it. All Bash commands Claude runs will be sandboxed for that session.

### 2. Disable sandbox for a single command (escape hatch)

Inside a session, the `dangerouslyDisableSandbox` flag on the Bash tool lets Claude bypass the sandbox for one specific command when it truly needs to (e.g. installing a package, running a migration).

Claude will ask for your approval before doing this — you'll see it flagged in the tool call.

### 3. Set it permanently via settings

Add to your `settings.json` (global or project-level):

```json
{
  "sandbox": true
}
```

Run `/update-config` in Claude Code to configure this interactively.

---

## When to Use It

**Use sandbox when:**
- Exploring an unfamiliar or third-party codebase
- Running Claude in a CI/CD pipeline
- You want read-only analysis (code review, debugging, explanation)
- Working with untrusted input (user-supplied code, external repos)
- You want peace of mind during a long autonomous session

**Skip sandbox when:**
- You need Claude to install packages, run migrations, or write build artifacts
- You are actively developing and need full tool access
- You have already reviewed and trust the task scope

---

## Quick Example

```bash
# Safe exploration of an unfamiliar repo
claude --sandbox "explain the architecture of this codebase"

# Safe CI audit — no writes allowed
claude --sandbox "review this PR for security issues"

# Normal dev work — sandbox off (default)
claude "add a login endpoint and write the tests"
```

---

## Summary

```
sandbox on  → read-heavy, exploratory, CI, untrusted code
sandbox off → active development, installs, migrations (default)
```

Think of sandbox like a **read-only mode for your filesystem** — it lets Claude think and explain freely, but keeps your machine safe from unintended writes.
