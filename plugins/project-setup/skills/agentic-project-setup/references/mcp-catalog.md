# MCP Server Catalog

Reference: load this file when recommending MCP servers based on project signals for multi-agent orchestration projects.

## Signal → Server Mapping

| Project Signal | MCP Server | Install Command | Why |
|----------------|------------|-----------------|-----|
| Any multi-agent project (always include) | **memory** | `claude mcp add memory -- npx -y @modelcontextprotocol/server-memory` | Cross-agent and cross-session state persistence — orchestrators need to remember context across agent calls |
| Uses React, Next.js, Vue, or any popular library / Claude Agent SDK / Anthropic SDK | **context7** | `claude mcp add context7 -- npx -y @upstash/context7-mcp@latest` | Live, version-accurate documentation lookup — essential for SDK-heavy agent development |
| Has a research-agent role / needs web search | **brave-search** | `claude mcp add brave-search` | Web search capability for research agents without needing a full browser |
| GitHub repository / PR-automation agent / code review agent | **github** | `claude mcp add github` | Issues, PRs, Actions, code search — for agents that work with repositories |
| PostgreSQL / Supabase database storage for agent state | **supabase** or **postgres** | `claude mcp add supabase` | Direct DB access for agents that read/write structured state |
| Browser automation agent / needs visual testing | **playwright** | `claude mcp add playwright -- npx -y @playwright/mcp@latest` | Browser automation for agents that interact with web UIs |
| Document processing / file analysis agent | **filesystem** | `claude mcp add filesystem` | Read/write local files for agents that process documents |

## .mcp.json Template

After deciding on servers, write `.mcp.json` in the project root:

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

memory + context7 are recommended for every agentic project. Add others based on your agent roles.

Check this file into the repo so the whole team gets the same servers.
