# Folder Structure Templates

Reference: load the section matching the user's stack when creating the multi-agent project scaffold.

## Multi-Agent Orchestrator (Python)

```
project-name/
├── .claude/
│   └── settings.json       ← hooks + permissions
├── .mcp.json               ← MCP server config
├── src/
│   ├── orchestrator.py          ← main entry, routes tasks to agents
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── base.py              ← shared agent interface / ABC
│   │   └── [role].py            ← one file per specialized agent (e.g. researcher.py)
│   ├── workflows/               ← multi-step workflow definitions
│   │   └── __init__.py
│   ├── state/
│   │   ├── __init__.py
│   │   └── context.py           ← typed context object passed between agents
│   ├── tools/                   ← shared tool definitions (MCP wrappers, API clients)
│   │   └── __init__.py
│   ├── prompts/
│   │   ├── orchestrator.md      ← orchestrator system prompt
│   │   └── agents/              ← one prompt file per agent role
│   └── utils/
├── tests/
│   └── conftest.py
├── CLAUDE.md
├── README.md
├── .env
├── .env.example
├── .gitignore
└── pyproject.toml
```

## Multi-Agent Orchestrator (TypeScript)

```
project-name/
├── .claude/
│   └── settings.json
├── .mcp.json
├── src/
│   ├── orchestrator.ts          ← main entry, routing logic
│   ├── agents/
│   │   ├── index.ts
│   │   ├── base.ts              ← shared agent interface
│   │   └── [role].ts            ← one file per specialized agent
│   ├── workflows/               ← workflow definitions
│   │   └── index.ts
│   ├── state/
│   │   └── context.ts           ← typed context object
│   ├── tools/                   ← shared tool definitions
│   │   └── index.ts
│   └── prompts/
│       ├── orchestrator.md
│       └── agents/              ← one .md per agent role
├── tests/
│   └── setup.ts             ← test setup / shared fixtures
├── CLAUDE.md
├── README.md
├── .env.example
├── .gitignore
├── tsconfig.json
└── package.json
```

## CLAUDE.md Template for Agentic Projects

Always generate this file. Replace [brackets]:

```markdown
# [Project Name]

## Project Overview
[1-2 sentences describing what this multi-agent system does and what problem it solves.]

## Tech Stack
- **Runtime**: [Node 20 / Python 3.12]
- **Framework**: [Claude Agent SDK / LangChain / custom]
- **LLM Client**: [anthropic / @anthropic-ai/sdk]
- **Testing**: [pytest / vitest / jest]

## Agent Architecture

### Orchestration Pattern
[e.g., "Orchestrator-Worker" — a single orchestrator receives tasks and delegates to specialized worker agents.]

### Agent Roles

| Name | Responsibility | Input | Output |
|------|---------------|-------|--------|
| orchestrator | Routes tasks, aggregates results | user request | final response |
| [role] | [what it does] | [what it receives] | [what it returns] |

### State Contract
The context object (`src/state/context.py` or `src/state/context.ts`) is the single source of truth passed between agents. It contains:
- [field]: [description]
- [field]: [description]

### Adding a New Agent
1. Create `src/agents/[role].py` (or `.ts`) implementing the base agent interface
2. Create `src/prompts/agents/[role].md` with the agent's system prompt
3. Register the agent in `src/orchestrator.py` (or `.ts`) routing logic

## Key Commands
```bash
# Install dependencies
[pip install -e ".[dev]" / npm install]

# Run the orchestrator
[python -m src.orchestrator / npm start]

# Run tests
[pytest / npm test]
```

## Environment Variables
See `.env.example` for required variables. Copy to `.env` and fill in values.

Key variables:
- `ANTHROPIC_API_KEY` — required for all agent LLM calls
- [OTHER_VAR] — [description]

## Conventions
- Agents are stateless — all state is passed via the context object, never stored on the agent instance
- Each agent has exactly one system prompt file in `src/prompts/agents/`
- Tools are defined in `src/tools/` and imported by agents that need them; never defined inline
- All inter-agent communication goes through the orchestrator; agents do not call each other directly
```
