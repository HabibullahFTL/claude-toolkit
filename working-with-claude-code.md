# Working with Claude Code — Step-by-Step Workflow Guide

A practical guide for building apps (Next.js, React, React Native, etc.) using Claude Code with custom skills.

---

## [Phase 1 — Starting a Brand New Project](#phase-1--starting-a-brand-new-project)

<details>
<summary><h4>Step 1 — Scaffold the project</h4><p>Use your normal tools outside Claude Code, e.g. <code>npx create-next-app@latest</code></p></summary>

```bash
npx create-next-app@latest my-app
# or
npx react-native init MyApp
# or
npx expo create my-app
```

Then open the project folder in Claude Code.

</details>

---

<details>
<summary><h4>Step 2 — Initialize git and <code>.gitignore</code> in just one command</h4><p>Run <code>/git-initial-setup commit-all</code> — detects your project type, creates <code>.gitignore</code>, and commits everything in one shot</p></summary>

**Why:** This detects your project type (Next.js, Expo, React Native, Flutter, etc.), creates a proper `.gitignore`, and commits everything in one shot.

```
/git-initial-setup commit-all
```

Use `commit-all` for the very first commit — it stages all your project files, not just `.gitignore`.

Without `commit-all`, it only commits `.gitignore`.

</details>

---

<details>
<summary><h4>Step 3 — Use <code>/init</code> to give Claude the full picture</h4><p>Use <code>#</code> to tell Claude your conventions first; run <code>/init</code> only after implementing a few real features</p></summary>

Don't run `/init` immediately after scaffolding. At that point your project is 100% boilerplate — Claude already knows the default commands for Next.js, Expo, React Native, etc. Running `/init` this early produces a [`CLAUDE.md`](#splitting-a-large-claudemd) that just echoes things Claude already knows.

Instead, as you start building, tell Claude your conventions using the `#` prefix:

```
# We use server actions, not API routes
# Folder structure: features go in /src/features/<name>/
# We use Zustand for client state
```

Each `#` line is saved to Claude's memory immediately and applied in every future session.

**When to run `/init`:** After you've built 1–2 real features. By then your project has actual structure, patterns, and conventions worth scanning — and [`CLAUDE.md`](#splitting-a-large-claudemd) becomes genuinely useful. Alternatively, write [`CLAUDE.md`](#splitting-a-large-claudemd) by hand — you know your own project better than Claude can infer from scanning.

Read more: [Splitting a Large CLAUDE.md](#splitting-a-large-claudemd)

</details>

---

## [Phase 1b — Joining an Existing Codebase](#phase-1b--joining-an-existing-codebase)

> Skip to Phase 2 if you just finished Phase 1. This phase is only for when you're opening a project that already has real code. Clone the repo, open the folder in Claude Code, then start with Step B.

<details>
<summary><h4>Step B — Run <code>/init</code> immediately</h4><p>Real code already exists — run <code>/init</code> right away so Claude scans the actual structure and writes <code>CLAUDE.md</code></p></summary>

**Why:** Unlike a brand new project, an existing codebase already has real structure, conventions, and patterns worth scanning. `/init` has something meaningful to discover here — run it right away.

```
/init
```

Claude will explore the codebase and write a `CLAUDE.md` at the project root.

Read more: [Splitting a Large CLAUDE.md](#splitting-a-large-claudemd)

</details>

---

<details>
<summary><h4>Step C — Review and refine <code>CLAUDE.md</code></h4><p>Read the generated file, correct anything wrong, and add what Claude couldn't infer (conventions, decisions, gotchas)</p></summary>

Claude infers what it can from code, but it cannot know:

- Why certain decisions were made
- Unwritten team conventions
- Known bugs or fragile areas to avoid
- Deployment or environment quirks

Edit `CLAUDE.md` directly to add or correct anything Claude missed.

</details>

---

<details>
<summary><h4>Step D — Fill remaining gaps with <code>#</code> memory</h4><p>Anything too personal or session-specific for <code>CLAUDE.md</code> goes into <code>#</code> memory instead</p></summary>

```
# Always run the dev server on port 4000 in this project
# The /legacy folder is frozen — never touch it
```

`CLAUDE.md` is shared with the team (committed to git). `#` memory is yours alone.

</details>

---

## [Phase 2 — Planning a Feature](#phase-2--planning-a-feature)

Always plan before asking Claude to code anything non-trivial. This prevents Claude from going in the wrong direction and wasting your time.

<details>
<summary><h4>Step 4 — Write a rough idea file</h4><p>Create a plain <code>.md</code> file in <code>plans/</code> describing what you want to build — no need for detail, just your thoughts</p></summary>

```
plans/user-auth.md
```

```
I want users to be able to sign up and log in using email + password.
Use NextAuth. Store users in PostgreSQL via Prisma.
```

</details>

---

<details>
<summary><h4>Step 5 — Generate a design document</h4><p>Run <code>/design-doc plans/&lt;feature&gt;.md</code> — Claude interviews you and writes a proper design doc</p></summary>

**Why:** Claude reads your rough idea, asks you focused questions about things it can't infer (edge cases, tradeoffs, requirements), and produces a proper design document.

```
/design-doc plans/user-auth.md
```

Claude interviews you one question at a time, then writes a structured `.spec.md`, `.plan.md`, or `.design.md` depending on the content.

**Use this for:**

- New features with real complexity
- Anything that touches multiple files or systems
- Features where requirements aren't obvious

**Skip this for:**

- Tiny changes (fix a bug, tweak a style, add a field)
- Things you already know exactly how to build

</details>

---

<details>
<summary><h4>Step 6 — Review the design doc before coding</h4><p>Fix anything wrong, then reference it with <code>@</code> when prompting Claude</p></summary>

Read the generated document. If something is wrong, edit it manually or re-run `/design-doc` on it to iterate.

The design doc becomes your source of truth. Reference it when prompting Claude:

```
@plans/user-auth.plan.md — implement this
```

</details>

---

## [Phase 3 — Building the Feature](#phase-3--building-the-feature)

<details>
<summary><h4>Step 7 — Use Plan Mode for large changes</h4><p>Run <code>/plan</code> so Claude lays out its approach before touching any files</p></summary>

For anything that touches many files, use plan mode before letting Claude write code.

```
/plan
```

Or just say: `"Think through this before writing any code"` — Claude will lay out its approach and wait for your approval.

**Use plan mode when:**

- Implementing a full feature from scratch
- Refactoring across multiple files
- You're unsure how Claude will approach something

**Skip plan mode when:**

- Fixing a specific bug
- Adding a small component
- Making an isolated change

</details>

---

<details>
<summary><h4>Step 8 — Prompt Claude to implement</h4><p>Reference the design doc with <code>@</code> and implement in small slices, not all at once</p></summary>

Reference your design doc so Claude has full context:

```
@plans/user-auth.plan.md — implement this step by step, starting with the Prisma schema
```

Break large features into pieces. Don't ask Claude to build everything at once.

```
Good:
"Implement the Prisma schema for user auth from @plans/user-auth.plan.md"
"Now implement the NextAuth config"
"Now implement the sign-up API route"

Too broad:
"Implement the entire authentication system"
```

</details>

---

<details>
<summary><h4>Step 9 — Use auto-accept for trusted, low-risk tasks</h4><p>Press <code>shift+tab</code> to skip permission prompts on safe, repetitive work</p></summary>

**Use:** `shift+tab` to toggle auto-accept mode

When you trust Claude on a task (e.g. generating boilerplate, installing packages, writing tests), toggle auto-accept so it doesn't stop to ask permission for every file write or bash command.

Turn it off again when Claude is doing something risky or unfamiliar.

</details>

---

<details>
<summary><h4>Step 10 — Reference files directly when asking questions</h4><p>Use <code>@src/file.ts</code> to point Claude at the exact file instead of describing it</p></summary>

Use `@` to point Claude at a specific file:

```
@src/lib/auth.ts — why is the session token not being persisted?
```

This is faster and more precise than describing the file in words.

</details>

---

## [Phase 4 — Committing Your Work](#phase-4--committing-your-work)

<details>
<summary><h4>Step 11 — Commit after each logical unit of work</h4><p>Run <code>/git-commit</code> — stages everything and writes a clean Conventional Commits message for you</p></summary>

**Why:** This stages all changes, reads the diff, and writes a proper Conventional Commits message with a clear title and bullet-point description of what changed and why.

```
/git-commit
```

**Optional flags:**

| Flag             | When to use                                                |
| ---------------- | ---------------------------------------------------------- |
| `add-scope`      | When you want scope in the message, e.g. `feat(auth): ...` |
| `no-description` | For tiny commits where a body is unnecessary               |

```
/git-commit add-scope
/git-commit no-description
```

**Commit often** — after each feature slice, bug fix, or refactor. Don't let changes pile up.

</details>

---

## [Phase 5 — Ongoing Development](#phase-5--ongoing-development)

<details>
<summary><h4>Step 12 — Save important decisions to memory</h4><p>Prefix with <code>#</code> to persist conventions across sessions, e.g. <code># use server actions not API routes</code></p></summary>

Use `#` to tell Claude to remember something across sessions:

```
# Always use server actions, not API routes, in this project
# We use Zustand for client state, not Context API
```

This gets saved to Claude's memory and applied automatically in future sessions — so you don't have to repeat yourself.

</details>

---

<details>
<summary><h4>Step 13 — Clear context when switching tasks</h4><p>Run <code>/clear</code> before starting something unrelated so Claude doesn't carry over wrong assumptions</p></summary>

When you finish one feature and start something unrelated, clear the context. This prevents Claude from carrying over assumptions from the previous task.

```
/clear
```

</details>

---

<details>
<summary><h4>Step 14 — Update <code>CLAUDE.md</code> when conventions change</h4><p>Edit the file manually when your stack or patterns shift; no need to re-run <code>/init</code></p></summary>

If your team adopts a new pattern, switches a tool, or adds a major convention, add it to `CLAUDE.md` manually:

```
- Use server actions for all form submissions (not API routes)
- All database calls go through /src/lib/db, never imported directly
```

You don't need to re-run `/init` — just edit the file directly.

</details>

---

## [Splitting a Large CLAUDE.md](#splitting-a-large-claudemd)

As a project grows, `CLAUDE.md` can get long and hard to maintain. Claude Code lets you split it into focused files and import them with `@`.

<details>
<summary><h4>How it works</h4><p>Use <code>@filename</code> in <code>CLAUDE.md</code> to import additional context files — Claude loads them automatically</p></summary>

In `CLAUDE.md`, any line starting with `@` that points to a file will cause Claude to load that file too:

```markdown
# CLAUDE.md

# for authentication

@docs/claude/auth.md

# for database

@docs/claude/database.md

# for api

@docs/claude/api.md
```

Claude loads `CLAUDE.md` first, then follows each `@` import and reads those files as well. The imported files are full markdown — same format, same rules.

</details>

---

<details>
<summary><h4>Suggested structure</h4><p>Keep <code>CLAUDE.md</code> as a short index; split conventions by area into <code>docs/claude/</code></p></summary>

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

</details>

---

<details>
<summary><h4>When to split</h4><p>Split when <code>CLAUDE.md</code> exceeds ~100 lines or different areas have unrelated conventions</p></summary>

Split `CLAUDE.md` when:

- It exceeds ~100 lines and keeps growing
- Different areas (auth, DB, UI) have unrelated conventions that don't need to be loaded together
- Multiple people are editing it and stepping on each other

Don't split prematurely — a single well-organized `CLAUDE.md` is fine for most projects.

</details>

---

<details>
<summary><h4>Conditional loading with <code>@</code></h4><p>Use <code>@</code> inline in a prompt to load context only for that task, without adding a permanent import to <code>CLAUDE.md</code></p></summary>

You can also use `@` inline when prompting Claude directly, not just in `CLAUDE.md`:

```
@docs/claude/database.md — add a new migration for the payments table
```

This loads the database conventions only for that prompt, without making them a permanent import in `CLAUDE.md`. Useful for context that's only relevant to a specific task.

</details>

---

## [Full New Project Checklist](#full-new-project-checklist)

```
[ ] Scaffold project (create-next-app, expo, etc.)
[ ] /git-initial-setup commit-all
[ ] Use # to tell Claude your conventions (run /init after a few real features)
[ ] Write rough idea in plans/<feature-name>.md
[ ] /design-doc plans/<feature-name>.md  →  review output
[ ] /plan  (for large features)
[ ] Implement in slices, referencing @design doc
[ ] /git-commit  after each slice
[ ] Repeat from "Write rough idea" for next feature
[ ] After 1-2 real features: /init  or  write CLAUDE.md by hand
```

## Joining an Existing Project Checklist

```
[ ] Clone repo and open in Claude Code
[ ] /init  →  Claude scans the codebase and writes CLAUDE.md
[ ] Review CLAUDE.md  →  fix errors, add conventions Claude couldn't infer
[ ] Use # to save personal/session-specific notes that don't belong in CLAUDE.md
[ ] Continue from Phase 2 for any feature work
```

---

## Toolkit Reference

**Your custom skills**

| Command                         | When to use                                                              |
| ------------------------------- | ------------------------------------------------------------------------ |
| `/git-initial-setup`            | Once at project start — init git, create `.gitignore`, commit            |
| `/git-initial-setup commit-all` | Same as above but also stages and commits all project files              |
| `/git-commit`                   | After finishing a unit of work — stages everything and writes the commit |
| `/git-commit add-scope`         | Same, but adds a scope to the commit e.g. `feat(auth): ...`              |
| `/git-commit no-description`    | Same, but title-only — no bullet description in the commit body          |
| `/design-doc <file>`            | Before coding a feature — turns a rough idea into a full design doc      |

**Built-in Claude Code**

| Command           | When to use                                                                   |
| ----------------- | ----------------------------------------------------------------------------- |
| `/init`           | After 1–2 real features exist — scans codebase and writes `CLAUDE.md`         |
| `/plan`           | Before coding anything large — Claude plans first, you approve, then it codes |
| `/clear`          | When switching to a completely different task                                 |
| `shift+tab`       | When doing safe, repetitive work — skips permission prompts                   |
| `# <instruction>` | To save a convention to Claude's memory so it applies every session           |
| `@filename`       | To reference a specific file directly in your prompt                          |
