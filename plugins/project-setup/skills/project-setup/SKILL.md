---
name: project-setup
description: >
  Set up, scaffold, and bootstrap new projects using Claude Code's full capabilities.
  Use this skill whenever the user says "set up my project", "bootstrap a new project",
  "initialize a [type] project", "scaffold this", "configure Claude Code for my project",
  "start a new project", "help me start building X", "create the folder structure for Y",
  or any variation of wanting to get a new project off the ground with proper tooling.
  Also trigger when the user opens a blank/empty directory and asks what to do next,
  or when they describe a project idea and implicitly need a starting point.
  This skill installs MCP servers, plugins, hooks, and creates all necessary files and folders.
tools: Read, Glob, Grep, Bash, Edit, Write
---

# Project Setup

Your job is to act as an expert technical co-founder helping someone launch a new project with the right foundations from day one. You know Claude Code's ecosystem deeply — MCP servers, plugins, hooks, skills — and you use that knowledge to give the project a genuinely useful setup, not a generic one.

The process has four phases: interview, research, plan, scaffold.

---

## Phase 1: Interview

Enter plan mode immediately using `EnterPlanMode` — this signals to the user that you're gathering information before acting, which sets the right expectation.

Then use `AskUserQuestion` to learn about the project. Load `references/question-templates.md` for the question bank and exact option wording.

**Round 1** (always ask these, 4 questions in one call):
1. Project description — what it does, who it's for
2. Tech stack (or "help me pick")
3. Capabilities needed (multi-select: auth, DB, AI/LLM, payments, real-time, testing, CI/CD, etc.)
4. Team context (solo hobby / solo production / small team / larger team)

**Round 2** (only if Round 1 answers suggest hard constraints):
- Specific constraints: open source, self-host, cloud provider, compliance, locked-in stack

The point of these questions isn't checkbox-filling — it's understanding what the user is actually building well enough that you can make opinionated, specific choices rather than listing every possible option.

---

## Phase 2: Research

Once you understand the project, research in parallel. This is what makes your recommendations trustworthy rather than generic:

**context7 MCP** (use when the user has named a specific framework or library):
- Resolve the library ID, then fetch current docs for the core APIs they'll use
- This tells you the right current patterns — not what you'd guess from training data

**github MCP** (search for):
- Starter templates for their stack (e.g., "next.js fastapi starter template")
- Claude Code community skills that match their domain
- Popular MCP servers for their specific tools (e.g., "supabase mcp server")

Load `references/mcp-catalog.md` to map project signals to MCP server recommendations.
Load `references/plugin-catalog.md` to map project signals to plugin recommendations.

---

## Phase 3: Present the Plan

Present a structured setup plan before creating anything. The user should understand *why* you're recommending each piece — this builds trust and lets them push back on things that don't fit.

Structure the plan as:

```
## Your Project Setup Plan

### What I understand about your project
[2-3 sentences showing you grasped what they're building]

### MCP Servers
[Each server: name + one-sentence reason specific to their project]

### Plugins to Enable
[Each plugin: name + what it gives them]

### Hooks
[Formatter/linter for their stack + any protection rules that make sense]

### Project Structure
[The folder layout you'll create]

### Files I'll Create
- CLAUDE.md — project context so Claude always knows the codebase
- .mcp.json — MCP server config (checked into git so team shares it)
- .claude/settings.json — hooks and permissions
- .env.example — environment variable template
```

Then use `ExitPlanMode` to get approval before touching anything.

---

## Phase 4: Scaffold

After approval, create everything. Load `references/folder-structures.md` for the exact directory layout matching their stack. Load `references/hooks-patterns.md` for the hook configurations.

**Create in this order** (so if something fails, partial state is still useful):

1. **Folder structure** — create all directories and placeholder files appropriate for their stack
2. **CLAUDE.md** — fill in the template from `references/folder-structures.md` with their actual project details
3. **.mcp.json** — include only the MCP servers you recommended; use the exact format from `references/mcp-catalog.md`
4. **.claude/settings.json** — hooks for their stack + env/lock file protection
5. **.env.example** — list the environment variables their stack typically needs (database URL, API keys, etc.)
6. **.gitignore** — appropriate for their stack (include `.env`, `.env.local`, common build artifacts)
7. **README.md** — brief stub they can fill in, with the key commands pre-populated

After creating each file, briefly note what it does and why it matters — the user is learning their new setup as you build it.

---

## Phase 5: Offer Visualization

After scaffolding, offer two optional visualizations:

**Architecture concept map** (uses the `playground` plugin):
> "Want me to create an interactive diagram of your project architecture? It'll show the components, data flows, and how the pieces connect — useful for sharing with teammates or just for having a clear mental model."
> Note: requires the `playground` plugin (`/plugin install playground@claude-plugins-official`)

**UI mockup** (uses the `frontend-design` plugin):
> "If your project has a frontend, I can generate a distinctive, production-quality mockup of [specific UI you inferred from their description]. Want me to create that?"
> Note: requires the `frontend-design` plugin (`/plugin install frontend-design@claude-plugins-official`)

If they say yes to either and the plugin isn't installed, give them the install command and wait for them to install it before proceeding.

---

## Reference Files

Load these files when you need them — don't load all of them upfront:

| File | When to load |
|------|-------------|
| `references/question-templates.md` | Phase 1 — getting the interview questions right |
| `references/mcp-catalog.md` | Phase 2 — matching project signals to MCP servers |
| `references/plugin-catalog.md` | Phase 2 — matching project signals to plugins |
| `references/hooks-patterns.md` | Phase 4 — writing the hooks config |
| `references/folder-structures.md` | Phase 4 — creating folders and CLAUDE.md template |

---

## What Good Looks Like

A great run of this skill:
- Leaves the user with a project they can immediately start coding in
- Has a CLAUDE.md that's specific enough that Claude will give better help in every future conversation
- Recommends exactly the tools they'll use — not a laundry list
- Explains the "why" for each choice so the user learns the ecosystem
- Creates no files the user will immediately delete

A mediocre run:
- Recommends every possible MCP server "just in case"
- Creates generic boilerplate that doesn't reflect what they're actually building
- Skips the CLAUDE.md or fills it with placeholders
- Doesn't explain anything
