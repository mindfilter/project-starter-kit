# project-setup

An intelligent project bootstrapper for Claude Code. Includes specialized skills for different project types — currently featuring `agentic-project-setup` for multi-agent orchestration projects.

## Skills

### agentic-project-setup

Scaffolds a production-ready multi-agent orchestration project from a conversation. When invoked, this skill:

1. **Interviews you** — agent roles, orchestration pattern, tech stack, team context
2. **Selects the right pattern** — orchestrator-worker, pipeline, fan-out/fan-in, router, or swarm/peer-to-peer, with a decision guide if you're unsure
3. **Researches** live docs (context7) and community templates (GitHub) for your specific stack
4. **Plans** the full setup and gets your approval before touching anything
5. **Scaffolds** every file with your actual agent names — no `[placeholder]` left behind

**Triggers on**: "multi-agent", "agent orchestration", "orchestrator", "swarm", "peer-to-peer agents", "agent pipeline", "distributed agents", and similar phrases.

#### What gets generated

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Agent roles table, state contract, orchestration pattern, operational considerations |
| `src/agents/base.py` | Shared agent interface / ABC |
| `src/agents/[role].py` | One typed stub per named agent |
| `src/prompts/agents/[role].md` | One system prompt file per agent |
| `src/state/context.py` | Typed context object with documented field ownership |
| `src/budget.py` | `TokenBudget` — resource-aware orchestration utility |
| `src/orchestrator.py` | Routing stub showing dispatch and budget-check pattern |
| `.mcp.json` | memory + context7 + agent-specific MCP servers |
| `.claude/settings.json` | Formatter hooks, prompt file validation, `.env` protection |

TypeScript projects use `.ts` equivalents throughout.

#### Orchestration patterns

- **Orchestrator-worker** — central coordinator routes to N specialized agents
- **Pipeline** — strict sequential processing, each stage transforms the last
- **Fan-out / fan-in** — parallel agents on independent subtasks, results aggregated
- **Router** — classifier dispatches to the right specialist based on input type
- **Swarm / peer-to-peer** — agents hand off directly; no central coordinator

#### Token budget management

The scaffolded `TokenBudget` utility gives the orchestrator awareness of remaining token budget across all agent calls. Before each dispatch, the orchestrator checks `budget.agent_config(priority)` — which scales `max_tokens` down as budget tightens, or returns `None` to skip low-priority agents entirely. This mirrors how managers allocate limited resources: less available means fewer agents or reduced output, not a hard failure.

## Requirements

- context7 MCP server (for live documentation lookup)
- github MCP plugin (for community resource discovery)

Both are available via the official Claude Code marketplace:

```
/plugin install context7@claude-plugins-official
/plugin install github@claude-plugins-official
```

## Usage

After installing this plugin, start a new project session and say:

> "Set up a multi-agent system" or "Build an agent pipeline" or "I want to scaffold a swarm orchestration project"

The skill will guide you through the rest.
