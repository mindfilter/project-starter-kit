# project-starter-kit

A curated collection of Claude Code skills and plugins for intelligent project workflows.

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add mindfilter/project-starter-kit
```

Then browse and install individual plugins:

```
/plugin install project-setup@project-starter-kit
```

## Plugins

| Plugin | Description |
|--------|-------------|
| **project-setup** | Intelligently bootstraps any project — interviews you, installs the right tools, scaffolds the full structure |

### project-setup skills

| Skill | Triggers on | What it does |
|-------|------------|--------------|
| **agentic-project-setup** | "multi-agent", "orchestrator", "swarm", "agent pipeline", "distributed agents", and similar | Scaffolds a complete multi-agent orchestration project: interviews you about agent roles and orchestration pattern, generates a concept map of the full system for review before building, then creates agent stubs, prompt files, OKR module, token budget utility, and an OKR-aware orchestrator |

#### Orchestration patterns covered

- **Orchestrator-worker** — central coordinator routes to N specialized agents
- **Pipeline** — agents process in strict sequence, each transforming the previous output
- **Fan-out / fan-in** — orchestrator spawns parallel agents, aggregates results
- **Router** — classifier agent dispatches to the right specialist based on input type
- **Swarm / peer-to-peer** — agents hand off directly with no central coordinator

#### What gets scaffolded

- `CLAUDE.md` with agent roles table, OKR contract, orchestration pattern, and operational considerations
- `src/agents/` — one typed stub per named agent, inheriting from a shared base
- `src/prompts/agents/` — one system prompt file per agent
- `src/okr/` — OKR module (types, registry, evaluator, escalation) driving self-correction
- `src/budget.py` (or `.ts`) — `TokenBudget` utility for resource-aware orchestration
- `src/orchestrator.py` (or `.ts`) — OKR-aware orchestrator stub with dispatch, evaluation, and self-correction loop
- `.mcp.json` — memory + context7 + agent-specific MCP servers
- `.claude/settings.json` — formatter hooks, prompt file validation, `.env` protection

## Requirements

Before using `project-setup`, install these from the official marketplace:

```
/plugin install context7@claude-plugins-official
/plugin install github@claude-plugins-official
/plugin install playground@claude-plugins-official
```

## Contributing

Open a PR or issue on [GitHub](https://github.com/mindfilter/project-starter-kit) to suggest new plugins or improvements.
