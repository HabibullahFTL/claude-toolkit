# Claude Code Toolkit

A collection of skills, agents, and reference guides that extend Claude Code with reusable, prompt-driven workflows.

---

## Table of Contents

- [What's in this toolkit](#whats-in-this-toolkit)
- [Skills](#skills)
  - [Available Skills](#available-skills)
  - [How to work with Claude on your projects](#how-to-work-with-claude-on-your-projects)
  - [Directory structure](#directory-structure)
  - [Option 1 — Global skills (all projects)](#option-1--global-skills-all-projects)
    - [Clone the entire repo](#clone-the-entire-repo)
    - [Add a single skill folder](#add-a-single-skill-folder)
    - [Keep skills up to date](#keep-skills-up-to-date)
  - [Option 2 — Project-level skills (one project)](#option-2--project-level-skills-one-project)
  - [Verify installation](#verify-installation)
  - [Writing your own skill](#writing-your-own-skill)
- [Agents](#agents)
- [Reference Guides](#reference-guides)

---

## What's in this toolkit

| Path | What it is |
| ---- | ---------- |
| `.claude/skills/` | Slash-command skills — installable from this repo (see [Skills](#skills)) |
| `.claude/agents/` | Custom sub-agent definitions (e.g. `fullstack-expert`) |
| `working-with-claude-code.md` | Full project lifecycle walkthrough |
| `claude-*.md` / `sandbox-guide.md` | Reference guides for specific Claude Code features |

---

## Skills

Skills are slash commands (`/skill-name`) that extend Claude Code with reusable, prompt-driven workflows. Each skill lives in its own folder containing a `SKILL.md` file.

### Available Skills

| Skill | Command | Description |
| ----- | ------- | ----------- |
| `git-initial-setup` | `/git-initial-setup [commit-all]` | Initialize git and create a comprehensive `.gitignore` for Next.js, React Native, Expo, Flutter, Node, Kotlin, and more |
| `git-commit` | `/git-commit [add-scope] [no-description]` | Stage all changes and write a Conventional Commits message with a clear title and bullet description |
| `design-doc` | `/design-doc <file>` | Read a file, conduct a focused technical interview, and produce a structured design document (`.spec.md`, `.plan.md`, `.design.md`) |
| `frontend-design` | `/frontend-design` | Create distinctive, production-grade frontend interfaces — auto-detects framework, styling method, component library, TypeScript, and dark mode setup |
| `handoff` | `/handoff [--update-claude] [--dated-folder] [--feedback]` | Generate a structured session handoff document capturing completed work, dead ends, key decisions, and next steps |
| `setup-next-convex-clerk` | `/setup-next-convex-clerk` | Scan a Next.js codebase, ask focused questions, then install packages and scaffold the full Convex + Clerk boilerplate — including middleware, schema, hooks, providers, utils, and CLAUDE.md rules |

### How to work with Claude on your projects

For a step-by-step walkthrough of the full development lifecycle (new projects, existing codebases, planning, building, committing), see [`working-with-claude-code.md`](./working-with-claude-code.md).

---

### Directory structure

```
~/.claude/skills/          ← global skills (available in every project)
  git-commit/
    SKILL.md
  git-initial-setup/
    SKILL.md
  design-doc/
    SKILL.md
  frontend-design/
    SKILL.md
  handoff/
    SKILL.md
  setup-next-convex-clerk/
    SKILL.md
    ref/                   ← boilerplate templates used by the skill

your-project/.claude/skills/   ← project skills (available in that project only)
  my-skill/
    SKILL.md
```

---

### Option 1 — Global skills (all projects)

Skills placed in `~/.claude/skills/` are available everywhere.

#### Clone the entire repo

Skills live inside `.claude/skills/` in this repo, so clone to a temp location and copy the skill folders into place:

```bash
git clone https://github.com/HabibullahFTL/claude-toolkit /tmp/claude-toolkit
mkdir -p ~/.claude/skills
cp -r /tmp/claude-toolkit/.claude/skills/. ~/.claude/skills/
```

#### Add a single skill folder

```bash
# Clone with sparse checkout to fetch just one skill
git clone --filter=blob:none --sparse https://github.com/HabibullahFTL/claude-toolkit /tmp/claude-toolkit
cd /tmp/claude-toolkit
git sparse-checkout set .claude/skills/git-commit
cp -r .claude/skills/git-commit ~/.claude/skills/
```

#### Keep skills up to date

Pull the latest changes and re-copy the skill folders:

```bash
cd /tmp/claude-toolkit && git pull
cp -r .claude/skills/. ~/.claude/skills/
```

---

### Option 2 — Project-level skills (one project)

Skills placed in `.claude/skills/` inside a project are only available when Claude Code is running in that project.

```bash
# From your project root
mkdir -p .claude/skills

# Copy a skill folder from a local clone or the global directory
cp -r ~/.claude/skills/git-commit .claude/skills/
```

Or clone the toolkit repo and copy what you need:

```bash
git clone https://github.com/HabibullahFTL/claude-toolkit /tmp/claude-toolkit
cp -r /tmp/claude-toolkit/.claude/skills/git-commit your-project/.claude/skills/
```

---

### Verify installation

Open Claude Code in the target directory and type `/` — your installed skills will appear in the autocomplete list.

Or ask Claude directly:

```
What skills do you have available?
```

---

### Writing your own skill

Create a new folder under the appropriate `skills/` directory and add a `SKILL.md` file:

```
~/.claude/skills/my-skill/SKILL.md
```

Minimal `SKILL.md` format:

```markdown
---
name: my-skill
description: One-line description shown in the skill list.
---

Instructions for Claude to follow when /my-skill is invoked.
```

Optional frontmatter fields:

| Field                      | Purpose                                                 |
| -------------------------- | ------------------------------------------------------- |
| `argument-hint`            | Hint shown after the skill name in autocomplete         |
| `disable-model-invocation` | Set `true` to skip the LLM and run the prompt literally |

Restart Claude Code after adding or editing a skill for changes to take effect.

---

## Agents

Agent definitions live in `.claude/agents/`. Each file is a markdown file with a YAML frontmatter block that defines the agent's name, description, and available tools.

**Included agents:**

| Agent | File | Description |
| ----- | ---- | ----------- |
| `fullstack-expert` | `.claude/agents/fullstack-expert.md` | Full-stack expert for Next.js (App Router), Convex, and Clerk projects — handles building, fixing, debugging, explaining, and refactoring across the entire stack |

The `fullstack-expert` agent has comprehensive built-in knowledge covering:

| Area | What it covers |
| ---- | -------------- |
| Convex functions | Middleware wrappers, response helpers, Zod validation, pagination, HTTP actions |
| Clerk integration | Middleware, providers, server and client auth helpers |
| Next.js conventions | App Router, Server Components, data fetching, `preloadQuery` patterns |
| Custom hooks | `useConvexQuery`, `useConvexMutation`, `useConvexAction`, `useConvexPaginatedQuery` |
| Component structure | Page-level layout, reusable components, naming conventions |
| Forms | React Hook Form + Zod + ShadCN full integration |
| Boilerplate | Full project bootstrap order and all utility file implementations |

> To scaffold the Convex + Clerk boilerplate in a new or existing Next.js project, use the [`/setup-next-convex-clerk`](./.claude/skills/setup-next-convex-clerk/SKILL.md) skill first, then use `fullstack-expert` for ongoing feature development.

---

## Reference Guides

Standalone guides covering specific Claude Code features and workflow tools:

### Workflow & Best Practices

| File | Topic |
| ---- | ----- |
| [`working-with-claude-code.md`](./working-with-claude-code.md) | Full project lifecycle walkthrough — scaffold → plan → build → commit → handoff |
| [`claude-code-best-practices.md`](./claude-code-best-practices.md) | Practical habits for faster, less frustrating workflows |

### Project Memory & Context

| File | Topic |
| ---- | ----- |
| [`claude-md-guide.md`](./claude-md-guide.md) | CLAUDE.md — setup, hierarchy, splitting, and best practices |
| [`claude-rules-guide.md`](./claude-rules-guide.md) | Rules — path-scoped instructions in `.claude/rules/` |

### Agents & Automation

| File | Topic |
| ---- | ----- |
| [`claude-agents-and-subagents.md`](./claude-agents-and-subagents.md) | Agents and sub-agents — built-in types, custom agents, tool restrictions, parallel execution |
| [`claude-task-list-guide.md`](./claude-task-list-guide.md) | Task list — tracking multi-step work with live progress in a session |
| [`claude-handoff-guide.md`](./claude-handoff-guide.md) | /handoff — creating session summaries for continuity across sessions |
| [`claude-background-tasks-guide.md`](./claude-background-tasks-guide.md) | Background tasks — parallel execution, non-interactive mode, CI/CD usage |
| [`claude-batch-and-simplify-guide.md`](./claude-batch-and-simplify-guide.md) | /batch and /simplify — running prompts without interaction and cleaning up code |

### Integrations & External Tools

| File | Topic |
| ---- | ----- |
| [`claude-mcp-guide.md`](./claude-mcp-guide.md) | MCP — connecting Claude to databases, GitHub, Slack, and other external services |
| [`claude-connectors-guide.md`](./claude-connectors-guide.md) | Connectors — pre-built integrations for scheduled tasks (web only) |

### Context & Security

| File | Topic |
| ---- | ----- |
| [`claude-add-dir-guide.md`](./claude-add-dir-guide.md) | /add-dir — adding directories (with glob filtering) to Claude's context |
| [`sandbox-guide.md`](./sandbox-guide.md) | Sandbox — OS-level security boundary that restricts Claude's file and network access |
