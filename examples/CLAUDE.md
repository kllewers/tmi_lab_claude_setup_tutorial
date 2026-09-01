# Project: <name>

<!-- Maintainer note: keep this file under ~200 lines. It loads into context every
     session. If a section becomes a procedure, move it to .claude/skills/. If it only
     applies to part of the tree, move it to .claude/rules/ with a paths: filter.
     HTML comments like this one are stripped before Claude sees the file. -->

One-sentence description of what this service does.

## Commands

We use [uv](https://docs.astral.sh/uv/) to manage the environment. If you don't have it,
`pipx install uv` or see the uv docs; without uv, drop the `uv run` prefix and activate
the venv yourself.

- Install: `uv sync`
- Dev server: `uv run uvicorn src.api.main:app --reload`
- Test: `uv run pytest` — single file: `uv run pytest tests/api/test_orders.py`
- Lint: `uv run ruff check .` — autofix: `uv run ruff check --fix .`
- Format: `uv run ruff format .`
- Typecheck: `uv run mypy src`

Run `uv run ruff check . && uv run mypy src && uv run pytest` before considering a change done.

## Layout

- `src/api/routes/` — one module per resource, each exposes an `APIRouter`
- `src/api/main.py` — app construction and router registration
- `src/api/deps.py` — shared FastAPI dependencies (auth, db session, pagination)
- `src/lib/` — shared helpers (`errors.py`, `ratelimit.py`, `db.py`)
- `src/domain/` — business logic; no FastAPI imports in here
- `src/_generated/` — generated API client; see gotchas
- `tests/` — `tests/api/test_*.py` for route tests, `tests/factories.py` for fixtures

## Conventions

- 4-space indent; formatting is Ruff's job (a hook runs `ruff format` after each edit).
- Absolute imports from `src` (`from src.lib.errors import error_response`). No wildcard imports.
- All API error responses go through `error_response()` in `src/lib/errors.py`. Never raise
  a bare `HTTPException` with a string detail.
- Database access goes through the session helpers in `src/lib/db.py`; do not create an
  engine or import the driver directly in a route or domain module.
- Type hints on every function signature; `mypy` runs in CI and must pass.
- A new route needs a test covering the happy path and at least one 4xx case.

## Gotchas

- `uv run pytest` needs a local Postgres on `:5432` — run `docker compose up -d db` first.
- `src/_generated/` is generated from the OpenAPI spec. Never edit it by hand; regenerate
  with `uv run python -m scripts.gen_client`.
- The `LEGACY_*` env vars feed the old billing path in `src/domain/billing/v1/`. Leave that
  package alone unless the task is explicitly about it.

## Workflow

- Use plan mode for anything touching `src/domain/` or the database schema.
- Branch from `main`; do not commit or push without being asked.
