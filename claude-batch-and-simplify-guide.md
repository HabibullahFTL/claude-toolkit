# `/batch` and `/simplify` in Claude Code — A Plain-English Guide

A short, practical guide to two useful Claude Code features: running tasks without back-and-forth, and cleaning up your code automatically.

---

## `/batch` — Run Multiple Prompts at Once

### What is it?

By default, Claude Code is interactive — you type a prompt, Claude responds, you type again. `/batch` (and batch mode in general) lets you **queue multiple tasks and run them without stopping for your input each time**.

Think of it like a to-do list you hand Claude all at once, instead of giving it one task at a time.

---

### Why Use It?

Without batch mode:
- You send one prompt → wait → respond → send another → wait again
- Slow, especially for repetitive or predictable tasks

With batch mode:
- You define everything upfront → Claude works through it all → you review the results

Best for tasks like: running tests, adding types to multiple files, generating boilerplate, reformatting code across a folder.

---

### How to Use It

**Option 1 — Non-interactive CLI flag (`-p`)**

Run Claude Code with the `-p` flag (print mode) to send a single prompt without entering an interactive session:

```bash
claude -p "Add JSDoc comments to all functions in src/utils/math.ts"
```

Claude runs, does the task, prints the result, and exits. No back-and-forth.

**Option 2 — Pipe multiple prompts**

Chain multiple prompts using shell scripting:

```bash
echo "Refactor getUserById to use async/await in src/services/user.ts" | claude -p
echo "Add error handling to createUser in src/services/user.ts" | claude -p
```

Or pipe from a file containing your prompts:

```bash
claude -p < tasks.txt
```

**Option 3 — Use `--output-format` for scripting**

When running Claude in batch from a script and you want structured output:

```bash
claude -p "List all TODO comments in src/" --output-format json
```

---

### Real-World Example

Say you want Claude to document three different files without babysitting it:

```bash
# Run each as a background batch task
claude -p "Add JSDoc to all exported functions in src/lib/auth.ts"
claude -p "Add JSDoc to all exported functions in src/lib/db.ts"
claude -p "Add JSDoc to all exported functions in src/lib/email.ts"
```

You kick these off, come back, and review. No waiting in between.

---

### When to Use Batch Mode

| Use batch when... | Skip it when... |
|---|---|
| The task is repetitive across many files | You need to review output before the next step |
| You know exactly what you want | The task is exploratory or unclear |
| You're scripting Claude into a workflow | You need Claude to ask clarifying questions |
| You don't need to approve each change | The task involves risky or irreversible changes |

---

## `/simplify` — Clean Up Code After You Write It

### What is it?

`/simplify` is a skill that **reviews your recently changed code and fixes anything that's unnecessarily complex, duplicated, or inefficient** — without changing what the code does.

It's like a second pass: you write the feature, then `/simplify` tightens it up.

---

### Why Use It?

When you're building fast, code can get messy:
- A function that's longer than it needs to be
- Logic that's duplicated in two places
- A variable that's never used
- A loop that could be a one-liner

You could clean this manually — or let `/simplify` do it.

---

### How to Use It

Run it after you've made changes:

```
/simplify
```

Claude will:
1. Look at the code you just changed
2. Find anything redundant, over-engineered, or duplicated
3. Fix it — cleaner, shorter, same behavior

That's it. No arguments needed.

---

### Real-World Example

You just built a function to filter active users:

```ts
// What you wrote
function getActiveUsers(users: User[]) {
  const result = [];
  for (let i = 0; i < users.length; i++) {
    if (users[i].isActive === true) {
      result.push(users[i]);
    }
  }
  return result;
}
```

After running `/simplify`:

```ts
// What Claude produces
function getActiveUsers(users: User[]) {
  return users.filter(u => u.isActive);
}
```

Same behavior. Half the lines.

---

### Another Example

You wrote a utility with a duplicated condition:

```ts
function formatName(user: User) {
  if (user.firstName && user.lastName) {
    return `${user.firstName} ${user.lastName}`;
  }
  if (user.firstName && !user.lastName) {
    return user.firstName;
  }
  if (!user.firstName && user.lastName) {
    return user.lastName;
  }
  return 'Unknown';
}
```

After `/simplify`:

```ts
function formatName(user: User) {
  return [user.firstName, user.lastName].filter(Boolean).join(' ') || 'Unknown';
}
```

---

### When to Use `/simplify`

| Use `/simplify` when... | Skip it when... |
|---|---|
| You just finished implementing something | Code hasn't changed yet |
| You wrote something in a hurry | The "complexity" is intentional (e.g. performance-critical code) |
| You want a second opinion on your code | You're in the middle of a task — finish first, simplify after |
| Code review is coming up | |

---

## Quick Comparison

| | `/batch` | `/simplify` |
|---|---|---|
| **What it does** | Runs multiple tasks without interaction | Cleans up changed code |
| **When to use** | Repetitive, scripted tasks | After writing new code |
| **How to trigger** | `claude -p "..."` or pipe | `/simplify` |
| **Requires review?** | Yes — check diffs after | Yes — always review AI changes |
| **Changes behavior?** | No — runs your prompts as-is | No — same logic, cleaner code |
