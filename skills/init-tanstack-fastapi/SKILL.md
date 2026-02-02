---
name: init-tanstack-fastapi
description: Initialize a full-stack monorepo with TanStack Start + shadcn/ui frontend (Bun, Biome, tsc) and FastAPI backend (uv, Ruff, ty). Includes hey-api OpenAPI client generation with Zod and TanStack Query, a justfile task runner, and Docker production setup. Use when creating a new full-stack project, scaffolding a monorepo, or setting up TanStack Start with FastAPI.
allowed-tools: Bash Read Write
---

# Initialize TanStack Start + FastAPI Monorepo

Create a production-ready monorepo with a TanStack Start frontend and FastAPI backend.

For detailed configuration file contents, see [references/stack-details.md](references/stack-details.md).

## Prerequisites

Verify these tools are installed before proceeding:

```bash
bun --version      # >= 1.2
uv --version       # >= 0.6
just --version     # >= 1.0
jj --version       # >= 0.25
lefthook --version # >= 1.0
docker --version   # >= 24.0 (optional, for production builds)
```

If any required tool is missing (bun, uv, just, jj, lefthook), stop and inform the user. Docker is optional — warn if missing but continue.

## Step 1: Initialize Version Control

```bash
jj git init --colocate
```

Create `.jj/.gitignore` with content `/*` so git ignores jj metadata.

## Step 2: Scaffold the Frontend

Run the shadcn create command to scaffold a TanStack Start project:

```bash
bunx --bun shadcn@latest create \
  --preset "https://ui.shadcn.com/init?base=base&style=mira&baseColor=neutral&theme=indigo&iconLibrary=lucide&font=geist&menuAccent=subtle&menuColor=default&radius=medium&template=start&rtl=false" \
  --template start \
  frontend
```

After scaffolding completes:

1. Remove the nested git repository created by shadcn (the project uses the root-level jj/git repo):
   ```bash
   rm -rf frontend/.git
   ```

2. Remove ESLint configuration if present (the project uses Biome instead):
   - Delete `eslint.config.js` or `.eslintrc.*` if they exist
   - Remove `eslint`, `@eslint/*`, and `prettier` packages from `package.json` devDependencies
   - Remove any eslint/prettier scripts from `package.json`

3. Install Biome:
   ```bash
   cd frontend && bun add -d @biomejs/biome
   ```

4. Create `frontend/biome.json` with the configuration from stack-details.md section "Biome Configuration".

5. Auto-fix the scaffolded code to conform to Biome rules (formatting, imports):
   ```bash
   cd frontend && bunx biome check --write --unsafe .
   ```

6. Install hey-api and API client dependencies:
   ```bash
   cd frontend && bun add @hey-api/client-fetch zod @tanstack/react-query
   cd frontend && bun add -d @hey-api/openapi-ts vitest jsdom
   ```

7. Create `frontend/openapi-ts.config.ts` with the configuration from stack-details.md section "Hey-API Configuration".

8. Create `frontend/vitest.config.ts` and `frontend/src/__tests__/smoke.test.ts` from stack-details.md section "Frontend Test Configuration".

9. Merge these scripts into `frontend/package.json` (preserve existing scripts from scaffolding):
   ```json
   {
     "scripts": {
       "check": "biome check .",
       "check:fix": "biome check --write .",
       "typecheck": "tsc --noEmit",
       "test": "vitest run",
       "gen:api": "bunx --bun @hey-api/openapi-ts"
     }
   }
   ```

## Step 3: Scaffold the Backend

1. Initialize a Python project with uv (pinned to Python 3.13 to match Docker images):
   ```bash
   uv init backend --app --python 3.13
   ```

2. Add FastAPI and dependencies:
   ```bash
   cd backend && uv add "fastapi[standard]"
   ```

3. Add development dependencies:
   ```bash
   cd backend && uv add --dev ruff ty pytest httpx mkdocs mkdocs-material "mkdocstrings[python]"
   ```

4. Create the backend application structure:
   ```
   backend/
   ├── app/
   │   ├── __init__.py
   │   ├── main.py
   │   └── routers/
   │       ├── __init__.py
   │       └── health.py
   └── scripts/
       └── generate_schema.py
   ```

5. Write `backend/app/main.py` with the starter code from stack-details.md section "FastAPI Main App".

6. Write `backend/app/routers/health.py` with the health check router from stack-details.md section "Health Router".

7. Create empty `backend/app/__init__.py` and `backend/app/routers/__init__.py`.

8. Write `backend/scripts/generate_schema.py` with the schema export script from stack-details.md section "Schema Generation Script".

9. Append the Ruff and ty configuration to `backend/pyproject.toml` from stack-details.md section "Ruff and ty Configuration" (includes pytest config).

10. Create test files from stack-details.md section "Backend Tests":
    - `backend/tests/__init__.py` (empty)
    - `backend/tests/conftest.py`
    - `backend/tests/test_health.py`

11. Create `backend/mkdocs.yml` from stack-details.md section "MkDocs Configuration".

12. Create `backend/docs/index.md` from stack-details.md section "MkDocs Index Page".

13. Create `backend/docs/api-reference.md` from stack-details.md section "MkDocs API Reference".

14. Remove `backend/hello.py` if it was created by `uv init` (the app now lives in `backend/app/`).

## Step 4: Create the Root Justfile

Create `justfile` in the project root with the content from stack-details.md section "Justfile".

Key recipes:
- `set dotenv-load` to read `.env`
- `dev` spawns frontend and backend in parallel
- `dev-frontend` / `dev-backend` for individual services
- `check` runs all linters, formatters, and type checkers
- `gen-api` generates OpenAPI client from static schema
- `gen-api-live` fetches schema from running backend server
- `docs-serve` / `docs-build` for backend API documentation

## Step 5: Create Docker Configuration

1. Create `frontend/Dockerfile` from stack-details.md section "Frontend Dockerfile".
2. Create `backend/Dockerfile` from stack-details.md section "Backend Dockerfile".
3. Create `docker-compose.yml` in the project root from stack-details.md section "Docker Compose".
4. Create `frontend/.dockerignore` and `backend/.dockerignore` from stack-details.md section "Dockerignore Files".

## Step 6: Create Root Configuration Files

1. Create `.env.example` from stack-details.md section "Environment Template".
2. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
3. Create `.gitignore` from stack-details.md section "Gitignore".
4. Create `README.md` from stack-details.md section "Project README".
5. Create `.editorconfig` from stack-details.md section "EditorConfig".
6. Create `.vscode/extensions.json` from stack-details.md section "VS Code Extensions".
7. Create `.vscode/settings.json` from stack-details.md section "VS Code Settings".
8. Create `.github/workflows/ci.yml` from stack-details.md section "GitHub Actions CI".

## Step 7: Install Pre-commit Hooks

1. Create `lefthook.yml` at the project root from stack-details.md section "Lefthook Configuration".

2. Install the hooks:
   ```bash
   lefthook install
   ```

## Step 8: Generate Initial API Client

```bash
cd backend && uv run python scripts/generate_schema.py
cd frontend && bun run gen:api
```

## Step 9: Verify Everything Works

1. Run checks:
   ```bash
   just check
   ```

2. Run tests:
   ```bash
   just test
   ```

3. Run dev servers:
   ```bash
   just dev
   ```
   Verify:
   - Frontend at http://localhost:3000
   - Backend at http://localhost:8000
   - API docs at http://localhost:8000/docs

4. Stop the dev servers after verification.

## Final Project Structure

```
project/
├── frontend/
│   ├── app/                  # TanStack Start app directory
│   ├── public/
│   ├── src/
│   │   ├── __tests__/        # Frontend tests
│   │   └── client/           # Generated API client (gitignored)
│   ├── biome.json
│   ├── openapi-ts.config.ts
│   ├── vitest.config.ts
│   ├── package.json
│   ├── tsconfig.json
│   ├── Dockerfile
│   └── .dockerignore
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   └── routers/
│   │       ├── __init__.py
│   │       └── health.py
│   ├── tests/
│   │   ├── conftest.py
│   │   └── test_health.py
│   ├── scripts/
│   │   └── generate_schema.py
│   ├── docs/
│   │   ├── index.md
│   │   └── api-reference.md
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
├── .env
├── .gitignore
└── README.md
```

Report the final status to the user, including any warnings or errors encountered during setup.
