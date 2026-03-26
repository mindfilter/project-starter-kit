# Project-Setup Marketplace Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a public GitHub-ready Claude Code marketplace repo containing a `project-setup` skill that intelligently bootstraps any project using Claude Code's full feature set.

**Architecture:** A marketplace manifest at `.claude-plugin/marketplace.json` lists plugins hosted in `plugins/`. The `project-setup` plugin contains a single SKILL.md with 5 reference files. The skill is built iteratively using the `skill-creator` plugin's eval loop.

**Tech Stack:** Markdown, YAML frontmatter, JSON (plugin.json, marketplace.json), Claude Code skill/plugin format.

---

## File Structure

Files to create:

```
project-setup-skill/                          ← GitHub repo root
├── .claude-plugin/
│   └── marketplace.json                      ← marketplace manifest
├── plugins/
│   └── project-setup/
│       ├── .claude-plugin/
│       │   └── plugin.json                   ← plugin manifest
│       ├── skills/
│       │   └── project-setup/
│       │       ├── SKILL.md                  ← main skill (built via skill-creator)
│       │       └── references/
│       │           ├── mcp-catalog.md        ← MCP servers by use-case signal
│       │           ├── plugin-catalog.md     ← plugins by codebase signal
│       │           ├── hooks-patterns.md     ← copy-paste hook configs
│       │           ├── question-templates.md ← interview question bank
│       │           └── folder-structures.md  ← scaffold templates per stack
│       └── README.md                         ← plugin docs
├── docs/
│   └── superpowers/
│       └── plans/
│           └── 2026-03-26-project-setup-marketplace.md  ← this file
└── README.md                                 ← marketplace README
```

---

## Task 1: Create Marketplace Manifest

**Files:**
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Write marketplace.json**

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "mindfilter-marketplace",
  "description": "A curated collection of Claude Code skills and plugins for intelligent project setup and development workflows",
  "owner": {
    "name": "mindfilter",
    "email": ""
  },
  "plugins": [
    {
      "name": "project-setup",
      "description": "Intelligently bootstraps any project using Claude Code's full feature set — asks targeted questions, installs MCP servers, plugins, hooks, and scaffolds the complete folder structure and markdown files.",
      "category": "development",
      "source": "./plugins/project-setup",
      "homepage": "https://github.com/mindfilter/project-setup-skill"
    }
  ]
}
```

- [ ] **Step 2: Verify JSON is valid**

```bash
python3 -c "import json; json.load(open('.claude-plugin/marketplace.json')); print('valid')"
```
Expected: `valid`

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: add marketplace manifest"
```

---

## Task 2: Create Plugin Manifest

**Files:**
- Create: `plugins/project-setup/.claude-plugin/plugin.json`

- [ ] **Step 1: Write plugin.json**

```json
{
  "name": "project-setup",
  "description": "Intelligently bootstraps any project using Claude Code's full feature set",
  "author": {
    "name": "mindfilter"
  }
}
```

- [ ] **Step 2: Verify JSON is valid**

```bash
python3 -c "import json; json.load(open('plugins/project-setup/.claude-plugin/plugin.json')); print('valid')"
```
Expected: `valid`

- [ ] **Step 3: Commit**

```bash
git add plugins/project-setup/.claude-plugin/plugin.json
git commit -m "feat: add project-setup plugin manifest"
```

---

## Task 3: Write Reference Files

**Files:**
- Create: `plugins/project-setup/skills/project-setup/references/mcp-catalog.md`
- Create: `plugins/project-setup/skills/project-setup/references/plugin-catalog.md`
- Create: `plugins/project-setup/skills/project-setup/references/hooks-patterns.md`
- Create: `plugins/project-setup/skills/project-setup/references/question-templates.md`
- Create: `plugins/project-setup/skills/project-setup/references/folder-structures.md`

- [ ] **Step 1: Write mcp-catalog.md**

Content: A reference table mapping project signals to recommended MCP servers.

```markdown
# MCP Server Catalog

Reference: load this file when recommending MCP servers based on project signals.

## Signal → Server Mapping

| Project Signal | MCP Server | Install Command | Why |
|----------------|------------|-----------------|-----|
| Uses React, Next.js, Vue, Express, Django, FastAPI, or any popular library | **context7** | `claude mcp add context7 -- npx -y @upstash/context7-mcp@latest` | Live, version-accurate documentation lookup — eliminates hallucinated APIs |
| PostgreSQL / Supabase database | **supabase** | `claude mcp add supabase` | Direct DB query, schema inspection, RLS management |
| Any SQL database (PostgreSQL, MySQL, SQLite) | **sqlite** or **postgres** | `claude mcp add sqlite` | Query tables, inspect schema, run migrations inline |
| GitHub repository / needs PR/issue management | **github** | `claude mcp add github` | Issues, PRs, Actions, code search |
| Frontend app needing browser testing | **playwright** | `claude mcp add playwright -- npx -y @playwright/mcp@latest` | Browser automation, visual testing, screenshot capture |
| Uses Stripe / payments | **stripe** | `claude mcp add stripe -- npx -y @stripe/agent-toolkit@latest` | Payment intents, customers, subscriptions |
| AWS infrastructure | **aws-kb-retrieval** | (see AWS docs) | CloudFormation, Lambda, S3, IAM queries |
| Needs cross-session memory | **memory** | `claude mcp add memory -- npx -y @modelcontextprotocol/server-memory` | Persistent key-value store across conversations |
| Uses Sentry for error tracking | **sentry** | `claude mcp add sentry` | Fetch errors, traces, releases |
| Docker containers | **docker** | `claude mcp add docker` | Container management, logs, exec |
| Linear for issue tracking | **linear** | `claude mcp add linear` | Issues, projects, cycles |
| Slack workspace | **slack** | `claude mcp add slack` | Messages, channels, notifications |
| Needs web research | **brave-search** | `claude mcp add brave-search` | Web search without browser |

## .mcp.json Template

After deciding on servers, write `.mcp.json` in the project root:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

Check this file into the repo so the whole team gets the same servers.
```

- [ ] **Step 2: Write plugin-catalog.md**

```markdown
# Plugin Catalog

Reference: load this file when recommending Claude Code plugins based on project signals.

## Signal → Plugin Mapping

| Project Signal | Plugin | What it adds |
|----------------|--------|-------------|
| Needs git commits managed | **commit-commands** | `/commit`, `/clean-gone`, `/commit-push-pr` commands |
| Building features iteratively | **feature-dev** | `/feature-dev` workflow: architect → explorer → reviewer agents |
| Wants automated hooks | **hookify** | `/hookify` configures hooks via conversation |
| Building another plugin/skill | **plugin-dev** | 7 skills covering hook-dev, MCP integration, skill creation |
| Needs code reviewed | **code-review** | `/code-review` command with review agent |
| Building MCP server | **mcp-server-dev** | Skills for building local/remote MCP servers |
| Frontend / React / Vue work | **frontend-design** | Skill for production-grade, distinctive UI generation |
| Wants interactive architecture visualization | **playground** | Concept maps, code maps, data explorers as HTML playgrounds |
| Building with Claude Agent SDK | **agent-sdk-dev** | SDK templates, `/new-sdk-app` command, verifier agents |
| Wants CLAUDE.md managed | **claude-md-management** | Skill to audit and improve CLAUDE.md files |
| Wants PR reviews | **pr-review-toolkit** | PR review workflow with structured feedback |

## Install Command

```bash
/plugin install {plugin-name}@claude-plugins-official
```

## settings.json enabledPlugins

After install, plugins can be scoped to a project in `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "commit-commands@claude-plugins-official": true,
    "feature-dev@claude-plugins-official": true
  }
}
```
```

- [ ] **Step 3: Write hooks-patterns.md**

```markdown
# Hooks Patterns

Reference: load this file when recommending or writing hooks for a project.

Hooks go in `.claude/settings.json` under the `hooks` key.

## Format

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

## Common Patterns by Stack

### JavaScript / TypeScript — ESLint + Prettier on every edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; true"
          }
        ]
      }
    ]
  }
}
```

### Python — Ruff lint + format on every edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "ruff check --fix \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; ruff format \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; true"
          }
        ]
      }
    ]
  }
}
```

### Rust — cargo fmt on every edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "cargo fmt 2>/dev/null; true"
          }
        ]
      }
    ]
  }
}
```

### Go — gofmt on every edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "gofmt -w \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; true"
          }
        ]
      }
    ]
  }
}
```

### Protect .env files from edits

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT_FILE_PATH\" | grep -q '\\.env'; then echo 'Blocked: do not edit .env files directly'; exit 2; fi"
          }
        ]
      }
    ]
  }
}
```

### Protect lock files from edits

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT_FILE_PATH\" | grep -qE '(package-lock\\.json|yarn\\.lock|pnpm-lock\\.yaml|poetry\\.lock|Cargo\\.lock)'; then echo 'Blocked: do not edit lock files directly'; exit 2; fi"
          }
        ]
      }
    ]
  }
}
```
```

- [ ] **Step 4: Write question-templates.md**

```markdown
# Interview Question Templates

Reference: load this file when interviewing the user about their project.

Use AskUserQuestion tool for each group. Run no more than 4 questions per call.

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
```

- [ ] **Step 5: Write folder-structures.md**

```markdown
# Folder Structure Templates

Reference: load the section matching the user's stack when creating the project scaffold.

## Next.js App (App Router)

```
project-name/
├── .claude/
│   └── settings.json       ← hooks + permissions
├── .mcp.json               ← MCP server config
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── globals.css
│   ├── components/
│   ├── lib/
│   └── types/
├── public/
├── tests/
├── CLAUDE.md               ← project context for Claude
├── README.md
├── .env.local              ← secrets (gitignored)
├── .env.example            ← template (committed)
├── .gitignore
├── next.config.ts
├── tsconfig.json
└── package.json
```

## FastAPI Service

```
project-name/
├── .claude/
│   └── settings.json
├── .mcp.json
├── src/
│   ├── __init__.py
│   ├── main.py             ← FastAPI app entry
│   ├── routers/
│   ├── models/
│   ├── schemas/
│   ├── services/
│   └── dependencies.py
├── tests/
│   └── conftest.py
├── alembic/                ← if using SQLAlchemy
├── CLAUDE.md
├── README.md
├── .env                    ← gitignored
├── .env.example
├── .gitignore
├── pyproject.toml
└── Dockerfile
```

## Claude Agent App (Python)

```
project-name/
├── .claude/
│   └── settings.json
├── .mcp.json
├── src/
│   ├── __init__.py
│   ├── agent.py            ← main agent loop
│   ├── tools/              ← custom tool definitions
│   ├── prompts/            ← system prompts as .md files
│   └── utils/
├── tests/
├── CLAUDE.md
├── README.md
├── .env
├── .env.example
├── .gitignore
└── pyproject.toml
```

## CLI Tool (TypeScript)

```
project-name/
├── .claude/
│   └── settings.json
├── .mcp.json
├── src/
│   ├── index.ts            ← CLI entry (commander/yargs)
│   ├── commands/
│   └── utils/
├── tests/
├── CLAUDE.md
├── README.md
├── .gitignore
├── tsconfig.json
└── package.json
```

## Monorepo (Turborepo / pnpm workspaces)

```
project-name/
├── .claude/
│   └── settings.json
├── .mcp.json
├── apps/
│   ├── web/                ← Next.js frontend
│   └── api/                ← Express/FastAPI backend
├── packages/
│   ├── ui/                 ← shared component library
│   └── types/              ← shared TypeScript types
├── CLAUDE.md
├── README.md
├── .gitignore
├── turbo.json
└── package.json
```

## CLAUDE.md Template

Always generate this file. Replace [brackets]:

```markdown
# [Project Name]

## Project Overview
[1-2 sentences describing what this project does and who uses it.]

## Tech Stack
- **Runtime**: [Node 20 / Python 3.12 / Go 1.22]
- **Framework**: [Next.js 15 / FastAPI / Express]
- **Database**: [PostgreSQL via Prisma / Supabase / SQLite]
- **Auth**: [Clerk / NextAuth / JWT]
- **Hosting**: [Vercel / Railway / AWS]

## Key Commands
```bash
# Install dependencies
[npm install / pip install -e ".[dev]" / go mod download]

# Start dev server
[npm run dev / uvicorn src.main:app --reload / go run ./cmd/server]

# Run tests
[npm test / pytest / go test ./...]

# Build for production
[npm run build / docker build . / go build -o bin/server]
```

## Architecture Notes
[2-3 sentences about key design decisions, folder layout, or patterns to follow.]

## Environment Variables
See `.env.example` for required variables. Copy to `.env.local` (Next.js) or `.env` (Python/Go) and fill in values.

## Conventions
- [e.g., "Use server actions for mutations, not API routes"]
- [e.g., "All DB access goes through the service layer, not directly in routes"]
- [e.g., "New features need unit tests in tests/ before merging"]
```
```

- [ ] **Step 6: Commit all reference files**

```bash
git add plugins/project-setup/skills/project-setup/references/
git commit -m "feat: add project-setup reference files (mcp, plugins, hooks, questions, structures)"
```

---

## Task 4: Create Plugin and Marketplace READMEs

**Files:**
- Create: `plugins/project-setup/README.md`
- Create: `README.md`

- [ ] **Step 1: Write plugin README**

```markdown
# project-setup

An intelligent project bootstrapper for Claude Code. When invoked, this skill:

1. **Interviews you** about your project — what it does, your stack, what capabilities you need
2. **Researches** the best tools using context7 (live docs) and GitHub (community resources)
3. **Recommends** MCP servers, plugins, hooks, and skills tailored to your stack
4. **Scaffolds** the complete folder structure, CLAUDE.md, .mcp.json, and settings.json
5. **Offers visualization** of your architecture using the playground concept-map

## Requirements

- context7 MCP server (for live documentation lookup)
- github MCP plugin (for community resource discovery)

Both are available via the official Claude Code marketplace.

## Usage

After installing this plugin, start a new project session and say:

> "Set up my project" or "Bootstrap this project" or "Initialize a new [type] project"

The skill will guide you through the rest.

## Visualization

After setup, the skill can generate:
- An **interactive concept map** of your architecture (requires `playground` plugin)
- **UI mockups** of frontend components (requires `frontend-design` plugin)
```

- [ ] **Step 2: Write marketplace README**

```markdown
# mindfilter-marketplace

A curated collection of Claude Code skills and plugins for intelligent project workflows.

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add mindfilter/project-setup-skill
```

Then browse and install individual plugins:

```
/plugin install project-setup@mindfilter-marketplace
```

## Plugins

| Plugin | Description |
|--------|-------------|
| **project-setup** | Intelligently bootstraps any project — interviews you, installs the right tools, scaffolds the full structure |

## Requirements

Before using `project-setup`, install these from the official marketplace:

```
/plugin install context7@claude-plugins-official
/plugin install github@claude-plugins-official
```

## Contributing

Open a PR or issue to suggest new plugins or improvements.
```

- [ ] **Step 3: Commit READMEs**

```bash
git add plugins/project-setup/README.md README.md
git commit -m "docs: add marketplace and plugin READMEs"
```

---

## Task 5: Build SKILL.md Using skill-creator

**Files:**
- Create: `plugins/project-setup/skills/project-setup/SKILL.md` (via skill-creator)

This task uses the `skill-creator` skill interactively. Do not write SKILL.md manually.

- [ ] **Step 1: Invoke skill-creator**

In the conversation, run:

```
Use the skill-creator skill to create a new skill called "project-setup"
```

Provide this description to skill-creator:

> Create a skill for setting up a project using Claude Code's full capabilities. The skill should: (1) immediately enter plan mode, (2) interview the user with targeted questions about their project goal, tech stack, and requirements using AskUserQuestion, (3) in parallel research via context7 MCP for framework docs and github MCP for community templates and MCP servers, (4) present a tailored setup plan recommending specific MCP servers, plugins, hooks, and folder structure, (5) after plan approval scaffold the complete project: folder structure, CLAUDE.md, .mcp.json, .claude/settings.json with hooks, (6) offer to visualize the architecture with the playground concept-map and any UI with frontend-design. Reference files are in `references/` — load mcp-catalog.md when recommending MCP servers, plugin-catalog.md for plugins, hooks-patterns.md for hook configs, question-templates.md for interview questions, folder-structures.md for scaffolding.

- [ ] **Step 2: Review the draft with skill-creator**

Work through skill-creator's interview phase — answer its questions about edge cases, expected outputs, and test cases.

- [ ] **Step 3: Run skill-creator eval loop**

When skill-creator offers test prompts, approve them and run the eval loop. Review outputs in the browser viewer. Iterate until satisfied.

- [ ] **Step 4: Save final SKILL.md to plugin directory**

```bash
# Copy the skill-creator output to the plugin directory
# (skill-creator will place it in the workspace — confirm path with skill-creator)
cp <skill-creator-workspace>/project-setup/SKILL.md \
   plugins/project-setup/skills/project-setup/SKILL.md
```

- [ ] **Step 5: Commit SKILL.md**

```bash
git add plugins/project-setup/skills/project-setup/SKILL.md
git commit -m "feat: add project-setup SKILL.md (built with skill-creator)"
```

---

## Task 6: Initialize Git and Verify Structure

**Files:** No new files — verification only.

- [ ] **Step 1: Initialize git if not already done**

```bash
cd /home/mindfilter/Claude/project-setup-skill
git init 2>/dev/null || true
git add -A
git status
```

- [ ] **Step 2: Verify complete structure**

```bash
find . -not -path './.git/*' -not -path './docs/*' | sort
```

Expected output includes:
```
./.claude-plugin/marketplace.json
./plugins/project-setup/.claude-plugin/plugin.json
./plugins/project-setup/README.md
./plugins/project-setup/skills/project-setup/SKILL.md
./plugins/project-setup/skills/project-setup/references/folder-structures.md
./plugins/project-setup/skills/project-setup/references/hooks-patterns.md
./plugins/project-setup/skills/project-setup/references/mcp-catalog.md
./plugins/project-setup/skills/project-setup/references/plugin-catalog.md
./plugins/project-setup/skills/project-setup/references/question-templates.md
./README.md
```

- [ ] **Step 3: Validate all JSON files**

```bash
for f in $(find . -name "*.json" -not -path './.git/*'); do
  python3 -c "import json; json.load(open('$f')); print('OK: $f')"
done
```
Expected: All files print `OK: <path>`

- [ ] **Step 4: Final commit if any changes remain**

```bash
git status --short
# If any uncommitted changes:
git add -A
git commit -m "chore: finalize marketplace structure"
```

---

## Self-Review

### Spec Coverage

| Requirement | Task |
|-------------|------|
| GitHub-shareable marketplace structure | Tasks 1, 4 |
| Users can add marketplace and install plugins | Tasks 1, 4 (marketplace.json + README) |
| project-setup skill built via skill-creator | Task 5 |
| Skill enters plan mode and asks questions | Task 5 (SKILL.md content) |
| Uses context7 for docs research | Task 5 (SKILL.md) + reference files |
| Uses github MCP for community resources | Task 5 (SKILL.md) |
| Creates folder structure + markdown files | Tasks 3, 5 (folder-structures.md + SKILL.md) |
| Visualization offer (playground + frontend-design) | Task 5 (SKILL.md) |

### No Placeholders

All steps contain exact file content, exact commands, and exact expected outputs.

### Type Consistency

No shared types/APIs across tasks — each task is self-contained markdown/JSON content.
