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

- **Frontend**: TanStack Start + shadcn/ui (Mira style, Indigo theme) + Biome + tsc
- **Backend**: FastAPI + Ruff + ty
- **API Client**: hey-api with Zod validation + TanStack Query hooks
- **Task Runner**: justfile with `dev`, `check`, `fix`, `gen-api` commands
- **Production**: Docker multi-stage builds + docker-compose

## Generated Project Structure

```
project/
├── frontend/              # TanStack Start + shadcn/ui
│   ├── app/               # Routes and pages
│   ├── src/client/        # Generated API client
│   ├── biome.json         # Lint + format config
│   ├── openapi-ts.config.ts
│   ├── Dockerfile
│   └── package.json
├── backend/               # FastAPI + uv
│   ├── app/
│   │   ├── main.py
│   │   └── routers/
│   ├── scripts/
│   │   └── generate_schema.py
│   ├── pyproject.toml
│   ├── Dockerfile
│   └── uv.lock
├── justfile               # Task runner
├── docker-compose.yml     # Production deployment
├── .env.example
├── .gitignore
└── README.md
```

## Prerequisites

The following tools must be installed before running the skill:

- [Bun](https://bun.sh) >= 1.2
- [uv](https://docs.astral.sh/uv/) >= 0.6
- [just](https://github.com/casey/just) >= 1.0
- [Docker](https://www.docker.com/) >= 24.0

## Tooling Overview

| Concern    | Frontend         | Backend      |
| ---------- | ---------------- | ------------ |
| Runtime    | Bun              | Python 3.12  |
| Package    | bun              | uv           |
| Lint       | Biome            | Ruff         |
| Format     | Biome            | Ruff         |
| Typecheck  | tsc              | ty           |
| Framework  | TanStack Start   | FastAPI      |
| API Client | hey-api + Zod + TanStack Query | OpenAPI auto-generation |

## License

MIT
