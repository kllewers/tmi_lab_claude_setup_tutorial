---
paths:
  - "tests/**/*.py"
  - "**/test_*.py"
---

# Testing rules

These load only when Claude is working with a test file, so they cost nothing on other work.

- Run tests via `uv run pytest <path>`; do not invoke `pytest` through a different Python.
- One behaviour per test function. Name it `test_<thing>_<condition>` — e.g.
  `test_create_order_rejects_negative_quantity`.
- Use the FastAPI `TestClient` / `httpx.AsyncClient` fixtures from `tests/conftest.py`.
  Do not construct the app inline.
- Prefer real objects over mocks. Mock only the network boundary and the clock
  (`freezegun` / a `time` fixture).
- Every route test covers: the happy path, one validation failure (422 or 400), and one
  auth failure (401/403) where the route requires auth.
- Build domain objects with the factories in `tests/factories.py`; do not hand-construct
  them inline.
- No snapshot tests for API responses — assert on the specific fields that matter.
