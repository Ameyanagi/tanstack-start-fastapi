# Stack Details Reference

All configuration file contents for the init-tanstack-fastapi skill.

---

## Biome Configuration

`frontend/biome.json`:

```json
{
  "$schema": "https://biomejs.dev/schemas/2.3.13/schema.json",
  "assist": { "actions": { "source": { "organizeImports": "on" } } },
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true,
    "defaultBranch": "main"
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedImports": "warn",
        "useExhaustiveDependencies": "warn"
      },
      "style": {
        "noNonNullAssertion": "off"
      },
      "a11y": {
        "noLabelWithoutControl": "off",
        "useSemanticElements": "off",
        "useKeyWithClickEvents": "off",
        "noRedundantAlt": "off"
      },
      "suspicious": {
        "noArrayIndexKey": "off"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "semicolons": "asNeeded"
    }
  },
  "css": {
    "parser": {
      "tailwindDirectives": true
    }
  },
  "files": {
    "includes": [
      "**",
      "!**/node_modules",
      "!**/.output",
      "!**/.tanstack",
      "!**/dist",
      "!**/src/client",
      "!**/src/routeTree.gen.ts"
    ]
  }
}
```

---

## Hey-API Configuration

`frontend/openapi-ts.config.ts`:

```typescript
import { defineConfig } from '@hey-api/openapi-ts'

export default defineConfig({
  input: '../backend/openapi.json',
  output: {
    path: 'src/client',
  },
  plugins: [
    '@hey-api/typescript',
    {
      name: '@hey-api/sdk',
      validator: true,
    },
    {
      name: 'zod',
      definitions: true,
      responses: true,
      requests: true,
    },
    {
      name: '@tanstack/react-query',
      queryOptions: true,
      mutationOptions: true,
    },
  ],
})
```

After creating the config, ensure these peer dependencies are installed:

```bash
cd frontend && bun add zod @tanstack/react-query @hey-api/client-fetch
```

---

## FastAPI Main App

`backend/app/main.py`:

```python
import os

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel

from app.routers import health

app = FastAPI(
    title="Backend API",
    version="0.1.0",
    docs_url="/docs",
    redoc_url="/redoc",
)

origins = [
    origin.strip()
    for origin in os.getenv("CORS_ORIGINS", "http://localhost:3000").split(",")
    if origin.strip()
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(health.router)


class RootResponse(BaseModel):
    message: str


@app.get("/", response_model=RootResponse)
async def root():
    return RootResponse(message="API is running")
```

---

## Health Router

`backend/app/routers/health.py`:

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter(prefix="/health", tags=["health"])


class HealthResponse(BaseModel):
    status: str


@router.get("", response_model=HealthResponse)
async def health_check():
    return HealthResponse(status="healthy")
```

`backend/app/__init__.py` and `backend/app/routers/__init__.py` should be empty files.

---

## Schema Generation Script

`backend/scripts/generate_schema.py`:

```python
#!/usr/bin/env python3
"""Export the OpenAPI schema from the FastAPI app to a JSON file."""

import json
import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).resolve().parent.parent))

from app.main import app

OUTPUT_PATH = Path(__file__).resolve().parent.parent / "openapi.json"


def main():
    schema = app.openapi()
    OUTPUT_PATH.write_text(json.dumps(schema, indent=2) + "\n")
    print(f"OpenAPI schema written to {OUTPUT_PATH}")


if __name__ == "__main__":
    main()
```

---

## Ruff and ty Configuration

Append to `backend/pyproject.toml` after the existing content:

```toml
[tool.ruff]
line-length = 100
target-version = "py313"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM", "ASYNC", "FAST"]
ignore = ["E501"]

[tool.ruff.lint.flake8-bugbear]
extend-immutable-calls = [
    "fastapi.Depends",
    "fastapi.Query",
    "fastapi.Path",
    "fastapi.Body",
    "fastapi.Header",
    "fastapi.Cookie",
    "fastapi.Form",
    "fastapi.File",
]

[tool.ruff.lint.flake8-type-checking]
runtime-evaluated-base-classes = ["pydantic.BaseModel"]

[tool.ruff.lint.isort]
known-first-party = ["app"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.ty.rules]
unresolved-import = "warn"
# Suppressed: ty incorrectly flags CORSMiddleware keyword arguments.
# Re-evaluate when ty stabilizes. https://github.com/astral-sh/ty/issues
invalid-argument-type = "warn"
```

Also append pytest configuration:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
```

---

## Backend Tests

`backend/tests/__init__.py`: empty file.

`backend/tests/conftest.py`:

```python
import pytest
from fastapi.testclient import TestClient

from app.main import app


@pytest.fixture
def client():
    return TestClient(app)
```

`backend/tests/test_health.py`:

```python
def test_health_check(client):
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "healthy"}


def test_root(client):
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "API is running"}
```

---

## Frontend Test Configuration

`frontend/vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
  },
})
```

`frontend/src/__tests__/smoke.test.ts`:

```typescript
import { describe, expect, it } from 'vitest'

describe('smoke test', () => {
  it('should pass', () => {
    expect(true).toBe(true)
  })
})
```

---

## Justfile

`justfile` (project root):

```justfile
set dotenv-load

# Default: show available commands
default:
    @just --list

# Run frontend and backend dev servers in parallel
dev:
    just dev-backend & just dev-frontend & wait

# Run frontend dev server
dev-frontend:
    cd frontend && bun run dev

# Run backend dev server
dev-backend:
    cd backend && uv run fastapi dev app/main.py --port ${BACKEND_PORT:-8000}

# Run all checks: lint, format, typecheck
check: check-frontend check-backend check-openapi

# Check frontend: biome + tsc
check-frontend:
    cd frontend && bun run check
    cd frontend && bun run typecheck

# Check backend: ruff + ty
check-backend:
    cd backend && uv run ruff check .
    cd backend && uv run ruff format --check .
    cd backend && uv run ty check

# Validate OpenAPI schema is up to date
check-openapi:
    #!/usr/bin/env bash
    set -euo pipefail
    cd backend
    temp=$(mktemp)
    trap 'rm -f "$temp"' EXIT
    uv run python -c "import json,sys;sys.path.insert(0,'.');from app.main import app;print(json.dumps(app.openapi(),indent=2))" > "$temp"
    if ! diff -q openapi.json "$temp" > /dev/null 2>&1; then
        echo "ERROR: openapi.json is out of date. Run 'just gen-api' to update."
        exit 1
    fi
    echo "OpenAPI schema is up to date."

# Fix lint and format issues
fix: fix-frontend fix-backend

# Fix frontend issues
fix-frontend:
    cd frontend && bun run check:fix

# Fix backend issues
fix-backend:
    cd backend && uv run ruff check --fix .
    cd backend && uv run ruff format .

# Generate OpenAPI client from static schema
gen-api:
    cd backend && uv run python scripts/generate_schema.py
    cd frontend && bun run gen:api

# Generate OpenAPI client from running backend server
gen-api-live:
    cd frontend && bunx --bun @hey-api/openapi-ts -i http://localhost:${BACKEND_PORT:-8000}/openapi.json -o src/client

# Run all tests
test: test-frontend test-backend

# Run frontend tests
test-frontend:
    cd frontend && bun run test

# Run backend tests
test-backend:
    cd backend && uv run pytest

# Serve backend API docs locally
docs-serve:
    cd backend && uv run mkdocs serve

# Build backend API docs
docs-build:
    cd backend && uv run mkdocs build

# Build Docker images
docker-build:
    docker compose build

# Run production stack with Docker
docker-up:
    docker compose up -d

# Stop production stack
docker-down:
    docker compose down

# Clean generated files
clean:
    rm -rf frontend/.output frontend/.tanstack frontend/node_modules frontend/src/client
    rm -rf backend/.venv backend/__pycache__ backend/openapi.json backend/site
```

---

## Frontend Dockerfile

`frontend/Dockerfile`:

```dockerfile
FROM oven/bun:1 AS base
WORKDIR /app

# Install dependencies
FROM base AS deps
COPY package.json bun.lock* ./
RUN bun install --frozen-lockfile

# Build the application
FROM deps AS build
COPY . .
RUN bun run build

# Production image (slim over alpine to avoid musl libc issues)
FROM oven/bun:1-slim AS runtime
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/.output ./.output
COPY --from=build /app/package.json ./package.json
ENV NODE_ENV=production
EXPOSE 3000
CMD ["bun", ".output/server/index.mjs"]
```

---

## Backend Dockerfile

`backend/Dockerfile`:

```dockerfile
FROM ghcr.io/astral-sh/uv:python3.13-bookworm-slim AS builder

ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy

WORKDIR /app

# Install dependencies first (cached layer)
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --no-install-project --no-dev

# Copy application code and sync
COPY . .
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --frozen --no-dev

# Runtime image
FROM python:3.13-slim-bookworm AS runtime

WORKDIR /app

RUN groupadd --system app && useradd --system --gid app app

COPY --from=builder --chown=app:app /app/.venv /app/.venv
COPY --from=builder --chown=app:app /app/app /app/app

ENV PATH="/app/.venv/bin:$PATH" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

USER app
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Docker Compose

`docker-compose.yml`:

```yaml
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "${FRONTEND_PORT:-3000}:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      backend:
        condition: service_healthy
    restart: unless-stopped

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "${BACKEND_PORT:-8000}:8000"
    env_file:
      - .env
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    restart: unless-stopped
```

---

## Dockerignore Files

`frontend/.dockerignore`:

```
node_modules
.output
.tanstack
.git
*.md
biome.json
openapi-ts.config.ts
```

`backend/.dockerignore`:

```
.venv
__pycache__
.git
*.md
.ruff_cache
openapi.json
scripts/
```

---

## Environment Template

`.env.example`:

```bash
# Frontend
FRONTEND_PORT=3000

# Backend
BACKEND_PORT=8000

# CORS (comma-separated origins)
CORS_ORIGINS=http://localhost:3000

# Add your application-specific variables below
# DATABASE_URL=postgresql://user:password@localhost:5432/dbname
# SECRET_KEY=change-me-in-production
```

---

## Gitignore

`.gitignore`:

```gitignore
# Dependencies
node_modules/
.venv/

# Build output
.output/
.tanstack/
dist/
__pycache__/
*.pyc

# Generated
frontend/src/client/
backend/openapi.json

# Environment
.env
.env.local
.env.*.local

# IDE
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Caches
.ruff_cache/
.ty/

# MkDocs
site/

# Lefthook
.lefthook-local.yml
```

---

## Project README

`README.md` (in the generated project root):

````markdown
# Project Name

Full-stack monorepo: TanStack Start + shadcn/ui (frontend) and FastAPI (backend).

## Prerequisites

- [Bun](https://bun.sh) >= 1.2
- [uv](https://docs.astral.sh/uv/) >= 0.6
- [just](https://github.com/casey/just) >= 1.0
- [jj](https://jj-vcs.github.io/jj/) >= 0.25 (colocated with git)
- [Lefthook](https://github.com/evilmartians/lefthook) >= 1.0
- [Docker](https://www.docker.com/) >= 24.0 (optional, for production builds)

## Quick Start

```bash
cp .env.example .env
just dev
```

- Frontend: http://localhost:3000
- Backend: http://localhost:8000
- API Docs: http://localhost:8000/docs

## Commands

```bash
just dev              # Run frontend + backend
just dev-frontend     # Frontend only
just dev-backend      # Backend only
just check            # Lint + format + typecheck + OpenAPI validation
just test             # Run frontend + backend tests
just fix              # Auto-fix lint and format issues
just gen-api          # Generate API client from static schema
just gen-api-live     # Generate API client from running server
just docs-serve       # Serve backend API docs locally
just docs-build       # Build backend API docs
just docker-build     # Build Docker images
just docker-up        # Start production stack
just docker-down      # Stop production stack
just clean            # Remove generated files
```

## Stack

| Layer    | Technology                                         |
| -------- | -------------------------------------------------- |
| Frontend | TanStack Start, React, shadcn/ui (Mira), Tailwind |
| Backend  | FastAPI, Python 3.13, Pydantic                     |
| API      | hey-api + TanStack Query + Zod                     |
| Lint     | Biome (frontend), Ruff (backend)                   |
| Types    | tsc (frontend), ty (backend)                       |
| Package  | Bun (frontend), uv (backend)                       |
| Tasks    | just                                               |
| VCS      | jj (colocated with git)                            |
| Hooks    | Lefthook (parallel pre-commit)                     |
| Docs     | MkDocs Material (backend API)                      |
| CI       | GitHub Actions                                     |
| Deploy   | Docker + docker compose                            |

## Project Structure

```
├── frontend/          # TanStack Start + shadcn/ui
├── backend/           # FastAPI + uv
├── justfile           # Task runner
├── docker-compose.yml # Production deployment
├── .env.example       # Environment template
└── .gitignore
```

## API Client Generation

The frontend API client is auto-generated from the FastAPI OpenAPI schema.

**Static** (no running server needed):
```bash
just gen-api
```

**Live** (from running backend):
```bash
just dev-backend &
just gen-api-live
```

Generated code lands in `frontend/src/client/` and includes TypeScript types, Zod schemas, and TanStack Query hooks.
````

---

## Lefthook Configuration

`lefthook.yml` (project root):

```yaml
pre-commit:
  parallel: true
  commands:
    biome-check:
      root: "frontend/"
      glob: "**/*.{ts,tsx,js,jsx,json,css}"
      run: bunx biome check --no-errors-on-unmatched {staged_files}
      stage_fixed: true
    tsc:
      root: "frontend/"
      glob: "**/*.{ts,tsx}"
      run: bun run typecheck
    ruff-check:
      root: "backend/"
      glob: "**/*.py"
      run: uv run ruff check {staged_files}
    ruff-format:
      root: "backend/"
      glob: "**/*.py"
      run: uv run ruff format --check {staged_files}
    ty-check:
      root: "backend/"
      glob: "**/*.py"
      run: uv run ty check
```

Note: `tsc` and `ty` run on the full project (not staged files) since type checkers need full context. Biome and Ruff use `{staged_files}` for speed. All commands run in parallel.

---

## MkDocs Configuration

`backend/mkdocs.yml`:

```yaml
site_name: Backend API Documentation
site_description: FastAPI backend API reference and guides
theme:
  name: material
  palette:
    - scheme: default
      primary: indigo
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - scheme: slate
      primary: indigo
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
  features:
    - content.code.copy
    - navigation.sections
plugins:
  - search
  - mkdocstrings:
      handlers:
        python:
          paths: [.]
          options:
            show_source: true
            show_root_heading: true
nav:
  - Home: index.md
  - API Reference: api-reference.md
```

---

## MkDocs Index Page

`backend/docs/index.md`:

```markdown
# Backend API

FastAPI backend for the project.

## Quick Start

```bash
just dev-backend
```

API will be available at:

- **API**: http://localhost:8000
- **Interactive docs**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

## Project Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py          # FastAPI application
│   └── routers/
│       ├── __init__.py
│       └── health.py    # Health check endpoint
├── tests/
│   ├── conftest.py      # Test fixtures
│   └── test_health.py   # Health endpoint tests
├── scripts/
│   └── generate_schema.py
├── docs/                # This documentation
├── mkdocs.yml
└── pyproject.toml
```
```

---

## MkDocs API Reference

`backend/docs/api-reference.md`:

```markdown
# API Reference

::: app.main
    options:
      show_root_heading: true
      members_order: source

::: app.routers.health
    options:
      show_root_heading: true
      members_order: source
```

---

## EditorConfig

`.editorconfig` (project root):

```ini
root = true

[*]
indent_style = space
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
charset = utf-8

[*.{js,ts,tsx,jsx,json,css,yml,yaml}]
indent_size = 2

[*.py]
indent_size = 4

[*.md]
trim_trailing_whitespace = false

[justfile]
indent_size = 4
```

---

## VS Code Extensions

`.vscode/extensions.json`:

```json
{
  "recommendations": [
    "biomejs.biome",
    "charliermarsh.ruff",
    "ms-python.python",
    "ms-python.vscode-pylance",
    "bradlc.vscode-tailwindcss",
    "martinvoelk.jj"
  ]
}
```

---

## VS Code Settings

`.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "biomejs.biome",
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff"
  },
  "[markdown]": {
    "editor.defaultFormatter": null
  },
  "typescript.tsdk": "frontend/node_modules/typescript/lib",
  "biome.lspBin": "frontend/node_modules/@biomejs/biome/bin/biome"
}
```

---

## GitHub Actions CI

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  frontend:
    name: Frontend
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: frontend
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - run: bun install --frozen-lockfile
      - run: bun run check
      - run: bun run typecheck
      - run: bun run test

  backend:
    name: Backend
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: backend
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync --frozen
      - run: uv run ruff check .
      - run: uv run ruff format --check .
      - run: uv run ty check
      - run: uv run pytest
```
