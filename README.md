# init-tanstack-fastapi

An [Agent Skill](https://agentskills.io) that initializes a full-stack monorepo with **TanStack Start + shadcn/ui** (frontend) and **FastAPI** (backend).

## Install

```bash
npx skills add <owner>/tanstack-start-fastapi
```

Or install for a specific agent:

```bash
npx skills add <owner>/tanstack-start-fastapi -a claude-code
```

## Usage

Once installed, invoke the skill:

```
/init-tanstack-fastapi
```

The skill will scaffold a complete monorepo with:

- **Frontend**: TanStack Start + shadcn/ui (Mira style, Indigo theme) + Biome + tsc + vitest
- **Backend**: FastAPI + Ruff + ty + pytest + MkDocs
- **API Client**: hey-api with Zod validation + TanStack Query hooks
- **Task Runner**: justfile with `dev`, `check`, `fix`, `test`, `gen-api`, `docs-serve`, `clean` commands
- **VCS**: jj (Jujutsu) colocated with git
- **Hooks**: Lefthook parallel pre-commit (Biome, tsc, Ruff, ty)
- **CI**: GitHub Actions (lint, typecheck, test)
- **Production**: Docker multi-stage builds + docker-compose

## Generated Project Structure

```
project/
├── frontend/
│   ├── app/                  # TanStack Start routes and pages
│   ├── src/
│   │   ├── __tests__/        # Frontend tests (vitest)
│   │   └── client/           # Generated API client (gitignored)
│   ├── biome.json
│   ├── openapi-ts.config.ts
│   ├── vitest.config.ts
│   ├── package.json
│   ├── Dockerfile
│   └── .dockerignore
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   └── routers/
│   ├── tests/
│   ├── scripts/
│   │   └── generate_schema.py
│   ├── docs/                 # MkDocs source
│   ├── mkdocs.yml
│   ├── pyproject.toml
│   ├── uv.lock
│   ├── Dockerfile
│   └── .dockerignore
├── .github/
│   └── workflows/
│       └── ci.yml
├── .vscode/
│   ├── extensions.json
│   └── settings.json
├── justfile
├── lefthook.yml
├── docker-compose.yml
├── .editorconfig
├── .env.example
├── .gitignore
└── README.md
```

## Prerequisites

The following tools must be installed before running the skill:

- [Bun](https://bun.sh) >= 1.2
- [uv](https://docs.astral.sh/uv/) >= 0.6
- [just](https://github.com/casey/just) >= 1.0
- [jj](https://jj-vcs.github.io/jj/) >= 0.25
- [Lefthook](https://github.com/evilmartians/lefthook) >= 1.0
- [Docker](https://www.docker.com/) >= 24.0 (optional)

## Tooling Overview

| Concern   | Frontend                       | Backend                 |
| --------- | ------------------------------ | ----------------------- |
| Runtime   | Bun                            | Python 3.13             |
| Package   | bun                            | uv                      |
| Framework | TanStack Start                 | FastAPI                 |
| Lint      | Biome                          | Ruff                    |
| Format    | Biome                          | Ruff                    |
| Typecheck | tsc                            | ty                      |
| Test      | vitest                         | pytest                  |
| API       | hey-api + Zod + TanStack Query | OpenAPI auto-generation |
| VCS       | jj (colocated with git)        |                         |
| Hooks     | Lefthook (parallel pre-commit) |                         |
| CI        | GitHub Actions                 |                         |
| Docs      |                                | MkDocs Material         |
| Deploy    | Docker + docker compose        |                         |

## License

MIT
