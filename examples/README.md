# examples/ — a working scaffold

Copy these into a real project's root and `.claude/` directory, then trim to fit. Every
file is referenced from the docs.

```
examples/
├── CLAUDE.md                          # -> your project root as CLAUDE.md
├── .mcp.json                         # -> your project root (delete if no MCP servers)
└── .claude/
    ├── settings.json                 # committed: permissions, hooks, env
    ├── settings.local.json.example   # copy to settings.local.json (gitignored) and edit
    ├── rules/
    │   └── testing.md                # path-scoped rule: only loads for test files
    ├── skills/
    │   └── pr-description/
    │       ├── SKILL.md
    │       └── template.md
    ├── commands/
    │   └── new-endpoint.md           # creates /new-endpoint
    └── agents/
        └── test-runner.md            # a subagent
```

> This scaffold assumes a **Python** project — a FastAPI service managed with
> [uv](https://docs.astral.sh/uv/), linted/formatted with Ruff, tested with pytest,
> type-checked with mypy. If you don't use uv, drop the `uv run` prefix and activate your
> venv. If you use different tools, swap the command names (`ruff` → `flake8` + `black`,
> `pytest` → `python -m pytest`, `mypy` → `pyright`, …). The *shape* is what matters, not
> the exact commands.

## Why `settings.json` has no comments

Claude Code settings are **strict JSON** — a `//` comment or trailing comma is a syntax
error. The explanation for each key is in
[../docs/02-project-scaffolding.md](../docs/02-project-scaffolding.md) and
[../docs/04-permissions-and-safety.md](../docs/04-permissions-and-safety.md). Keep the
`$schema` line: it gives you autocomplete and validation in VS Code.

## Applying it

```bash
# from your project root
cp path/to/this/examples/CLAUDE.md ./CLAUDE.md
cp -r path/to/this/examples/.claude ./.claude
cp ./.claude/settings.local.json.example ./.claude/settings.local.json   # then edit
# review every file, delete what you don't need, commit the rest
```

`.claude/settings.local.json` and `CLAUDE.local.md` should **not** be committed — see the
root [.gitignore](../.gitignore).
