# Connectors in Claude Code — What They Are and How They Work

---

## What Are Connectors?

Connectors are **pre-built integrations** (Slack, Telegram, Notion, Gmail, etc.) that feed external data into Claude when it runs a **scheduled task**.

They are **not** for regular chat. They exist specifically for automated, scheduled jobs — when Claude runs on a schedule without you being present, connectors give it a source of input to work from.

**Simple analogy:** You set up a job that runs every morning. Claude wakes up, reads your Slack messages via the Slack connector, summarizes overnight discussions, and posts a digest. You didn't have to do anything — the connector fed the data in automatically.

---

## Connectors vs MCP Servers

These two are often confused. Here's the difference:

| | Connectors | MCP Servers |
|---|---|---|
| **What it is** | Pre-built integration for scheduled tasks | Flexible tool protocol (pre-built or custom) |
| **Where it works** | Web scheduled tasks only | Everywhere (CLI, desktop, web) |
| **Purpose** | Deliver input context to a scheduled job | Give Claude tools to read/write external systems |
| **Triggered by** | A schedule (cron) | Claude deciding to use the tool |
| **Setup** | Configured when creating a scheduled task | Configured via `claude mcp add` |

**In short:**
- Use **connectors** when you want Claude to automatically pull data from a service on a schedule
- Use **MCP servers** when you want Claude to actively query or interact with a service during a conversation

---

## When Do You Need Connectors?

You need connectors when:

- You want Claude to run **automatically** without you being in the chat
- Claude needs **fresh data from an external service** as input (new Slack messages, new emails, new Notion updates)
- You're building an **automated workflow** — daily reports, monitoring, digests, alerts

You don't need connectors when:
- You're just chatting with Claude
- You want Claude to use a tool during a conversation (use MCP instead)
- The task is one-off, not recurring

---

## How It Works — Step by Step

1. **You create a scheduled task** on Claude Code web
2. **You attach a connector** (e.g. Slack) as the input source
3. **You set a schedule** (e.g. every morning at 8am)
4. At the scheduled time, Claude wakes up, **pulls data from the connector**, and runs your task using that data
5. Output can be written back, posted somewhere, or stored — depending on your setup

---

## Real-World Examples

### Example 1 — Daily Slack Digest
```
Connector: Slack
Schedule: Every day at 9am
Task: Summarize all unread messages from #engineering channel 
      and post a bullet-point digest back to #standup
```
Claude reads Slack messages each morning and posts the summary — no manual work.

### Example 2 — Gmail Triage
```
Connector: Gmail
Schedule: Every hour
Task: Check new emails, label urgent ones, draft replies 
      for routine requests
```
Claude monitors your inbox and handles routine emails on a schedule.

### Example 3 — Notion Updates
```
Connector: Notion
Schedule: Every Friday at 5pm
Task: Read all pages updated this week in the "Projects" database
      and generate a weekly progress report
```
Claude reads your Notion workspace and writes a weekly report automatically.

---

## How to Set Up a Connector

Connectors are configured through **Claude Code on the web** (not the CLI).

When creating a scheduled task:
1. Go to Claude Code web → Scheduled Tasks
2. Create a new task
3. Choose a **connector** as the input source
4. Set your repository, branch, and environment
5. Write your task prompt
6. Set the schedule

> The CLI (`claude` command) does not support connectors directly — connectors are a web-only feature tied to scheduled tasks.

---

## Quick Summary

| Question | Answer |
|---|---|
| What are connectors? | Pre-built integrations that feed data into scheduled Claude tasks |
| Where do they work? | Claude Code web only (not CLI or desktop) |
| When do I need them? | When automating recurring tasks that pull from external services |
| How are they different from MCP? | Connectors = scheduled input; MCP = interactive tool use |
| Can I build custom connectors? | Not directly — use MCP servers for custom integrations |
