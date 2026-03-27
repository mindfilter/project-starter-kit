# Interview Question Templates

Reference: load this file when interviewing the user about their multi-agent system.

Use the AskUserQuestion tool for each group. Run no more than 4 questions per call.

## Round 1 — Core Intent

Round 1 has six questions — split across two AskUserQuestion calls (max 4 per call).

**Call 1a (first AskUserQuestion):**
```
Question 1: "Describe your multi-agent system — what will it do and what problem does it solve?"
(free text / Other)

Question 1b: "What is the mission of this agent system — why does it exist? (one sentence)"
(free text — e.g., "Help researchers find and synthesize academic literature faster")
Store answer as: mission

Question 1c: "What does success look like when this system is working perfectly?"
(free text — e.g., "A researcher gets a fully cited synthesis in under 2 minutes with no manual search")
Store answer as: vision

Question 2: "What's your tech stack?"
Options:
- Python (Claude Agent SDK, LangChain, or custom)
- TypeScript / Node.js (Anthropic SDK, LangChain.js, or custom)
- I don't know yet — help me pick
- Other (free text)
```

**Call 1b (second AskUserQuestion):**
```
Question 3: "Which orchestration pattern fits your system?"
Options:
- Orchestrator-worker (one orchestrator routes tasks to N specialized agents)
- Pipeline (agents process in sequence, each passing output to the next)
- Fan-out / fan-in (orchestrator spawns parallel agents, collects and merges results)
- Router (classifier agent routes each request to the right specialist)
- Swarm / peer-to-peer (agents hand off directly to each other; no central coordinator)
- I'm not sure — describe my use case and help me pick

Question 4: "What's your context?"
Options:
- Solo developer, personal/experimental project
- Solo developer, production system
- Small team (2-5), startup
- Team (5+), established company
```

## Round 2 — Agent Details

Ask if Round 1 answers reveal multiple distinct agent roles.

```
Question: "What are the specialized agent roles in your system?"
(free text)
Prompt them: "List the agents (e.g., 'researcher, summarizer, writer') and what each one does."

Question: "What state or context needs to be shared between agents?"
Options:
- Conversation history / memory
- Structured task state (progress, partial results)
- User profile / preferences
- External data (documents, search results)
- No shared state needed — agents are fully independent
```

## Round 3 — Constraints

Ask only if answers suggest hard requirements.

```
Question: "Any hard constraints?"
Options:
- Must use Claude API only (no other LLMs)
- Must self-host (no external API calls for LLM)
- Specific cloud provider required
- Compliance requirements (HIPAA, SOC2, GDPR)
- Existing codebase to integrate with
- Other (free text)
```

## Research Triggers

After the interview, these signals should trigger research:

| User mentioned | Research action |
|----------------|-----------------|
| Claude Agent SDK | Use context7 to fetch current SDK docs for orchestration patterns |
| Anthropic SDK | Use context7 to fetch claude messages API docs |
| "pipeline" / "fan-out" / "parallel" | Load agent-orchestration-patterns.md workflow patterns section |
| "swarm" / "peer-to-peer" / "handoff" / "no central coordinator" | Load swarm pattern and Pattern Selection Guide from agent-orchestration-patterns.md |
| "I'm not sure" (pattern question) | Load Pattern Selection Guide from agent-orchestration-patterns.md and use it to recommend a pattern based on their Round 1 description |
| A specific framework (LangChain, CrewAI, etc.) | Use context7 to fetch current framework docs |
| "memory" / "cross-session" / "persistent" | Add MCP memory server to recommendations |
| "web research" / "search" | Add brave-search MCP for research-agent role |
| "GitHub" / "PR" / "code review" | Add github MCP for automation agent workflows |
| Production system (solo or team) | Note multi-agent token cost in CLAUDE.md "Operational Considerations" section |
