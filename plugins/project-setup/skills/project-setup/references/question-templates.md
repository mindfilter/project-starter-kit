# Interview Question Templates

Reference: load this file when interviewing the user about their project.

Use the AskUserQuestion tool for each group. Run no more than 4 questions per call.

## Round 1 — Core Intent

```
Question 1: "Describe your project — what will it do and who is it for?"
(free text / Other)

Question 2: "What's your tech stack?"
Options:
- JavaScript / TypeScript (Node, React, Next.js, Vue, Svelte)
- Python (FastAPI, Django, Flask, scripts)
- Go / Rust / Java / other compiled language
- I don't know yet — help me pick
- Other (free text)

Question 3: "Which capabilities does your project need?" (multi-select)
Options:
- User authentication
- Database / persistent storage
- External API integrations
- AI / LLM features (Claude, OpenAI, etc.)
- Real-time (websockets, SSE)
- File uploads / storage
- Payments (Stripe, etc.)
- Email / notifications
- Background jobs / queues
- Testing (unit, integration, e2e)
- CI/CD pipeline
- Docker / containerization

Question 4: "What's your context?"
Options:
- Solo developer, personal/hobby project
- Solo developer, production app
- Small team (2-5), startup
- Team (5+), established company
```

## Round 2 — Constraints (ask only if answers from Round 1 suggest need)

```
Question: "Any hard constraints?"
Options:
- Must be open source
- Must self-host (no SaaS dependencies)
- Specific cloud provider required (AWS / GCP / Azure)
- Compliance needed (HIPAA, SOC2, GDPR)
- Language/framework already decided — don't suggest alternatives
- Other (free text)
```

## Research Triggers

After interview, these signals should trigger research:

| User mentioned | Research action |
|----------------|-----------------|
| A specific framework (Next.js, FastAPI, etc.) | Use context7 to fetch current docs |
| "AI" or "LLM" or "Claude" | Search GitHub for Claude agent starter templates |
| A database (Postgres, MongoDB, etc.) | Look up relevant MCP server |
| "payments" | Look up Stripe MCP + context7 Stripe docs |
| "real-time" | context7 docs for websocket/SSE library they're using |
| "testing" | Look up testing framework MCP/hooks for their stack |
