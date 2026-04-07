# Claude Agents & Sub-Agents: A Complete Guide

## Table of Contents

1. [What is an Agent?](#1-what-is-an-agent)
2. [What are Sub-Agents?](#2-what-are-sub-agents)
3. [How Do They Work Under the Hood?](#3-how-do-they-work-under-the-hood)
4. [Built-in Sub-Agent Types](#4-built-in-sub-agent-types)
5. [When & How Sub-Agents Are Called](#5-when--how-sub-agents-are-called)
6. [Do Sub-Agents Run Automatically?](#6-do-sub-agents-run-automatically)
7. [Custom Sub-Agents: What, Why & When](#7-custom-sub-agents-what-why--when)
8. [Creating a Custom Sub-Agent](#8-creating-a-custom-sub-agent)
9. [Tool Access & Restrictions](#9-tool-access--restrictions)
10. [Context Window Management](#10-context-window-management)
11. [Parallel & Background Execution](#11-parallel--background-execution)
12. [Worktree Isolation](#12-worktree-isolation)
13. [Sub-Agents vs Agent Teams](#13-sub-agents-vs-agent-teams)
14. [Pros & Cons](#14-pros--cons)
15. [Must-Know Things](#15-must-know-things)

---

## 1. What is an Agent?

An **agent** is a language model (like Claude) combined with tools that allow it to take autonomous actions — reading files, running commands, editing code, browsing the web, etc.

Claude Code itself IS the main agent. When you open a terminal and start using Claude Code, you are interacting with the main agent running an **agentic loop**:

```
1. Gather context   →  Read files, search code, understand what's needed
2. Take action      →  Edit files, run commands, call tools
3. Verify results   →  Run tests, check outputs, course-correct
         ↑____________________ repeat until done _____________________|
```

This loop runs autonomously until the task is complete or it needs your input.

---

## 2. What are Sub-Agents?

Sub-agents are **specialized helpers that the main agent spawns to handle specific tasks** — they run in their own separate context window, work independently, and report results back to the main agent.

Think of it like a manager (main agent) and employees (sub-agents):

```
You ──asks──> Main Agent (Manager)
                  │
                  ├──spawns──> Sub-Agent A: "Research the auth module"
                  ├──spawns──> Sub-Agent B: "Plan the refactor strategy"
                  └──spawns──> Sub-Agent C: "Review code for security issues"
                                    │
                  <──results────────┘ (summary returned to main agent)
```

**Key differences from the main agent:**

| | Main Agent | Sub-Agent |
|---|---|---|
| Context | Shared with entire conversation | Isolated — fresh start |
| Lifespan | Entire session | One task, then done |
| Communication | Direct with user | Reports back to main agent only |
| Memory of conversation | Full history | None (only the spawn prompt) |
| Can spawn other agents | Yes | No (no nesting) |

---

## 3. How Do They Work Under the Hood?

When the main agent decides to use a sub-agent, it calls the **Agent tool** with:
- A **task description** (what to do)
- The **sub-agent type** or name (which agent to use)

The sub-agent then:

1. **Loads a fresh context** — CLAUDE.md, MCP servers, skills, but NOT the main conversation history
2. **Reads the spawn prompt** — the specific task it was given
3. **Runs its own agentic loop** — gathers context, takes actions, verifies results
4. **Returns a summary** — only a final summary goes back to the main agent (not all the internal details)

```
Main Agent Context Window
├── Full conversation history
├── CLAUDE.md
├── Your instructions
└── ...

       Spawns →

Sub-Agent Context Window (ISOLATED)
├── Fresh start
├── CLAUDE.md (loaded fresh)
├── The spawn prompt (task description)
└── (NO main conversation history)

       Returns →

Summary back to Main Agent
(just the result, not all internal work)
```

This isolation is what makes sub-agents valuable — they don't pollute the main conversation with verbose exploration details.

---

## 4. Built-in Sub-Agent Types

Claude Code comes with several built-in sub-agent types. You don't create these — they're always available.

### `Explore` — Fast Codebase Research Agent

- **Purpose**: Search and understand code without making any changes
- **Tools available**: Read, Grep, Glob, WebFetch, WebSearch (read-only)
- **When Claude uses it**: Searching for a bug's location, understanding code architecture, finding where a function is defined
- **Thoroughness levels**: You (or Claude) can specify `quick`, `medium`, or `very thorough`

**Example scenario:**
> You ask: "Why is the login failing?"
> Claude spawns an Explore agent → it reads auth files, searches for error patterns, investigates the flow → returns a summary of findings → Claude uses that to fix the issue.

---

### `Plan` — Architecture & Planning Agent

- **Purpose**: Research the codebase to help plan before implementing
- **Tools available**: Read, Grep, Glob (read-only, no web)
- **When Claude uses it**: In plan mode (when you press `Shift+Tab`), Claude uses this to gather context before proposing a design
- **Safe by design**: Cannot modify files — only reads

**Example scenario:**
> You press `Shift+Tab` to enter plan mode and say: "Refactor the payment module"
> Plan agent investigates the current architecture → returns findings → main agent designs the refactor plan → shows it to you for approval.

---

### `general-purpose` — Default Multi-Purpose Agent

- **Purpose**: Handles complex multi-step tasks — researching, searching code, executing tasks
- **Tools available**: All tools (unless restricted)
- **When Claude uses it**: When a task doesn't match a more specific agent type
- **Best for**: Broad tasks that need read + write + execute capabilities

---

### `claude-code-guide` — Claude Code Knowledge Agent

- **Purpose**: Answers questions about Claude Code features, settings, hooks, MCP servers, API usage
- **When Claude uses it**: When you ask "how do I..." or "can Claude..." about Claude Code itself
- **Best for**: Learning Claude Code features, understanding configuration options

---

## 5. When & How Sub-Agents Are Called

Sub-agents are called by the **main agent's reasoning** — Claude decides when delegation makes sense.

**Triggering factors:**

1. **Task matches agent description** — If a custom agent is described as "Review PRs for security issues" and you ask Claude to review a PR, it automatically considers using that agent.

2. **Context preservation** — When a task would bloat the main context (like deep code exploration), Claude delegates to Explore.

3. **Specialization** — When a task benefits from a focused expert role (security reviewer, performance analyzer, etc.)

4. **You explicitly ask** — "Use the security-reviewer subagent to check the auth module"

**How the Agent tool is invoked (internally):**

```
Claude calls Agent tool with:
  - subagent_type: "Explore"  (or custom agent name)
  - prompt: "Find all places where user input is not sanitized..."
  - run_in_background: false  (or true for parallel)
  - isolation: "worktree"     (optional, for filesystem isolation)
```

---

## 6. Do Sub-Agents Run Automatically?

**Yes — for built-in agents**, Claude automatically delegates when appropriate.

**For custom agents**, delegation is automatic based on the description you write. If your description is clear and specific, Claude will delegate at the right times without you asking.

**You can also explicitly trigger them:**
```
"Use the code-reviewer subagent to check the auth module"
"Have the debugger agent investigate the crash in payments.js"
```

**The key to good auto-delegation:** Write a clear, specific description in your custom agent's frontmatter:

```yaml
# Vague (bad for auto-delegation):
description: Review code

# Specific (good for auto-delegation):
description: Review code for security vulnerabilities including SQL injection, XSS, and authentication bypass risks
```

---

## 7. Custom Sub-Agents: What, Why & When

### What Are They?

Custom sub-agents are agents you define for your specific workflow. You write a YAML/Markdown file with a system prompt, tool restrictions, and a description of when to use them.

### Why Create Custom Sub-Agents?

**1. Domain Expertise**
Give an agent a highly focused system prompt so it behaves like a specialist:
```
"You are a PostgreSQL performance expert. Analyze queries for N+1 issues, 
missing indexes, and slow join patterns. Always suggest EXPLAIN ANALYZE output."
```

**2. Cost Optimization**
Use `claude-haiku-4-5` for high-volume, straightforward tasks instead of Sonnet:
```yaml
model: claude-haiku-4-5-20251001  # much cheaper for repetitive tasks
```

**3. Safety Enforcement**
Restrict tools to prevent dangerous operations:
```yaml
tools: Read, Grep  # can only read files, cannot edit or run commands
```

**4. Reusability**
Define an agent once, use it across all your projects (user-level agents at `~/.claude/agents/`).

**5. Parallel Processing**
Spawn multiple specialized agents simultaneously for different aspects of a large task.

### When NOT to Create Custom Sub-Agents

- For one-off tasks you'll never repeat
- When the built-in agents already handle your use case
- When the task is simple enough that the main agent handles it fine
- When you don't have a clear, focused purpose for the agent

---

## 8. Creating a Custom Sub-Agent

### Interactive Setup (Recommended)

In Claude Code, run:
```
/agents
```
Then select "Create New subagent" and follow the prompts.

### Manual File Creation

Create a `.md` file in:
- `.claude/agents/` — project-level (only for this project)
- `~/.claude/agents/` — user-level (available in all projects)

**File format:**

```markdown
---
name: security-reviewer
description: Review code for security vulnerabilities including SQL injection, XSS, CSRF, and authentication bypass. Use this when reviewing auth code, API endpoints, or user input handling.
model: claude-sonnet-4-6
tools: Read, Grep, Glob
---

You are a security expert specializing in web application security.

When reviewing code, always check for:
- SQL injection via unparameterized queries
- XSS via unsanitized user output
- CSRF missing token validation
- Authentication bypass through logic flaws
- Sensitive data exposure in logs or responses
- Rate limiting enforcement

Report each finding with:
1. Severity: Critical / High / Medium / Low
2. Location: file:line
3. Description of the vulnerability
4. Recommended fix with code example
```

### All Available Frontmatter Fields

| Field | Description | Example |
|---|---|---|
| `name` | Unique identifier | `security-reviewer` |
| `description` | When Claude should use this agent | `Review code for security vulnerabilities...` |
| `model` | Which Claude model to use | `claude-haiku-4-5-20251001` |
| `tools` | Allowed tools (allowlist or denylist) | `Read, Grep` or `!Bash, !Edit` |
| `isolation` | Filesystem isolation mode | `worktree` |
| `memory` | Persistent memory across sessions | `enabled` |
| `permission-mode` | Agent's permission mode | `default` |
| `disable-model-invocation` | Don't show in Claude's tool list | `true` (manual-only) |

---

## 9. Tool Access & Restrictions

Each sub-agent can have its own set of allowed tools. This is one of the most powerful safety features.

### Allowlist (only these tools)

```yaml
tools: Read, Grep, Glob
# Agent can ONLY read and search — cannot edit or run commands
```

### Denylist (everything except these)

```yaml
tools: "!Bash, !Edit, !Write"
# Agent can use everything EXCEPT running commands or editing files
```

### Practical Examples

| Agent Role | Tools | Why |
|---|---|---|
| Security reviewer | `Read, Grep` | Read-only audit — no accidental modifications |
| Debugger | `Read, Bash` | Can run diagnostic commands, not edit code |
| Code reviewer | `Read, Grep, Edit` | Can suggest edits but not run arbitrary commands |
| Test runner | `Bash, Read` | Run tests and read results |
| Documentation writer | `Read, Grep, Edit, Write` | Read code, write docs |
| Data analyzer | `Read, Bash` | Read data files, run analysis scripts |

### Built-in Agent Tool Access

| Agent | Tools Available |
|---|---|
| `Explore` | Read, Grep, Glob, WebFetch, WebSearch (all read-only) |
| `Plan` | Read, Grep, Glob (filesystem read-only) |
| `general-purpose` | All tools |

---

## 10. Context Window Management

This is one of the biggest benefits of sub-agents.

### The Problem Without Sub-Agents

When Claude explores a large codebase, all those file reads fill up the main context window. A 200-file exploration could use 50,000+ tokens just in context — leaving less room for actual implementation work.

### How Sub-Agents Solve It

```
WITHOUT sub-agents:
Main context = conversation + 200 file reads + implementation work
= quickly fills up → performance degrades

WITH Explore sub-agent:
Main context = conversation + SUMMARY of exploration + implementation work
= lean, focused context → better performance
```

The sub-agent does the heavy lifting in its own context. Only the final summary (a few hundred tokens) comes back to the main agent.

### Context Isolation Model

```
Main Session Context Window
├── Your conversation history
├── CLAUDE.md
├── Previous summaries from sub-agents
└── Current implementation work

Sub-Agent Context Window (independent)
├── The task prompt
├── CLAUDE.md (fresh load)
├── Its own work (file reads, searches, etc.)
└── (NOT the main conversation)
```

### Strategic Use

- Use `Explore` for deep codebase research before big changes
- Use custom agents for repetitive checks (security audits, style reviews)
- This keeps your main session lean and focused on implementation

---

## 11. Parallel & Background Execution

### Parallel Execution

Multiple sub-agents can run at the same time for independent tasks:

```
Claude needs to audit 3 modules simultaneously:

Main Agent
├──(parallel)──> Explore Agent: "Audit auth module"
├──(parallel)──> Explore Agent: "Audit payment module"
└──(parallel)──> Explore Agent: "Audit user module"

All three run simultaneously → all report back → Claude synthesizes results
```

This is especially useful for:
- Auditing multiple independent modules
- Researching different parts of a codebase
- Running checks that don't depend on each other

### Background Execution

Sub-agents can also run in the background while the main agent continues with other work. When a background agent finishes, the main agent is notified automatically.

```
Main Agent spawns sub-agent (background: true)
Main Agent continues working on other things...
...
Sub-agent finishes → Main Agent is notified → uses the results
```

**Important:** You cannot steer or interrupt a running sub-agent. Once spawned, it runs autonomously until done.

---

## 12. Worktree Isolation

For agents that edit files, worktree isolation gives them a **completely separate copy of the repository** to work in.

### How It Works

```yaml
---
name: safe-refactor-agent
isolation: worktree
---
```

When this agent runs:
1. A new git worktree is created at `.claude/worktrees/<name>/`
2. The agent works in that isolated copy
3. If the agent makes changes: the worktree is preserved for you to review
4. If the agent makes no changes: the worktree is automatically deleted

### Why Use It?

- **Parallel editing**: Multiple agents can edit files simultaneously without conflicts
- **Safe experimentation**: Agent's changes don't affect your main working directory
- **Easy review**: You can diff the worktree against main before applying changes
- **Rollback**: Just delete the worktree if you don't like the results

### Example Use Case

```
You: "Refactor the auth module AND the payment module in parallel"

Main Agent spawns:
├── refactor-agent (worktree A): refactors auth/
└── refactor-agent (worktree B): refactors payments/

Both work simultaneously without conflicts.
You review both worktrees and merge the ones you like.
```

---

## 13. Sub-Agents vs Agent Teams

These are two different collaboration models in Claude Code.

### Sub-Agents

- Spawned by the main agent for focused tasks
- Run autonomously and report back (one-way reporting)
- Cannot be messaged mid-task
- No nesting (sub-agents can't spawn other sub-agents)
- Best for: focused, isolated tasks

### Agent Teams

Agent teams are **multiple independent Claude Code sessions** that can communicate directly with each other.

- Each teammate is a full Claude session with its own conversation
- You can send messages directly to any teammate (`Shift+Down` to cycle)
- Teammates can message each other
- Shared task list for coordination
- Full back-and-forth conversation possible
- Best for: collaborative, parallel implementation work

**Comparison:**

| Feature | Sub-Agents | Agent Teams |
|---|---|---|
| Communication | Report back to main only | Direct messaging between all agents |
| Mid-task steering | Not possible | Possible |
| Nesting | Not allowed | Full independence |
| Setup | Simpler | More complex |
| Best for | Isolated research/review | Collaborative implementation |
| Context | Isolated per agent | Independent sessions |

**Rule of thumb:**
- Use **sub-agents** when you need focused, one-shot delegation
- Use **agent teams** when you need multiple agents to collaborate or be redirected

---

## 14. Pros & Cons

### Pros

| Benefit | How It Helps |
|---|---|
| Context preservation | Sub-agent work doesn't bloat main conversation |
| Specialization | Focused system prompts = expert behavior |
| Cost control | Use Haiku for cheap, repetitive tasks |
| Safety | Tool restrictions prevent dangerous operations |
| Parallel work | Multiple agents work simultaneously |
| Reusability | Define once, use across projects |
| Isolation | Sub-agent errors don't affect main session |

### Cons

| Limitation | What It Means |
|---|---|
| No mid-task steering | You can't guide a running sub-agent |
| No inter-agent communication | Sub-agents can't talk to each other |
| No nesting | Sub-agents can't spawn other sub-agents |
| Extra token cost | Each sub-agent has its own context overhead |
| Setup required | Custom agents need upfront configuration |
| Results are summarized | You don't see all of the sub-agent's internal reasoning |

---

## 15. Must-Know Things

### 1. Description drives auto-delegation

The `description` field in your custom agent is what Claude uses to decide when to delegate. Write it as a clear statement of when it should be used:

```yaml
# Good description:
description: Review API endpoints for missing authentication, authorization checks, and rate limiting. Use when reviewing route handlers or middleware.

# Bad description:
description: A helpful security assistant
```

### 2. Sub-agents cannot spawn other sub-agents

No nesting. A sub-agent runs in isolation and cannot create additional sub-agents. This is intentional to prevent cascading complexity.

### 3. CLAUDE.md applies to all sub-agents

Your project's `CLAUDE.md` (project conventions, rules, etc.) is loaded fresh by every sub-agent. So all your team conventions automatically apply.

### 4. Results are always summarized

The main agent only receives a **summary** of the sub-agent's work — not all the internal file reads, search results, etc. This is the key mechanism that keeps the main context lean.

### 5. Model selection matters for cost

```yaml
model: claude-haiku-4-5-20251001   # Fast and cheap — good for repetitive tasks
model: claude-sonnet-4-6           # Balanced — good for most tasks (default)
model: claude-opus-4-6             # Most capable — use for complex reasoning
```

### 6. Persistent memory is available

```yaml
memory: enabled
```

With this, the agent gets a dedicated memory directory that persists across sessions. The agent can build up project-specific knowledge over time.

### 7. Tool restrictions are hard limits

If you set `tools: Read, Grep`, that agent genuinely cannot edit files or run commands — not even if it "decides" to. This makes agents safe to use for audit-only roles.

### 8. User-level vs project-level agents

- `~/.claude/agents/` — available in ALL projects (good for general-purpose agents like security reviewer, code formatter)
- `.claude/agents/` — only available in THIS project (good for project-specific agents)

### 9. View and manage agents

```
/agents
```

This shows all available agents, lets you create new ones, edit existing ones, or delete them.

### 10. Sub-agents are not magic — they need clear instructions

A sub-agent is only as good as its system prompt. Spend time writing a clear, specific prompt that defines the agent's role, what it should check for, and how it should format its output.

---

## Quick Reference: Cheat Sheet

### Built-in Agents

| Name | Use For | Tools |
|---|---|---|
| `Explore` | Codebase research, finding code | Read, Grep, Glob (read-only) |
| `Plan` | Pre-implementation research | Read, Grep, Glob (read-only) |
| `general-purpose` | General multi-step tasks | All tools |
| `claude-code-guide` | Questions about Claude Code | Read, Web |

### Custom Agent Template

```markdown
---
name: your-agent-name
description: Specific description of when Claude should use this agent and what it does.
model: claude-sonnet-4-6
tools: Read, Grep, Edit
---

You are a [role] specializing in [domain].

When given a task, always:
1. [Step 1]
2. [Step 2]
3. [Step 3]

Format your output as:
[output format instructions]
```

### File Locations

| Scope | Path |
|---|---|
| User-level agents | `~/.claude/agents/` |
| Project-level agents | `.projectroot/.claude/agents/` |
| Manage agents | `/agents` in Claude Code |

### Decision: Use Sub-Agent or Not?

```
Is the task focused and well-defined?
├── Yes → Sub-agent is a good fit
└── No  → Handle in main agent

Will it pollute the main context?
├── Yes (deep exploration, many reads) → Use Explore sub-agent
└── No (quick task) → Main agent is fine

Do you need to steer it mid-task?
├── Yes → Use Agent Teams instead
└── No  → Sub-agent is fine

Is it a repetitive specialized check?
├── Yes → Create a custom sub-agent
└── No  → Main agent handles it
```
