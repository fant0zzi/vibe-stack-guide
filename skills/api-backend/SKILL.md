---
name: api-backend
description: Scaffold and iterate on a vibe-coded REST or GraphQL API, microservice, webhook handler, or server-side backend using FastAPI + Pydantic + uv + ruff + mypy + pytest (SQLite in dev, Postgres in prod, Alembic for migrations). Use this skill whenever the user says "API", "backend", "REST", "GraphQL", "webhook", "microservice", "server-side", "FastAPI", or describes server logic without a UI. Apply it for the full lifecycle — scaffold, endpoint-by-endpoint development, tests, and deploy — not just the initial project. If the user is building an AI agent / RAG app, route to `ai-agent` instead.
---

# API / Backend Service — Vibe Coding Path

This skill is for helping an experienced developer ship a backend API, mostly with an AI coding agent. The goal is the shortest verifiable path from "empty folder" to "curl works and returns the right thing."

**Default recommendation, no hedging:** FastAPI + Pydantic + `uv` for packaging + `ruff` for lint/format + `mypy` for types + `pytest`. SQLite in dev, Postgres in prod, Alembic for migrations.

## Why this stack (use only when asked)

Python dominates every public agent benchmark for repo-level tasks — [Multi-SWE-bench](https://arxiv.org/html/2504.02605) puts Python at 52% task resolution vs. 22% for Java and single digits for JS/TS. FastAPI has the smallest surface area of the serious Python frameworks: one file = a working API, Pydantic handles validation, docs auto-generate at `/docs`, and `pytest` runs in seconds.

Don't default to Django (admin panel + ORM batteries add surface area the user will inherit and not understand). Don't split into microservices early — start as one FastAPI app and extract later if needed. Don't use Node/Express for backend-heavy work when the user isn't a JS developer — the user gives up the highest agent reliability and can't easily verify output.

## Scaffold workflow

Run in order, verify between steps.

1. `uv init <project>`, `cd <project>`, `uv add fastapi uvicorn pydantic pydantic-settings`, `uv add --dev pytest ruff mypy httpx`.
2. Single-file `main.py` with a `/healthz` endpoint that returns `{"ok": True}`. Add one `pytest` test that hits it with `httpx.AsyncClient`.
3. Verify: `uv run uvicorn main:app --reload` → open `http://127.0.0.1:8000/docs`. Run `uv run pytest -v`.
4. Add DB only when the first persistent endpoint appears: `uv add sqlalchemy alembic psycopg[binary]`, `alembic init migrations`. Wire SQLite for dev via `DATABASE_URL`.
5. Deploy when the first real endpoint passes tests — Railway, Fly.io, or Render. Keep it to one service.

## First prompt to hand the user

```
I'm building an API that [describe what it does in one sentence].

Stack: FastAPI, Pydantic, uv, ruff, mypy, pytest. SQLite for dev,
Postgres for prod. Alembic for migrations.

Apply these stack-specific constraints:

- One endpoint at a time. Write a pytest test for each endpoint before
  moving to the next.
- Every endpoint gets a Pydantic request and response model — no raw
  dicts in signatures.
- No abstract base classes, no service layers until I have 5+ endpoints.
  One file can be a complete working API.
- async def by default, but no async patterns (background tasks,
  websockets, event streams) until I need them.
- No microservices — one FastAPI app.
- Every response includes a curl command to test the endpoint.

Start by creating the project with uv init, adding dependencies with
uv add, and building one health-check endpoint with one test.
```

## Constraints (enforce on every response)

- **One endpoint at a time**, with a `pytest` test for that endpoint before writing the next one.
- **Pydantic models for every request and response shape** — no raw dicts in signatures.
- **No abstract base classes, no "service layers"** until 5+ endpoints exist. A single file can be a complete working API.
- **`async def` by default** — but no background tasks, websockets, or event streams until there's a real need.
- **One FastAPI app, not microservices.** Extract later only when the scaling wall is visible.
- **Every response includes a `curl` (or `httpie`) command** that tests the endpoint end-to-end.
- **Keep files under 300 lines.** When `main.py` gets long, split by router module — not by premature domain abstractions.

## Verification checklist

```bash
uv run uvicorn main:app --reload                    # "Uvicorn running on http://127.0.0.1:8000"
# open http://127.0.0.1:8000/docs                   # Swagger UI works
uv run pytest -v                                    # all tests pass
uv run mypy .                                       # types clean
uv run ruff check . && uv run ruff format --check . # lint / format clean
```

For every new endpoint, end the response with a `curl` line the user can paste.

## Stop signals

- Agent creates 3+ "service" layers before there are 5 endpoints.
- The user has Alembic migration files they can't read.
- Agent introduces Celery, Redis, or a message queue before there are 10 real users.
- Errors reference SQLAlchemy internals the user can't parse.
- Agent generates abstract base classes "for future extensibility."

When these fire, pause and simplify before adding more.

## Escape routes

- **API outgrew vibe-coding capacity:** hire a Python backend specialist — FastAPI is the most readable of the Python web frameworks, so a specialist can land quickly.
- **Heavy background work arrives:** only then add a task queue (e.g., `arq`, RQ, Celery). Not as precaution.
- **The backend is mostly LLM orchestration:** switch to `ai-agent` (Pydantic AI shares the Python toolchain).

## Handing off to other skills

- Frontend consumer app → `web-app` or `mobile-app-store` / `mobile-pwa`.
- Deeper spec work for a big feature → `sdd` skill.
- Multi-agent development process → `pipeline` skill.
- Something feels off → `red-flags` skill.
