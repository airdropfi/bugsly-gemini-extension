# Bugsly for Gemini CLI

> Bugsly is error tracking built for AI agents to operate: your agent (ChatGPT, Claude, Cursor, Gemini CLI) reads, triages and, when asked, resolves issues over MCP while you tag the work, approve and review. It is hosted, accepts Sentry SDK events and adds AI explanations, cron and uptime monitoring and session replay.

## At a glance

| | |
|---|---|
| What it is | Bugsly is error tracking built for AI agents to operate: your agent (ChatGPT, Claude, Cursor, Gemini CLI) reads, triages and, when asked, resolves issues over MCP while you tag the work, approve and review. It is hosted, accepts Sentry SDK events and adds AI explanations, cron and uptime monitoring and session replay. |
| Who it is for | Engineering teams at startups and small companies that let an AI agent work on their errors (the agent reads and triages, you approve and audit); individual developers on the Free plan. |
| What it does not do | Bugsly is not open source and cannot be self-hosted, has no single sign-on, and never merges code by itself. This repository is a connector, not an error tracker: you need a Bugsly account with a project that has received errors. |
| Price | Free $0 (100 issues/month), Team $19/month (2,000 issues), Business $49/month (20,000 issues), Enterprise custom; billed by unique issues, unlimited users on paid plans. Current numbers: https://bugsly.dev/pricing.md |
| MCP server | https://bugsly.dev/api/mcp (Streamable HTTP, OAuth 2.1). Official MCP Registry name: `dev.bugsly/mcp-server` |
| Human oversight | OAuth consent page, every connection listed and revocable under Settings, AI Agent / MCP, destructive tools annotated so the client can ask first, status changes reversible in the dashboard. |
| More | https://bugsly.dev/mcp (install steps, agent quickstart) , https://bugsly.dev/llms.txt , https://bugsly.dev/.well-known/mcp/server-card.json |

## Connect in 60 seconds

```bash
gemini extensions install https://github.com/airdropfi/bugsly-gemini-extension
```
Restart Gemini CLI, run `/mcp auth bugsly`, sign in to Bugsly in the browser and press Allow.

## Prompts to try

Tag the work, then review: add "Show me what you read and what you changed so I can approve" to any of these.

- "Use Bugsly to list my projects and show the 5 newest unresolved errors in the first one."
- "Use Bugsly to find the issue about TypeError, open its latest event and explain the stack trace and the likely cause."
- "Use Bugsly to check whether my cron monitors and uptime monitors are healthy."
- "Use Bugsly to give me the crash-free sessions and users percentage for the last 30 days."
- "Use Bugsly to read the most frequent unresolved issue, find the failing line in this repository and propose a fix. Do not change the issue's status."

Site: https://bugsly.dev - MCP page: https://bugsly.dev/mcp - Registry: https://registry.modelcontextprotocol.io/v0/servers?search=dev.bugsly

---

Read, search and triage the errors in your own [Bugsly](https://bugsly.dev) projects without leaving Gemini CLI. The extension connects Gemini to the Bugsly MCP server and adds three slash commands and a context file. Gemini can read issues with stack traces and breadcrumbs, check cron monitors, uptime monitors and crash-free rate, and, only when you ask, mark one issue resolved or ignored.

## Before you start

- **You need a Bugsly account** (sign up at https://bugsly.dev) with a project that has received errors. This extension is a connector, not an error tracker by itself.
- Reading is harmless. `resolve_issue` and `ignore_issue` change an issue's status for everyone on the project (reversible in the dashboard), and `create_project` creates a real project. Gemini CLI asks you to confirm MCP tool calls.
- You need Gemini CLI and a browser once, for sign-in.

## Install

```bash
gemini extensions install https://github.com/airdropfi/bugsly-gemini-extension
```

Restart Gemini CLI, then sign in:

```text
/mcp auth bugsly
```

A browser window opens: sign in to Bugsly and approve on the consent page. Sign-in is OAuth 2.1 with PKCE and dynamic client registration, discovered from the server, so there is no key to paste. The access appears in Bugsly under Settings, AI Agent / MCP, as a token you can revoke at any time. Check it with `/mcp` and try `/bugsly:issues`.

Update later with `gemini extensions update bugsly`; remove with `gemini extensions uninstall bugsly`.

### Antigravity CLI

This repository is also an Antigravity plugin (`.antigravity-plugin/plugin.json`, `mcp_config.json`, `rules/AGENTS.md`):

```bash
agy plugin install https://github.com/airdropfi/bugsly-gemini-extension
```

It connects the same MCP server (`https://bugsly.dev/api/mcp`) with the plan-checkout tool switched off. Antigravity plugins have no slash commands. We have not run the plugin end to end in Antigravity.

## Commands

| Command | What it does |
|---|---|
| `/bugsly:issues [project]` | Newest unresolved issues of a project. Read-only. |
| `/bugsly:triage <issue id or keyword>` | Reads an issue's events and stack trace, explains the cause, proposes a fix. Changes nothing unless you ask. |
| `/bugsly:health [project]` | Cron monitors, uptime monitors, crash-free rate, event usage. Read-only. |

You can also just ask ("what broke in my Demo Shop project today?"); `GEMINI.md` carries the same rules.

## Tools

The server exposes 16 tools, each with read-only/destructive hints. This extension switches off `get_checkout_url` (`excludeTools`).

| Tool | What it does | Kind |
|---|---|---|
| `list_projects` | List the projects in your first Bugsly organization, with project IDs and DSN host (the DSN key is not shown; the token acts as your user and other organizations are not listed). | read-only |
| `list_issues` | List issues for a Bugsly project. | read-only |
| `get_issue` | Get details of a specific issue including metadata. | read-only |
| `get_issue_events` | Get recent events (occurrences) for an issue with stack traces and breadcrumbs. | read-only |
| `resolve_issue` | Mark an issue as resolved. | changes data, destructive hint |
| `ignore_issue` | Mark an issue as ignored. | changes data, destructive hint |
| `search_errors` | Search for issues by title keyword. | read-only |
| `get_project_stats` | Get transaction performance stats for a project. | read-only |
| `create_project` | Create a new project in your Bugsly organization (the first one unless org_id is given). | writes |
| `get_usage` | Get current billing usage for your organization (events used, limit, plan). | read-only |
| `get_checkout_url` | Creates a Polar checkout link for the Team or Business plan; plans and prices are listed at https://bugsly.dev/pricing. | writes |
| `list_monitors` | List cron monitors for a project. | read-only |
| `get_monitor` | Get details and recent check-ins for a cron monitor. | read-only |
| `list_uptime_monitors` | List uptime monitors for a project. | read-only |
| `get_uptime_status` | Get uptime stats (uptime %, avg latency, p95) for a monitor. | read-only |
| `get_crash_free` | Get crash-free sessions % and crash-free users % (release-health stability) for a project over the last N days. | read-only |

## What it will and will not do

- It will not resolve or ignore an issue unless you ask for that issue.
- Event content is returned wrapped as untrusted data, and the context file tells Gemini to treat it as data, not instructions. That reduces, but does not remove, prompt-injection risk from error text: review proposed code changes.
- If you mark the `bugsly` server as trusted in your own Gemini settings, confirmations are skipped, including for status changes; the extension does not set `trust` and we recommend you leave it off.

## Contents

```text
gemini-extension.json        manifest: name, version, remote MCP server https://bugsly.dev/api/mcp
.antigravity-plugin/plugin.json  Antigravity plugin manifest
mcp_config.json              Antigravity MCP server definition
rules/AGENTS.md              Antigravity rule (a copy of GEMINI.md: keep both in sync)
GEMINI.md                    context: what Bugsly is, the tools, the rules
commands/bugsly/*.toml       /bugsly:issues, :triage, :health
assets/icon.png              icon
scripts/validate.mjs         schema checks, no dependencies: node scripts/validate.mjs
```

## Data and privacy

The MCP server receives what the tools need: the project IDs, issue IDs and search terms the agent sends, and it returns your issue, event, monitor and usage data from your Bugsly account. Issue and event data are returned wrapped as untrusted data; event output is size-capped and omits user email, IP address, username and extra data, and the DSN key is not shown. See the [privacy policy](https://bugsly.dev/privacy) and [terms](https://bugsly.dev/terms). The extension itself stores nothing and runs no code on your machine; it is a server address, Markdown and prompts.

## Support

https://bugsly.dev/agents or hello@bugsly.dev.

## License

MIT for the files in this repository (see `LICENSE`). The Bugsly service is governed by its terms.
