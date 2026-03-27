# Agent Orchestration Patterns

Reference: load this file during research and scaffold phases when agentic signals are detected.

---

## Orchestration Patterns

| Pattern | Description | Use When |
|---------|-------------|----------|
| **Orchestrator-worker** | One orchestrator routes tasks to N specialized workers, collects results | Tasks are categorically distinct; each worker is an expert in one domain |
| **Pipeline** | Agents pass output sequentially — A → B → C | Each stage transforms output from the previous; strict ordering required |
| **Fan-out / fan-in** | Orchestrator spawns parallel agents for the same task, aggregates results | Tasks are independent and can run concurrently (e.g., multi-source research) |
| **Router** | A classifier agent inspects input and hands off to the right specialist | Input type is unpredictable at design time; specialists are mutually exclusive |

Most real systems combine patterns. A router often dispatches to orchestrator-worker sub-graphs.

---

## Canonical Folder Layout

Language-agnostic concept names — substitute `.py` or `.ts` based on the project stack.

```
project-root/
├── orchestrator.py          # Entry point; routing and coordination logic
├── agents/
│   ├── base.py              # Shared interface all agents implement
│   ├── researcher.py        # One file per specialized agent
│   ├── writer.py
│   └── reviewer.py
├── workflows/
│   ├── research_pipeline.py # Multi-step workflow: ordered sequence of agent calls
│   └── review_cycle.py
├── state/
│   └── context.py           # Typed context object passed between agents
├── tools/
│   ├── search.py            # Tool definitions shared across agents
│   └── file_io.py
└── prompts/
    ├── orchestrator.md      # Orchestrator system prompt
    └── agents/
        ├── researcher.md    # One prompt file per agent role
        ├── writer.md
        └── reviewer.md
```

Key rule: one agent per file. If a file is doing two things, split it.

---

## State Management Strategies

### Context passing

Pass a typed context object through every orchestrator call. Agents read from it and write results back into it. Nothing travels via global variables or module-level state.

```python
# state/context.py
@dataclass
class RunContext:
    task: str
    research_results: list[str] = field(default_factory=list)
    draft: str = ""
    review_notes: list[str] = field(default_factory=list)
```

The orchestrator owns the context object. Agents receive slices, not the whole thing.

### Shared memory (cross-session)

Use an MCP memory server when state must persist across conversations or agent restarts.

```
claude mcp add memory -- npx -y @modelcontextprotocol/server-memory
```

Store only structured, retrievable facts — not raw conversation history.

### Agent isolation

Each agent call receives only the context slice it needs. Do not pass the full state object to every agent — it leaks irrelevant information, inflates prompts, and makes agents harder to test in isolation.

```python
# Good: pass only what the writer needs
writer_agent.run(draft_prompt=ctx.task, research=ctx.research_results)

# Bad: pass everything
writer_agent.run(ctx)
```

---

## CLAUDE.md Conventions for Agentic Projects

Every agentic project needs a CLAUDE.md that covers these sections:

### Agent roles table

| Agent | Responsibility | Input | Output |
|-------|---------------|-------|--------|
| `researcher` | Gather facts on a topic | Task string, search tool access | List of findings |
| `writer` | Draft content from research | Task + findings | Draft string |
| `reviewer` | Score and annotate a draft | Draft + rubric | Score, notes |

### Orchestration pattern in use

State the pattern explicitly:

> This project uses a **pipeline** pattern: researcher → writer → reviewer. The orchestrator in `orchestrator.py` runs each stage in order and passes the context object between them.

### State contract

Document what lives in the context object and who writes to each field:

| Field | Type | Written by | Read by |
|-------|------|-----------|---------|
| `task` | str | Caller | All agents |
| `research_results` | list[str] | researcher | writer |
| `draft` | str | writer | reviewer |
| `review_notes` | list[str] | reviewer | orchestrator |

### How to add a new agent

1. Create `agents/your_agent.py` implementing the `BaseAgent` interface
2. Add a prompt file at `prompts/agents/your_agent.md`
3. Register it in the orchestrator routing logic
4. Add its input/output fields to `state/context.py`
5. Add a row to the agent roles table in CLAUDE.md

### Communication

Document how agents receive tasks and return results — function call, message queue, MCP tool, etc. Be explicit about sync vs. async.

---

## Key Design Principles

**One agent per file** — clear responsibility boundaries; easy to test, replace, or extend individual agents without touching others.

**Prompts as `.md` files** — version-controlled, reviewable in PRs, editable without touching code. One file per agent role.

**State is explicit** — context object is typed; no hidden globals. Every field has an owner and a consumer.

**Agents are stateless** — all state is passed in, nothing stored on the agent object itself. The same agent can be called with different contexts without side effects.
