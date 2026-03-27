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
│   ├── budget.py                ← token budget tracker; orchestrator imports this
│   ├── okr/                     ← OKR module (skip for Swarm/P2P)
│   │   ├── __init__.py
│   │   ├── types.py             ← KRUpdate dataclass
│   │   ├── registry.py          ← KeyResult, Objective, OKRRegistry
│   │   ├── evaluator.py         ← evaluate(), process_updates(), SelfCorrectionAction
│   │   └── escalation.py        ← format_escalation_report()
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
│   ├── budget.ts                ← token budget tracker; orchestrator imports this
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

## Operational Considerations

**Token cost**: Multi-agent systems run at significantly higher token cost than single-agent — budget accordingly for production workloads. Token budget management is in `src/budget.py` (or `budget.ts`).

**Failure handling**:
- Critical agents (blocks the pipeline if they fail): [list them]
- Optional agents (system degrades gracefully if they fail): [list them]
- Retry strategy: [e.g., exponential backoff with 3 retries; circuit breaker after 5 consecutive failures]

## OKR Contract

**Mission:** [mission from interview]
**Vision:** [vision from interview]

OKRs are initialized at run start by `initialize_okrs()` in `src/orchestrator.py` using the mission, vision, and current task. The LLM derives concrete objectives and key results appropriate to the task — no manual pre-definition required.

**How KRs are assigned to agents:**
- The orchestrator selects relevant KR IDs for each agent's role
- KR IDs and descriptions are injected into the agent's task instructions
- Agents return a `KRUpdate` (score 0.0–1.0 + narrative) for each assigned KR

**Self-correction progression:**
1. Score below threshold, attempts=0 → retry with adjusted instructions
2. Score below threshold, attempts=1 → re-route to alternate agent
3. Score below threshold, attempts=2 → downgrade threshold 20%, continue
4. Score below threshold, attempts=3 → escalate to human (see below)

**Escalation:**
When all self-correction is exhausted, `format_escalation_report()` prints a structured report and halts the run. The human decides: revise instructions, accept current progress, or abort.

**Dashboard:**
`okr-status.json` is written after every agent cycle. Read it at any point to see current objective scores and KR narratives without interrupting the run.

## Conventions
- Agents are stateless — all state is passed via the context object, never stored on the agent instance
- Each agent has exactly one system prompt file in `src/prompts/agents/`
- Tools are defined in `src/tools/` and imported by agents that need them; never defined inline
- All inter-agent communication goes through the orchestrator; agents do not call each other directly
- The orchestrator checks `TokenBudget.agent_config()` before each dispatch and respects `None` (skip) returns
- Agents never import `src/okr/` directly — they receive KR IDs in their task instructions and return `KRUpdate` objects in their output
```
