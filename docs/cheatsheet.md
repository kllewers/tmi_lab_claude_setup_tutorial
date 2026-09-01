# CHEATSHEET

---

## Framing

> "Claude Code is an agent that runs in our terminal and in VS Code. This repo is about
> the *setup around it* — the files that make it useful instead of expensive and messy."

The one idea, say it plainly:

- **Every turn, the model is re-sent the entire conversation so far** — every message,
  every file it read, every command's output. Think of a whiteboard that gets photographed
  and re-sent before every question. It only gets more crowded.
- So setup is a question of **what goes on the whiteboard, and when.** Stable facts loaded
  once; situational stuff kept out until it's needed; hard rules enforced, not hoped for.
- Prompt caching makes the *stable prefix* cheap, but only if it stays stable and small.

That's the whole philosophy. The five pillars are just where different things live.

---

## The flow

The order you actually do this in a repo:

1. **Install once** — `npm i -g @anthropic-ai/claude-code`, install the VS Code "Claude
   Code" extension, run `claude` in the integrated terminal, `/login`. (CLI ships on
   Node; your project can be any language — ours is Python.)
2. **`cd` into the repo, run `/init`** — Claude reads the codebase and drafts a
   `CLAUDE.md`. You then add what it couldn't know.
3. **Add a safety floor** — commit a `.claude/settings.json` that `deny`s reads of
   `.env`/secrets and `allow`s your safe, frequent commands (test, lint, typecheck).
4. **As you work, put things in their right home** (the five pillars):
   - a fact you keep re-explaining → **CLAUDE.md**
   - …but only relevant to some files → **a rule**
   - a procedure / checklist → **a skill**
   - something that must happen every time → **a hook**
   - a permission / behaviour / env var → **settings.json**
5. **Run it economically** — one session per task, `/clear` between unrelated work, plan
   mode (Shift+Tab) before non-trivial changes, point Claude at specific files instead of
   letting it search from zero.

Everything in `examples/` is a working version of step 4 you can copy in.

---

## The five pillars

### 1. CLAUDE.md — the standing facts

- **What:** markdown file of standing instructions, **loaded into context every session.**
- **Lives:** repo root (`CLAUDE.md` or `.claude/CLAUDE.md`). Also `~/.claude/CLAUDE.md`
  for personal, machine-wide.
- **Holds:** build/test/lint commands, where code lives, conventions that differ from
  defaults, gotchas.
- **In this repo:** [`examples/CLAUDE.md`](../examples/CLAUDE.md) — note the *Gotchas*:
  "`src/domain/billing/v1/` is legacy, leave it alone," and "`pytest` needs Postgres on
  5432 — `docker compose up -d db` first." Claude can't infer those; one line each saves a
  bad afternoon.
- **The rule:** keep it **under ~200 lines** — bigger costs more context *and* gets
  followed less. If a section reads like a procedure → make it a skill. If Claude could
  learn it by reading the code → delete it.

### 2. RULES — CLAUDE.md, but scoped to paths

- **What:** extra instruction files that load **only when Claude touches matching files.**
- **Lives:** `.claude/rules/*.md`, with `paths:` frontmatter (a rule with no `paths:`
  loads every session, like CLAUDE.md).
- **Holds:** conventions for one part of the tree — API rules, test rules, frontend rules.
- **In this repo:** [`examples/.claude/rules/testing.md`](../examples/.claude/rules/testing.md)
  — `paths: ["tests/**/*.py", "**/test_*.py"]`. Our pytest conventions are in context when
  we write tests, and *not* riding along during a database migration.
- **The rule:** this is how you keep a big project's instructions from all loading at once.
  Context you get back for free.

### 3. SKILLS — procedures and reference, loaded on demand

- **What:** a folder with a `SKILL.md`; loads **only when invoked (`/name`) or when Claude
  judges it relevant** from its `description`. Nearly free until used.
- **Lives:** `.claude/skills/<name>/SKILL.md` (+ optional supporting files);
  `~/.claude/skills/` for personal.
- **Holds:** deploy checklists, "write the PR," code-gen recipes, domain reference docs.
  Can run shell with `` !`cmd` ``, take `$1` args, link sibling files that load lazily.
- **In this repo:** [`examples/.claude/skills/pr-description/`](../examples/.claude/skills/pr-description/)
  — `/pr-description` runs the staged diff and fills a template. The whole procedure is
  never in context until you type the command.
- **The rule:** if a chunk of CLAUDE.md reads "step 1, step 2" → it's a skill. Add
  `disable-model-invocation: true` to anything with side effects (`/deploy`, `/commit`) so
  only *you* can fire it.

### 4. HOOKS — deterministic enforcement

- **What:** a shell command Claude Code runs at a lifecycle event. **Runs regardless of
  what the model decides.**
- **Lives:** the `hooks` block in `settings.json`. Events: `PreToolUse`, `PostToolUse`,
  `UserPromptSubmit`, `SessionStart`, `Stop`, `PreCompact`, …
- **Holds:** the answer to every "from now on, always / never / before / after…"
- **In this repo:** [`examples/.claude/settings.json`](../examples/.claude/settings.json)
  — `PostToolUse` runs `ruff format` after every edit. The docs also show a `PreToolUse`
  hook that **vetoes** edits to `src/_generated/` by exiting non-zero.
- **The rule:** "please format your code" in CLAUDE.md is a line in the style guide. A
  hook is a CI check that fails the build. Use hooks when it *must* happen.

### 5. settings.json — how Claude Code behaves

- **What:** JSON keys controlling permissions, hooks, env, model. **Strict JSON** — no
  comments, no trailing commas. Keep the `$schema` line for editor autocomplete.
- **Four scopes:**
  - `~/.claude/settings.json` — you, every project
  - `.claude/settings.json` — **committed**, everyone on the project
  - `.claude/settings.local.json` — you, this project, **gitignored** (Claude Code
    auto-excludes it; it's where "don't ask again" approvals get saved)
  - managed — your org, wins over everything
  - list keys like `permissions.allow` **merge** across scopes
- **Holds — mainly `permissions`:** `allow` (runs with no prompt — put your safe, frequent
  commands here; fewer prompts = fewer round-trips = cheaper), `ask` (always prompt, e.g.
  `git push`), `deny` (blocked — `.env`, `secrets/**`, `*.pem`, `src/_generated/`).
- **In this repo:** [`examples/.claude/settings.json`](../examples/.claude/settings.json)
  — allowlisted `uv run pytest`/`ruff`/`mypy`, `ask` on `git push`/`git commit`, `deny` on
  secrets and generated code.
- **The rule:** the allowlist is also a *speed and cost* tool. What stays **off** it:
  anything irreversible or reaching outside the repo — deploys, migrations, `rm -rf`,
  `sudo`, `npx`/`uvx` of unknown packages.

---

## 9:00 · Which bucket does this go in? (30s)

| The thing | Goes in |
| --- | --- |
| A fact true in every session | **CLAUDE.md** |
| A fact true only for some files | **a rule** (`paths:`) |
| A procedure or reference doc | **a skill** |
| Something that must happen every time | **a hook** |
| A permission, env var, or behaviour toggle | **settings.json** |

---

## 9:30 · Close — the 10-minute starter (30s)

> "You don't need all of this on day one."

1. Run `/init`.
2. Add `deny` rules for `.env` / `secrets/` to `.claude/settings.json`.
3. Allowlist your test command.

That's already better than most setups. Then it compounds: every time you re-explain
something → a CLAUDE.md line or a skill. Every time a prompt annoys you → an allowlist
entry. Grab `examples/`, apply it to the repo you work in most this week.
