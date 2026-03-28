---
name: agentic-project-setup
description: >
  Use when setting up, scaffolding, or bootstrapping a multi-agent orchestration project.
  Triggers on: "multi-agent", "multi-agent system", "agent orchestration", "orchestrator", "subagents",
  "agent pipeline", "fan-out agents", "parallel agents", "agent workflow", "build an agent
  that spawns agents", "specialized agents", "swarm", "peer-to-peer agents", "agent handoff",
  "distributed agents", or any variation where the user needs to set up a project involving
  multiple AI agents coordinating with each other.
  Also triggers when a user opens a blank directory and describes a system with multiple
  AI agents working together.
  This skill scaffolds orchestration-ready folder structure, agent role architecture,
  shared state management, MCP servers, plugins, hooks, and CLAUDE.md.
---

# Agentic Project Setup

Your job is to act as an expert agent systems architect helping someone launch a multi-agent orchestration project with the right foundations from day one. You understand orchestration patterns, state management, agent boundaries, and Claude Code's ecosystem. Your recommendations should be specific to their system's agents — not generic.

The process has four phases: interview, research, plan, scaffold.

---

## Phase 1: Interview

Enter plan mode immediately using `EnterPlanMode` — this signals to the user that you're gathering information before acting, which sets the right expectation.

Then use `AskUserQuestion` to learn about their agent system. Load `references/question-templates.md` for exact question wording and options.

**Round 1** (6 questions across two AskUserQuestion calls — max 4 per call):
- **Call 1**: Project description, mission (store as `mission`), vision (store as `vision`), tech stack
- **Call 2**: Orchestration pattern, team context

**Round 2** (conditional — if Round 1 reveals multiple distinct agent roles):
- Agent roles — free text: name each agent and what it's responsible for
- Shared state needs — what information flows between agents, what must be persisted

**Round 3** (conditional — if hard constraints are likely):
- Claude-only vs. multi-model, self-host requirements, compliance constraints, locked-in infrastructure

The goal of the interview is to understand the agent roles well enough to name them specifically in the scaffold — not just use `[agent1]` and `[agent2]` as placeholders.

---

## Phase 2: Research

Research in parallel based on interview signals. Load the research triggers section of `references/question-templates.md` to guide what to look up.

**context7 MCP** (use when specific frameworks or SDKs are mentioned):
- If Claude Agent SDK or Anthropic SDK mentioned: resolve the library ID and fetch current orchestration docs
- If LangChain, LangGraph, CrewAI, AutoGen, or other frameworks mentioned: fetch their current multi-agent API docs
- Focus on the patterns they'll actually use — agent handoffs, state passing, tool definitions

**github MCP** (always search for):
- Multi-agent orchestration starter templates matching their stack (e.g., "claude sdk multi-agent python starter")
- Popular MCP servers relevant to their specific agent roles (e.g., if one agent does web research: "firecrawl mcp server")

Load `references/mcp-catalog.md` to map agent role signals to MCP server recommendations.
Load `references/plugin-catalog.md` to map project signals to plugin recommendations.
Load `references/agent-orchestration-patterns.md` to select and understand the right pattern for their use case. If the user selected "I'm not sure" or chose swarm/peer-to-peer, also use the Pattern Selection Guide in that file to recommend or confirm the right pattern before planning.

---

## Phase 3: Present the Plan

Phase 3 has three steps — do them in order:

**Step 1 — Present the text plan.** The user should understand why you're recommending each piece. Structure it as:

```
## Your Agentic Project Setup Plan

### What I understand about your system
[2-3 sentences naming the specific agents and what they do — use the names they gave you]

### Orchestration Pattern
[Named pattern + 1 sentence on why it fits their specific system]

### Agent Roles
| Name | Responsibility | Key tools / MCPs it will use |
|------|---------------|------------------------------|
| [actual agent name] | [what it does] | [tools] |

### MCP Servers
[Each server: name + why it matters for their specific agents]

### Plugins to Enable
[Each plugin + what it gives them]
| playground | Concept map of the full system (used for pre-scaffold review) |

### Hooks
[Formatter for their stack + prompt file validation + .env protection]

### Project Structure
[The folder layout with their actual agent names filled in]

### Files I'll Create
- CLAUDE.md — agent architecture documented so Claude always knows the system
- .mcp.json — memory + context7 + any agent-specific servers
- .claude/settings.json — hooks and permissions
- .env.example — environment variables (ANTHROPIC_API_KEY + any others)
- src/agents/ stubs for each named agent
- src/prompts/agents/ prompt files for each agent
```

**Step 2 — Offer a concept map.** After presenting the text plan, use `AskUserQuestion` to ask:

> "Before we build this — would you like a concept map of the full system? This shows how you, the orchestrator, agents, OKR framework, tools, and token budget all connect — so you can review the complete operational picture and adjust anything before files are created."
> - Yes — generate the concept map
> - No — the text plan is enough, proceed to approval

If yes, first install the playground plugin if not already present:
```
/plugin install playground@claude-plugins-official
```

Then use the playground plugin to generate a concept map with these layers:

**Human layer**
- `[You]` — entry point showing what the user provides as input and receives as output

**Goal layer (OKR framework)**
- `[Mission]` — why the system exists (from interview)
- `[Vision]` — what success looks like (from interview)
- `[Objectives]` → `[Key Results]` — goal hierarchy driving self-correction

**Orchestration layer**
- `[Orchestrator]` — receives task from user; dispatches to agents; evaluates KR updates; writes `okr-status.json`; connected to Token Budget and OKR Registry

**Agent layer** (one node per named agent)
- Each agent node: role label, tools it uses, which KRs it reports on
- Directed edges: Orchestrator → agent (dispatch) and agent → Orchestrator (KRUpdate)

**Infrastructure layer**
- `[Token Budget]` — tracks total / spent / remaining across all agent dispatches
- `[OKR Registry]` — stores objectives, KRs, and current scores
- `[Tools / MCPs]` — grouped by agent: which MCP servers each agent calls

**Self-correction flow**
- RETRY → REROUTE → DOWNGRADE → ESCALATE as a feedback loop from OKR Registry back to Orchestrator

**Output layer**
- `[Result]` → back to `[You]`
- `[okr-status.json]` — dashboard artifact written each cycle

Present the map and ask if the user wants to adjust agent names, roles, tools, or any flow. Incorporate any feedback before moving to Step 3.

**Step 3 — Get scaffold approval.** Use `ExitPlanMode` to get final approval before touching anything.

---

## Phase 4: Scaffold

After approval, load `references/folder-structures.md` for the directory layout and CLAUDE.md template. Load `references/hooks-patterns.md` for hook configurations.

**Create in this order** (so if something fails, partial state is still useful):

> **OKR pattern note:** Steps 1a–1e apply to all patterns **except Swarm / peer-to-peer**. If the user selected Swarm/P2P, skip steps 1a–1e entirely and omit the OKR Contract section from CLAUDE.md (Step 2). Instead, add this note to CLAUDE.md:
> ```
> ## OKR Tracking
> This project uses the Swarm/P2P pattern. OKR-driven goal tracking is not applied —
> swarm systems have no central authority to set objectives or monitor key results.
> Agents coordinate through shared context signals rather than hierarchical goal assignment.
> ```

1. **Folder structure** — create all directories with their actual agent names filled in (e.g., `src/agents/researcher.py` not `src/agents/[role].py`). Also create `src/okr/` directory (skip for Swarm/P2P).

1a. **src/okr/types.py** (skip for Swarm/P2P):
```python
# src/okr/types.py
from dataclasses import dataclass


@dataclass
class KRUpdate:
    """Progress update an agent returns for a single Key Result."""
    kr_id: str
    score: float   # 0.0–1.0, where 1.0 = fully achieved
    narrative: str # "Found 4 of 20 sources — domain too narrow"
```

1b. **src/okr/registry.py** (skip for Swarm/P2P):
```python
# src/okr/registry.py
from dataclasses import dataclass, field


@dataclass
class KeyResult:
    id: str
    description: str
    unit: str                  # e.g. "sources", "completeness", "accuracy"
    threshold: float = 0.5     # below this triggers self-correction (0.0–1.0)
    current: float = 0.0       # updated by evaluator after each KRUpdate
    narrative: str = ""        # latest agent-provided explanation
    attempts: int = 0          # tracks self-correction cycles


@dataclass
class Objective:
    id: str
    description: str
    key_results: list[KeyResult] = field(default_factory=list)

    def score(self) -> float:
        if not self.key_results:
            return 0.0
        return sum(kr.current for kr in self.key_results) / len(self.key_results)


@dataclass
class OKRRegistry:
    mission: str
    vision: str
    objectives: list[Objective] = field(default_factory=list)

    def summary(self) -> dict:
        return {
            "mission": self.mission,
            "vision": self.vision,
            "objectives": [
                {
                    "id": obj.id,
                    "description": obj.description,
                    "score": round(obj.score(), 2),
                    "key_results": [
                        {
                            "id": kr.id,
                            "description": kr.description,
                            "unit": kr.unit,
                            "current": round(kr.current, 2),
                            "threshold": kr.threshold,
                            "narrative": kr.narrative,
                            "attempts": kr.attempts,
                        }
                        for kr in obj.key_results
                    ],
                }
                for obj in self.objectives
            ],
        }
```

1c. **src/okr/evaluator.py** (skip for Swarm/P2P):
```python
# src/okr/evaluator.py
from enum import Enum

from .registry import KeyResult, OKRRegistry
from .types import KRUpdate


class SelfCorrectionAction(Enum):
    CONTINUE = "continue"
    RETRY = "retry"
    REROUTE = "reroute"
    DOWNGRADE = "downgrade"
    ESCALATE = "escalate"


def process_updates(registry: OKRRegistry, updates: list[KRUpdate]) -> None:
    """Apply agent KRUpdates to the registry."""
    kr_index: dict[str, KeyResult] = {
        kr.id: kr
        for obj in registry.objectives
        for kr in obj.key_results
    }
    for update in updates:
        if update.kr_id in kr_index:
            kr = kr_index[update.kr_id]
            kr.current = update.score
            kr.narrative = update.narrative


def evaluate(registry: OKRRegistry) -> list[tuple[KeyResult, SelfCorrectionAction]]:
    """Return (KeyResult, action) pairs for any KR below its threshold."""
    results = []
    for obj in registry.objectives:
        for kr in obj.key_results:
            if kr.current < kr.threshold:
                action = _decide(kr)
                kr.attempts += 1
                results.append((kr, action))
    return results


def _decide(kr: KeyResult) -> SelfCorrectionAction:
    if kr.attempts == 0:
        return SelfCorrectionAction.RETRY
    if kr.attempts == 1:
        return SelfCorrectionAction.REROUTE
    if kr.attempts == 2:
        kr.threshold = round(kr.threshold * 0.8, 2)  # accept lower standard
        return SelfCorrectionAction.DOWNGRADE
    return SelfCorrectionAction.ESCALATE
```

1d. **src/okr/escalation.py** (skip for Swarm/P2P):
```python
# src/okr/escalation.py
from .registry import KeyResult, OKRRegistry


def format_escalation_report(registry: OKRRegistry, failing_krs: list[KeyResult]) -> str:
    lines = [
        "# OKR Escalation Report",
        "",
        f"**Mission:** {registry.mission}",
        f"**Vision:** {registry.vision}",
        "",
        "## Underperforming Key Results",
        "",
    ]
    for kr in failing_krs:
        lines += [
            f"### {kr.id}",
            f"- **Description:** {kr.description}",
            f"- **Score:** {kr.current:.2f} (threshold: {kr.threshold:.2f})",
            f"- **Unit:** {kr.unit}",
            f"- **Narrative:** {kr.narrative}",
            f"- **Self-correction attempts:** {kr.attempts}",
            "",
        ]
    lines += [
        "## Decision Required",
        "",
        "Autonomous self-correction has been exhausted. Please choose:",
        "1. Provide revised instructions and continue",
        "2. Accept current progress and proceed",
        "3. Abort the run",
        "",
    ]
    return "\n".join(lines)
```

1e. **src/okr/\_\_init\_\_.py** (skip for Swarm/P2P):
```python
# src/okr/__init__.py
from .escalation import format_escalation_report
from .evaluator import SelfCorrectionAction, evaluate, process_updates
from .registry import KeyResult, Objective, OKRRegistry
from .types import KRUpdate

__all__ = [
    "format_escalation_report",
    "SelfCorrectionAction",
    "evaluate",
    "process_updates",
    "KeyResult",
    "Objective",
    "OKRRegistry",
    "KRUpdate",
]
```

2. **CLAUDE.md** — fill the template with: project description, tech stack, agent roles table with actual names, orchestration pattern, state contract, how to add a new agent for their system. Always include an "Operational Considerations" section noting: (a) multi-agent systems have significantly higher token costs than single-agent — flag if this is a production system; (b) the failure handling strategy (which agents are critical vs. optional, retry/circuit-breaker approach). For non-Swarm/P2P patterns, also include the **OKR Contract** section (see `references/folder-structures.md` template).
3. **.mcp.json** — always include memory + context7; add agent-specific servers based on what each agent does
4. **.claude/settings.json** — formatter hook for their stack + prompt file validation + .env protection
5. **.env.example** — ANTHROPIC_API_KEY + any others their stack and agent roles require
6. **.gitignore** — appropriate for their stack + `.env`
7. **src/agents/base.py** (or `base.ts`) — a minimal abstract base class or interface stub that all agents inherit from. `AgentResponse` must include a `kr_updates: list[KRUpdate]` field (with default empty list) so agents can report KR progress alongside their output.
8. **src/agents/[each-named-role].py** — one stub file per agent, inheriting from base, with the agent's responsibility noted in a docstring
9. **src/prompts/agents/[each-named-role].md** — placeholder system prompt per agent with a comment to fill in the agent's role, tone, constraints, and output format
10. **src/budget.py** (or `budget.ts`) — `TokenBudget` dataclass stub: `total`, `spent`, `remaining`, `utilization`, `record(usage)`, `agent_config(priority)`. Include a comment explaining the pattern: the orchestrator imports this and checks `agent_config()` before every agent dispatch.
11. **src/orchestrator.py** (or `.ts`) — OKR-aware stub that: (a) imports `OKRRegistry` and calls `initialize_okrs(mission, vision, task, client)` before any agent dispatch; (b) injects relevant KR IDs into each agent's task instructions via `format_kr_instructions()`; (c) calls `process_updates()` and `evaluate()` after each agent returns; (d) writes `okr-status.json` dashboard after each cycle; (e) handles self-correction actions (RETRY, REROUTE, DOWNGRADE) and raises `SystemExit` on ESCALATE. See `references/okr-orchestration.md` for the full template. For Swarm/P2P, use the standard dispatch-check pattern without OKR integration.
12. **README.md** — brief stub with key commands and architecture overview

After creating each file, briefly note what it does. For agent stubs especially, explain the pattern to follow when implementing the agent's logic — the user is learning the architecture as you build it.

---

## Reference Files

Load these files when you need them — don't load all of them upfront:

| File | When to load |
|------|-------------|
| `references/question-templates.md` | Phase 1 — interview questions and research triggers |
| `references/agent-orchestration-patterns.md` | Phase 2 — selecting and explaining the right pattern |
| `references/mcp-catalog.md` | Phase 2 — matching agent roles to MCP servers |
| `references/plugin-catalog.md` | Phase 2 — matching project signals to plugins |
| `references/hooks-patterns.md` | Phase 4 — writing the hooks config |
| `references/folder-structures.md` | Phase 4 — directory layout and CLAUDE.md template |
| `references/okr-orchestration.md` | Phase 4 — OKR module guide, orchestrator template, KR writing tips |

---

## What Good Looks Like

A great run of this skill:
- CLAUDE.md names actual agent roles (not `[placeholder]`) and explains what each one does
- The orchestrator stub shows the routing pattern clearly — a developer can follow the wiring
- Each agent file has one clear responsibility, not a generic `agent.py` for everything
- Prompt files exist for every agent, even if they're stubs with instructions to fill in
- The user can immediately start implementing agent logic — no structural decisions left to make
- No generic boilerplate — every file reflects their specific system and agent names

A mediocre run:
- Generic `agent.py` with no role differentiation
- CLAUDE.md with `[brackets]` still in it
- Missing prompt files for some agents
- No base class or shared state module
- Orchestrator doesn't show the handoff pattern
