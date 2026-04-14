# `/add-dir` in Claude Code — A Plain-English Guide

A short, practical guide to adding directories (with optional file filtering) to your Claude Code session context.

---

## What is `/add-dir`?

By default, Claude Code only reads files within your current working directory. `/add-dir` lets you **add another directory to Claude's context** so it can read, understand, and work with files from that location too.

You can also pass a glob pattern to tell Claude to only load files matching a specific extension or name format — keeping the context focused and relevant.

---

## Why Use It?

Without `/add-dir`:
- Claude only sees the current project folder
- Files in sibling folders, shared libs, or docs directories are invisible to it

With `/add-dir`:
- Claude can read files from any path you point it to
- You can narrow it down to just the file types you care about (e.g. only `*.md` files)

---

## How to Use It

**Basic syntax:**

```
/add-dir <path>
```

**With a glob filter:**

```
/add-dir <path> <glob-pattern>
```

---

## Examples

**Add an entire directory:**

```
/add-dir ./docs
```

Claude can now read everything inside the `docs/` folder.

---

**Add only Markdown files from a directory:**

```
/add-dir ./docs *.md
```

Claude loads only files matching `*.md` (e.g. `README.md`, `CHANGELOG.md`, `API.md`). All other file types in that folder are ignored.

---

**Add only TypeScript files from a shared library:**

```
/add-dir ../shared-lib *.ts
```

Useful in a monorepo where your shared utilities live outside the current project folder.

---

**Add only config files:**

```
/add-dir ./config *.json
```

Loads just the JSON configs from the `config/` directory — nothing else.

---

## When to Use `/add-dir`

| Use `/add-dir` when... | Skip it when... |
|---|---|
| You need Claude to read files outside the current folder | All relevant files are already in the working directory |
| You want to load only specific file types (e.g. `*.md`) | You want Claude to see everything — just use the default context |
| You're in a monorepo and need a sibling package | The file you need is a one-off — just reference it directly |
| You're working with a large folder but only care about docs | Adding everything would flood the context with irrelevant files |

---

## Quick Tip

Combining `/add-dir` with a glob pattern is the best practice for large directories. Instead of loading hundreds of files, you give Claude exactly what it needs — faster, cheaper, and more focused.

```
# Instead of this (loads everything):
/add-dir ./my-big-project

# Do this (loads only what matters):
/add-dir ./my-big-project *.md
```
