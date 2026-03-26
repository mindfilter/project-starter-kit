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
