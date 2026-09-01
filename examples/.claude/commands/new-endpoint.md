---
description: Scaffold a new FastAPI route — router module, registration, and a test.
argument-hint: "[resource-name]"
---

Scaffold a new route for the resource `$1`.

Follow the existing patterns exactly — read `src/api/routes/users.py` and
`tests/api/test_users.py` first.

1. Create `src/api/routes/$1.py` exposing an `APIRouter`. Validate the request body with a
   Pydantic model. On failure, return the standard envelope from `src/lib/errors.py`
   (`error_response(...)`), never a bare `HTTPException` with a string.
2. Register the router in `src/api/main.py` next to the other resources, alphabetically.
3. Create `tests/api/test_$1.py` covering the happy path and a 422 for an invalid body.
   Use the fixtures in `tests/conftest.py` and the factories in `tests/factories.py`.
4. Run `uv run pytest tests/api/test_$1.py` and `uv run mypy src`; fix anything that fails.
5. Summarise what you created and anything the caller still needs to wire up (migrations,
   auth dependency, docs).

Do not commit.
