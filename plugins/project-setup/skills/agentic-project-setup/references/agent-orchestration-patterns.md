# Agent Orchestration Patterns

Reference: load this file during research and scaffold phases when agentic signals are detected.

---

## Orchestration Patterns

| Pattern | Description | Use When | OKR Compatible |
|---------|-------------|----------|----------------|
| **Orchestrator-worker** | One orchestrator routes tasks to N specialized workers, collects results | Tasks are categorically distinct; each worker is an expert in one domain | Yes |
| **Pipeline** | Agents pass output sequentially — A → B → C | Each stage transforms output from the previous; strict ordering required | Yes |
| **Fan-out / fan-in** | Orchestrator spawns parallel agents for the same task, aggregates results | Tasks are independent and can run concurrently (e.g., multi-source research) | Yes |
| **Router** | A classifier agent inspects input and hands off to the right specialist | Input type is unpredictable at design time; specialists are mutually exclusive | Yes |
| **Swarm / peer-to-peer** | Agents detect handoff signals and route directly to the next appropriate agent; no central coordinator | Requirements emerge dynamically; rigid upfront planning is counterproductive; no single point of failure required | No — no central registry |

Most real systems combine patterns. A router often dispatches to orchestrator-worker sub-graphs.

**Caution — orchestrator-worker "telephone game"**: When an orchestrator paraphrases or summarizes agent outputs before passing them downstream, information fidelity degrades with each hop. Mitigation: pass structured typed fields between agents, not natural language summaries. The orchestrator should route results directly, not reinterpret them.

---

## Pattern Selection Guide

Use this when the user selects "I'm not sure" or when their description is ambiguous.

| If the user needs... | Recommend |
|----------------------|-----------|
| Rigid control, auditability, human oversight at each step | Orchestrator-worker |
| Clear sequential stages where each step transforms the last | Pipeline |
| The same task run against multiple independent sources simultaneously | Fan-out / fan-in |
| Dynamic routing based on unpredictable input type | Router |
| Flexible exploration, emergent requirements, no rigid plan upfront | Swarm / peer-to-peer |

**Swarm vs. orchestrator-worker decision test**: Does the system need a single authority that approves and coordinates every handoff? If yes → orchestrator-worker. If agents can detect task completion and route themselves to the next step → swarm eliminates the central bottleneck and single point of failure.

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

## Failure Resilience

Multi-agent systems fail in ways single-agent systems don't. Document the failure strategy in CLAUDE.md and scaffold for it from the start.

**Circuit breaker** — if an agent fails N consecutive times, route to a backup agent or degrade gracefully rather than blocking the pipeline indefinitely. Implement at the orchestrator or handoff layer, not inside individual agents.

**Exponential backoff** — transient failures (rate limits, network timeouts) should be retried with increasing delays. Agents that call external APIs should wrap calls with retry logic.

**Idempotency** — agents that write to shared state (context object, memory server, database) must be safe to call more than once with the same input. If a retry produces a duplicate write, the system should handle it without corrupting state.

**What to document in CLAUDE.md** — add an "Operational Considerations" section that states: which agents are failure-critical (pipeline blocks if they fail), which are optional (graceful degradation if they fail), and what the retry/circuit-breaker strategy is.

---

## Economic Considerations

Multi-agent systems cost significantly more to run than single-agent systems — each agent invocation consumes its own context window and token budget. In production, this can reach 10–15x the token cost of an equivalent single-agent chat.

**Flag this at scaffold time**: add a comment to `src/orchestrator.py` and a note in CLAUDE.md under "Operational Considerations" so the user knows what they're signing up for before they're surprised by a bill.

**Design implication**: keep agent context windows lean. Each agent should receive only the slice of state it needs (see Agent isolation below), not the full context object. Bloated context windows are the primary cost driver in multi-agent systems.

### Budget-Aware Orchestration

In the orchestrator-worker pattern, the orchestrator can act as a resource manager — tracking remaining token budget across all agent calls and making dispatch decisions based on what's left, just as a project manager allocates budget across a team.

The Anthropic SDK exposes everything needed:

```python
# Before a call — estimate cost without sending
token_estimate = client.messages.count_tokens(
    model="claude-opus-4-6",
    messages=[{"role": "user", "content": agent_prompt}]
).input_tokens

# After a call — record actual spend
response = client.messages.create(...)
budget.record(response.usage)  # usage.input_tokens + usage.output_tokens

# Per-call — cap what an agent can output
client.messages.create(max_tokens=256)
```

Scaffold a `src/budget.py` (or `budget.ts`) utility that the orchestrator imports:

```python
# src/budget.py
from dataclasses import dataclass, field
from anthropic.types import Usage

@dataclass
class TokenBudget:
    total: int
    spent: int = 0

    @property
    def remaining(self) -> int:
        return self.total - self.spent

    @property
    def utilization(self) -> float:
        return self.spent / self.total

    def record(self, usage: Usage) -> None:
        self.spent += usage.input_tokens + usage.output_tokens

    def agent_config(self, priority: str = "normal") -> dict | None:
        """
        Returns kwargs to pass to client.messages.create(), scaled to remaining budget.
        Returns None if this agent should be skipped entirely.
        """
        if self.utilization < 0.5:
            return {"max_tokens": 4096}
        elif self.utilization < 0.8:
            return {"max_tokens": 1024}
        else:
            if priority == "low":
                return None  # skip non-essential agents when budget is tight
            return {"max_tokens": 256}
```

The orchestrator checks `budget.agent_config(priority)` before each dispatch. `None` means skip or degrade; a dict means forward those kwargs to the API call. This keeps budget logic out of individual agents — they just execute with the resources they're given.

**Three scenarios where this pays off**:
1. **Research pipelines** — skip the low-priority synthesis step if 80% of budget is spent and the core output is already done
2. **Retry loops** — check remaining budget before retrying a failed agent; a retry under severe constraint may warrant a simpler prompt
3. **Fan-out scaling** — `n_agents = min(desired, remaining // estimated_cost_per_agent)` instead of always spawning a fixed N

**Honest caveat**: `count_tokens` adds a round-trip API call before every agent invocation. For tight loops, use heuristics (prompt character count × factor) for cheap agents and reserve pre-flight counting for expensive ones.

---

## Key Design Principles

**One agent per file** — clear responsibility boundaries; easy to test, replace, or extend individual agents without touching others.

**Prompts as `.md` files** — version-controlled, reviewable in PRs, editable without touching code. One file per agent role.

**State is explicit** — context object is typed; no hidden globals. Every field has an owner and a consumer.

**Agents are stateless** — all state is passed in, nothing stored on the agent object itself. The same agent can be called with different contexts without side effects.

---

## OKR-Driven Goal Tracking

All hierarchical patterns (orchestrator-worker, pipeline, fan-out/fan-in, router) support OKR tracking. See `references/okr-orchestration.md` for the full guide and orchestrator template.

The `src/okr/` module provides:
- `OKRRegistry` — mission, vision, objectives, key results
- `KRUpdate` — agent progress report (score 0.0–1.0 + narrative)
- `evaluate()` — threshold check with self-correction action
- `format_escalation_report()` — human-readable escalation output

Swarm/P2P patterns do not use OKRs. OKRs require a central authority to set objectives and monitor key results — swarm systems have no such authority by design.
