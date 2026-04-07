# MCP in Claude Code — What It Is and How to Use It

A plain-English guide to understanding MCP, why it exists, and how to set it up.

---

## What is MCP?

**MCP stands for Model Context Protocol.**

By default, Claude only knows what's in the conversation — the files you share, the code in your project. It has no way to talk to your database, browse the web, or call an external API on its own.

MCP is the bridge that fixes this. It's an open standard that lets Claude connect to external tools and services — databases, GitHub, browsers, Slack, and more. Once connected, Claude can use those tools automatically when they're relevant to your request.

**Simple analogy:** Claude is a smart assistant sitting in a room. MCP opens doors to other rooms — one door leads to your database, another to GitHub, another to a browser. Without MCP, Claude can only work with what's already in the room with it.

---

## Why Use MCP?

Without MCP, you have to copy-paste data into the chat manually.

With MCP, Claude can:
- Query your database and use the results directly
- Read/write files on your filesystem
- Open a browser and interact with web pages
- Fetch GitHub issues, PRs, and code
- Read from Slack, Notion, or other tools

It removes manual copy-paste and lets Claude work with live, real data.

---

## How It Works

You configure one or more **MCP servers** — small programs that expose tools to Claude. Each server is responsible for one integration (e.g. one server for your database, another for GitHub).

Once configured, Claude **auto-discovers** the tools those servers expose. You don't have to invoke them manually — Claude uses them on its own when they're relevant to what you asked.

---

## How to Add an MCP Server

Use the `claude mcp add` command:

```bash
# Add a GitHub MCP server (HTTP transport)
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# Add a PostgreSQL MCP server (stdio transport)
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://user:pass@localhost:5432/mydb"
```

Manage your servers:

```bash
claude mcp list              # see all configured servers
claude mcp get github        # see details of one server
claude mcp remove github     # remove a server
```

---

## Scopes — Where the Config Lives

MCP servers can be configured at three levels:

| Scope | Config file | Use for |
|---|---|---|
| **local** (default) | `~/.claude.json` | Private/experimental servers |
| **project** | `.mcp.json` (project root) | Team-shared servers, committed to git |
| **user** | `~/.claude.json` | Personal servers across all projects |

```bash
# Add to project scope (shared with team via git)
claude mcp add --scope project --transport http github https://api.githubcopilot.com/mcp/

# Add to user scope (your personal tools, all projects)
claude mcp add --scope user --transport stdio myserver -- npx my-mcp-server
```

---

## Real-World Examples

### Connect to a PostgreSQL database
```bash
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://user:pass@localhost:5432/mydb"
```
Now you can ask: *"Show me all users who signed up in the last 7 days"* — Claude queries the DB directly.

### Connect to GitHub
```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```
Now you can ask: *"Summarize the open issues labeled 'bug'"* — Claude fetches them from GitHub.

### Connect to Sentry
```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```
Now you can ask: *"What errors are happening most in production?"* — Claude reads from Sentry.

---

## When to Use MCP

| Use MCP when... | Skip MCP when... |
|---|---|
| You need Claude to read live data (DB, API) | The data fits in a file you can just share |
| You want Claude to interact with external services | You only need Claude to work with local code |
| Your team uses the same tools repeatedly | It's a one-off task |
| You want to avoid manual copy-paste | |

---

## Important Things to Know

- **MCP servers run with full access** — only add servers you trust. A malicious MCP server could do damage.
- **Claude uses tools automatically** — once configured, you don't have to ask Claude to use the tool. It decides when it's relevant.
- **Large outputs are capped** — MCP output is limited to 10,000 tokens by default to keep context manageable.
- **Project-scoped servers go in `.mcp.json`** — commit this file so your whole team shares the same tools.
- **Each server handles one integration** — don't try to put everything in one server.
