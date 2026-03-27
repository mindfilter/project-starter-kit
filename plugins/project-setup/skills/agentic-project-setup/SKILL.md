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

**Round 1** (always ask these, 4 questions in one call):
1. Project description — what the system does, what problem it solves, who uses it
2. Tech stack — Python, TypeScript, or "help me pick"
3. Orchestration pattern — sequential pipeline, parallel fan-out, router, swarm/peer-to-peer, or "not sure yet"
4. Team context — solo hobby / solo production / small team / larger team

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

Present a structured plan before creating anything. The user should understand why you're recommending each piece — this builds trust and lets them correct misunderstandings before you build the wrong thing.

Structure the plan as:

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

Then use `ExitPlanMode` to get approval before touching anything.

---

## Phase 4: Scaffold

After approval, load `references/folder-structures.md` for the directory layout and CLAUDE.md template. Load `references/hooks-patterns.md` for hook configurations.

**Create in this order** (so if something fails, partial state is still useful):

1. **Folder structure** — create all directories with their actual agent names filled in (e.g., `src/agents/researcher.py` not `src/agents/[role].py`)
2. **CLAUDE.md** — fill the template with: project description, tech stack, agent roles table with actual names, orchestration pattern, state contract, how to add a new agent for their system. Always include an "Operational Considerations" section noting: (a) multi-agent systems have significantly higher token costs than single-agent — flag if this is a production system; (b) the failure handling strategy (which agents are critical vs. optional, retry/circuit-breaker approach).
3. **.mcp.json** — always include memory + context7; add agent-specific servers based on what each agent does
4. **.claude/settings.json** — formatter hook for their stack + prompt file validation + .env protection
5. **.env.example** — ANTHROPIC_API_KEY + any others their stack and agent roles require
6. **.gitignore** — appropriate for their stack + `.env`
7. **src/agents/base.py** (or `base.ts`) — a minimal abstract base class or interface stub that all agents inherit from
8. **src/agents/[each-named-role].py** — one stub file per agent, inheriting from base, with the agent's responsibility noted in a docstring
9. **src/prompts/agents/[each-named-role].md** — placeholder system prompt per agent with a comment to fill in the agent's role, tone, constraints, and output format
10. **src/budget.py** (or `budget.ts`) — `TokenBudget` dataclass stub: `total`, `spent`, `remaining`, `utilization`, `record(usage)`, `agent_config(priority)`. Include a comment explaining the pattern: the orchestrator imports this and checks `agent_config()` before every agent dispatch.
11. **src/orchestrator.py** (or `.ts`) — stub with routing logic that imports each agent and the `TokenBudget`, shows the dispatch-check pattern: `config = budget.agent_config(priority); if config is None: skip`
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
