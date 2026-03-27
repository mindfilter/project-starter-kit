# OKR-Driven Orchestration

## When to Use OKRs

OKR tracking is scaffolded for all hierarchical patterns (orchestrator-worker, pipeline, fan-out/fan-in, router). It is excluded for swarm/P2P, which has no central authority.

Use OKRs whenever:
- The system has a mission/vision that should govern agent behavior
- You want the orchestrator to self-correct rather than silently produce subpar output
- You want a real-time status dashboard of operation progress
- You want a human-readable escalation report when things go wrong

## Writing Good Key Results

Good KRs are measurable on a 0.0–1.0 scale. Each should have a clear unit.

**Good KRs:**
- "Find at least 20 peer-reviewed sources" → unit: `sources`, threshold: 0.6
- "Output covers 90% of required topics" → unit: `topic_coverage`, threshold: 0.7
- "Response is grammatically correct" → unit: `quality`, threshold: 0.9

**Bad KRs (avoid):**
- "Do a good job" — not measurable
- "Complete the task" — binary, not progressive
- "Be accurate" — no unit or scale anchor

## Threshold Guidance

- `0.9+` — high-stakes KRs (accuracy, correctness, completeness)
- `0.6–0.8` — normal KRs (source count, coverage breadth)
- `0.4–0.6` — exploratory KRs where partial success is acceptable

Lower thresholds = more tolerance before self-correction triggers.

## Self-Correction Philosophy

The orchestrator self-corrects before escalating to the human. The progression is:

1. **Retry** — same agent, adjusted instructions (add context about what was missing)
2. **Re-route** — different agent or different approach
3. **Downgrade** — reduce threshold by 20%, accept lower standard, continue
4. **Escalate** — print escalation report, halt run, request human decision

This keeps the human focused on goals and outcomes, not implementation details.

## Orchestrator Template

The full OKR-aware orchestrator template. Substitute `[first_agent]`, `[FirstAgent]`, etc. with actual agent names from the interview:

```python
# src/orchestrator.py
import json

import anthropic

from src.agents.[first_agent] import [FirstAgent]
from src.agents.[second_agent] import [SecondAgent]
from src.budget import TokenBudget
from src.okr import (
    KRUpdate,
    KeyResult,
    Objective,
    OKRRegistry,
    SelfCorrectionAction,
    evaluate,
    format_escalation_report,
    process_updates,
)
from src.state.context import RunContext


def initialize_okrs(mission: str, vision: str, task: str, client: anthropic.Anthropic) -> OKRRegistry:
    """Use the LLM to derive objectives and key results from mission/vision/task."""
    response = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=1024,
        messages=[
            {
                "role": "user",
                "content": f"""You are designing OKRs for an AI agent system.

Mission: {mission}
Vision: {vision}
Current task: {task}

Define 1-3 objectives and 2-4 key results per objective. Key results must be measurable on a 0.0-1.0 scale.

Return ONLY valid JSON:
{{
  "objectives": [
    {{
      "id": "obj-short-id",
      "description": "Objective description",
      "key_results": [
        {{
          "id": "kr-short-id",
          "description": "Measurable outcome description",
          "unit": "sources|completeness|accuracy|coverage|etc",
          "threshold": 0.6
        }}
      ]
    }}
  ]
}}""",
            }
        ],
    )
    data = json.loads(response.content[0].text)
    registry = OKRRegistry(mission=mission, vision=vision)
    for obj_data in data["objectives"]:
        obj = Objective(id=obj_data["id"], description=obj_data["description"])
        for kr_data in obj_data["key_results"]:
            obj.key_results.append(
                KeyResult(
                    id=kr_data["id"],
                    description=kr_data["description"],
                    unit=kr_data["unit"],
                    threshold=kr_data.get("threshold", 0.5),
                )
            )
        registry.objectives.append(obj)
    return registry


def format_kr_instructions(kr_ids: list[str], registry: OKRRegistry) -> str:
    """Build the KR measurement block to append to agent instructions."""
    kr_index = {
        kr.id: kr
        for obj in registry.objectives
        for kr in obj.key_results
    }
    lines = ["\n\nYou will be measured on the following Key Results.", "Return a KRUpdate for each:", ""]
    for kr_id in kr_ids:
        if kr_id in kr_index:
            kr = kr_index[kr_id]
            lines.append(f"- {kr.id} ({kr.unit}, score 0.0-1.0): {kr.description}")
    lines += [
        "",
        "Append to your response as JSON:",
        '[{"kr_id": "...", "score": 0.0, "narrative": "..."}]',
    ]
    return "\n".join(lines)


def write_dashboard(registry: OKRRegistry, log_path: str = "okr-status.json") -> None:
    """Write current OKR state to a status file for human monitoring."""
    import json as _json
    with open(log_path, "w") as f:
        _json.dump(registry.summary(), f, indent=2)


def _parse_kr_updates(agent_output: str) -> list[dict]:
    """Extract KRUpdate dicts from agent output. Expects JSON array at end of output."""
    import re
    match = re.search(r"\[.*\]", agent_output, re.DOTALL)
    if match:
        try:
            return json.loads(match.group())
        except json.JSONDecodeError:
            return []
    return []


class Orchestrator:
    def __init__(self, budget: TokenBudget, mission: str, vision: str):
        self.budget = budget
        self.mission = mission
        self.vision = vision
        self.client = anthropic.Anthropic()
        self.[first_agent] = [FirstAgent]()
        self.[second_agent] = [SecondAgent]()

    def run(self, task: str) -> str:
        ctx = RunContext(task=task)

        # Initialize OKRs from mission/vision/task
        registry = initialize_okrs(self.mission, self.vision, task, self.client)
        write_dashboard(registry)

        # --- Agent dispatch (repeat this block per agent, adjust kr_ids per role) ---
        config = self.budget.agent_config(priority="normal")
        if config is not None:
            kr_ids = ["kr-short-id"]  # KR IDs relevant to this agent's role
            instructions = ctx.task + format_kr_instructions(kr_ids, registry)
            response = self.[first_agent].run(task=instructions, **config)
            self.budget.record(response.usage)

            # Parse KRUpdates from agent output and evaluate
            updates = [KRUpdate(**u) for u in _parse_kr_updates(response.output)]
            process_updates(registry, updates)
            actions = evaluate(registry)
            write_dashboard(registry)

            for kr, action in actions:
                if action == SelfCorrectionAction.RETRY:
                    print(f"[OKR] Retrying for {kr.id} (score {kr.current:.2f} < {kr.threshold:.2f})")
                    # Re-dispatch with adjusted instructions — add context about what was missing
                elif action == SelfCorrectionAction.REROUTE:
                    print(f"[OKR] Re-routing {kr.id} to alternate agent")
                    # Route to a different agent
                elif action == SelfCorrectionAction.DOWNGRADE:
                    print(f"[OKR] Downgrading threshold for {kr.id} to {kr.threshold:.2f}")
                elif action == SelfCorrectionAction.ESCALATE:
                    report = format_escalation_report(registry, [kr])
                    print(report)
                    raise SystemExit("Human intervention required. See escalation report above.")
        # --- End agent dispatch block ---

        return ctx.draft  # replace with your actual output field
```

## Dashboard

`okr-status.json` is updated after every agent cycle. It contains the full OKR registry summary — mission, vision, all objectives with scores, all KRs with current scores and narratives.

Read it at any time:
```bash
cat okr-status.json | python -m json.tool
```

## Swarm/P2P Exclusion

OKRs require a central authority to set objectives and monitor key results. Swarm/P2P systems are explicitly decentralized — there is no orchestrator to hold the registry. Applying OKRs to a swarm would require designating a goal-keeper agent, which effectively converts the swarm into an orchestrator-worker pattern. For this reason, OKRs are excluded from swarm scaffolding.
