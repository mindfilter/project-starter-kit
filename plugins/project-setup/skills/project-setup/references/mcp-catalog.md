# MCP Server Catalog

Reference: load this file when recommending MCP servers based on project signals.

## Signal → Server Mapping

| Project Signal | MCP Server | Install Command | Why |
|----------------|------------|-----------------|-----|
| Uses React, Next.js, Vue, Express, Django, FastAPI, or any popular library | **context7** | `claude mcp add context7 -- npx -y @upstash/context7-mcp@latest` | Live, version-accurate documentation lookup — eliminates hallucinated APIs |
| PostgreSQL / Supabase database | **supabase** | `claude mcp add supabase` | Direct DB query, schema inspection, RLS management |
| Any SQL database (PostgreSQL, MySQL, SQLite) | **sqlite** or **postgres** | `claude mcp add sqlite` | Query tables, inspect schema, run migrations inline |
| GitHub repository / needs PR/issue management | **github** | `claude mcp add github` | Issues, PRs, Actions, code search |
| Frontend app needing browser testing | **playwright** | `claude mcp add playwright -- npx -y @playwright/mcp@latest` | Browser automation, visual testing, screenshot capture |
| Uses Stripe / payments | **stripe** | `claude mcp add stripe -- npx -y @stripe/agent-toolkit@latest` | Payment intents, customers, subscriptions |
| AWS infrastructure | **aws-kb-retrieval** | (see AWS docs) | CloudFormation, Lambda, S3, IAM queries |
| Needs cross-session memory | **memory** | `claude mcp add memory -- npx -y @modelcontextprotocol/server-memory` | Persistent key-value store across conversations |
| Uses Sentry for error tracking | **sentry** | `claude mcp add sentry` | Fetch errors, traces, releases |
| Docker containers | **docker** | `claude mcp add docker` | Container management, logs, exec |
| Linear for issue tracking | **linear** | `claude mcp add linear` | Issues, projects, cycles |
| Slack workspace | **slack** | `claude mcp add slack` | Messages, channels, notifications |
| Needs web research | **brave-search** | `claude mcp add brave-search` | Web search without browser |

## .mcp.json Template

After deciding on servers, write `.mcp.json` in the project root:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

Check this file into the repo so the whole team gets the same servers.
