# 2. Project scaffolding — the `.claude/` directory and `CLAUDE.md`

This is the per-project setup. The goal: a new teammate (or a fresh session) becomes
productive without you re-explaining the project, and without loading a novel into context
every time.

A fully scaffolded project looks like this:

```
your-project/
├── CLAUDE.md                     # standing instructions, loaded every session
├── .mcp.json                    # MCP servers this project uses (optional)
└── .claude/
    ├── settings.json            # shared, committed: permissions, hooks, env
    ├── settings.local.json      # personal, gitignored: your overrides
    ├── rules/                   # optional: topic- or path-scoped instructions
    │   └── testing.md
    ├── skills/                  # procedures & reference knowledge, loaded on demand
    │   └── pr-description/
    │       └── SKILL.md
    ├── commands/                # slash commands (a simpler form of skill)
    │   └── new-endpoint.md
    └── agents/                  # subagents with their own context
        └── test-runner.md
```

Working copies of every one of these files are in [../examples/](../examples/).

---

## 2.1 `CLAUDE.md` — standing instructions

`CLAUDE.md` at the repo root (or `.claude/CLAUDE.md`) is loaded into context **at the
start of every session**. It is the single most important file in your setup, and the
easiest to get wrong by making it too big.

### What belongs in it

Facts that are true in *every* session and that Claude can't reliably infer from the code:

- Build / test / lint / typecheck commands (the exact invocations)
- High-level architecture and where things live (`API routes in src/api/routes/`)
- Conventions that differ from tool defaults (import style, error format, naming)
- "Always do X / never do Y" rules
- Gotchas that have bitten you before

### What does *not* belong in it

- **Multi-step procedures** → make a [skill](03-skills-commands-subagents.md).
- **Instructions that only matter for one part of the tree** → a path-scoped rule (below).
- **Anything Claude can derive by reading the code** — directory dumps, dependency lists,
  a restatement of your framework's docs. This is pure context tax.
- **Things that must happen deterministically** ("run the formatter after every edit") → a
  [hook](04-permissions-and-safety.md).

### How to write it

- **Target under 200 lines.** Longer files consume more context *and* measurably reduce
  how well Claude follows them.
- Use markdown headers and bullets. Claude scans structure like a person does.
- Be concrete and verifiable: "Use 4-space indentation" beats "format code properly";
  "Run `uv run pytest` before committing" beats "test your changes."
- Remove contradictions. If two lines disagree, Claude picks one arbitrarily.

### Getting started and maintaining it

```
/init        # generates a starting CLAUDE.md from your codebase.
             # If one exists, it suggests improvements instead of overwriting.
/context     # after a session starts: confirm CLAUDE.md appears under "Memory files"
/memory      # browse/open all memory files (CLAUDE.md, rules, auto-memory)
```

Add to `CLAUDE.md` when: Claude makes the same mistake twice, a review catches something
it should have known, or you type the same correction you typed last session.

### Related files

| File | Scope | Committed? |
| --- | --- | --- |
| `CLAUDE.md` / `.claude/CLAUDE.md` | This project, everyone | Yes |
| `~/.claude/CLAUDE.md` | You, every project | n/a (personal) |
| `CLAUDE.local.md` | This project, just you | **No — gitignore it** |
| `.claude/rules/*.md` | This project; optionally path-scoped | Yes |

### `@` imports and path-scoped rules

`CLAUDE.md` can pull in other files with `@path/to/file` — useful for sharing content with
other agents (e.g. `@AGENTS.md`). Note: **imported files still load at launch**, so this
helps organization, not context size.

To load instructions *only when relevant*, use `.claude/rules/` with `paths:` frontmatter:

```markdown
---
paths:
  - "src/api/**/*.py"
---

# API rules
- Every endpoint validates its input with a Pydantic model.
- Use the standard error envelope from `src/lib/errors.py`.
```

That rule enters context only once Claude touches a matching file. Rules *without* a
`paths:` field load every session, same as `CLAUDE.md`.

---

## 2.2 `settings.json` — how Claude Code behaves

Settings are strict JSON (no comments, no trailing commas). Add the `$schema` line for
autocomplete and validation in VS Code.

### The four locations

| Scope | File | Applies to | Commit? |
| --- | --- | --- | --- |
| **User** | `~/.claude/settings.json` | You, every project | personal |
| **Shared project** | `.claude/settings.json` | Everyone who clones the repo | **yes** |
| **Project local** | `.claude/settings.local.json` | You, this project only | **no** (auto-gitignored) |
| **Managed** | OS-level / MDM / console | Everyone the org deploys to; wins over everything | org-controlled |

Higher in that list wins when the same key is set twice — **except list-valued keys like
`permissions.allow`, which merge across all files** so each scope can add entries.

`.claude/settings.local.json` is where Claude Code saves the approvals you give at
permission prompts ("Yes, and don't ask again"). The first time it writes that file in a
git repo it adds it to your global git excludes, so you don't have to gitignore it
yourself unless you created it by hand.

### What to put in the committed `settings.json`

- **`permissions.allow`** — your safe, frequent commands so teammates aren't prompted for
  `uv run pytest` fifty times a day.
- **`permissions.deny`** — reads of secret files; anything genuinely off-limits.
- **`hooks`** — deterministic automation (formatter, guard rails).
- **`env`** — environment variables the project needs (not secrets).
- **`model`** — only if the project genuinely needs a specific model; otherwise leave it
  to each developer.

See [../examples/.claude/settings.json](../examples/.claude/settings.json) for a commented
version, and [04 — Permissions and safety](04-permissions-and-safety.md) for the rule
syntax.

### Verifying

```
/status      # lists the settings sources that loaded for this session
claude doctor  # reports entries Claude Code rejected
```

---

## 2.3 `.mcp.json` — external tools

MCP (Model Context Protocol) servers give Claude Code tools beyond files and shell:
databases, issue trackers, browsers, cloud APIs. A project-scoped `.mcp.json` at the repo
root is committed and shared:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    }
  }
}
```

You can also add servers ad hoc with `claude mcp add`, and manage them with `/mcp`.

> **Cost warning:** every enabled MCP server injects its tool definitions into context on
> **every turn**. Two or three focused servers are fine; a dozen "might be handy" servers
> is a standing tax on every message you send. Enable what the project uses, disable the
> rest. This is covered again in [05 — Credit efficiency](05-credit-efficiency.md).

---

## 2.4 What to commit

**Commit:**
- `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/`
- `.claude/settings.json`
- `.claude/skills/`, `.claude/commands/`, `.claude/agents/`
- `.mcp.json`

**Don't commit (add to `.gitignore`):**
- `.claude/settings.local.json` (Claude Code handles this automatically in most cases)
- `CLAUDE.local.md`
- Anything with a secret in it

A ready-made [.gitignore](../.gitignore) is in this repo.

---

Next: **[03 — Skills, commands, and subagents](03-skills-commands-subagents.md)**
