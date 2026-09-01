# 3. Skills, commands, and subagents

Three ways to extend Claude Code. They solve different problems, and the biggest setup
mistake is cramming everything into `CLAUDE.md` instead of reaching for one of these.

| Mechanism | Lives in | Loaded | Use it for |
| --- | --- | --- | --- |
| **Skill** | `.claude/skills/<name>/SKILL.md` | On demand (invoked or judged relevant) | Procedures and reference knowledge that shouldn't sit in context all the time |
| **Slash command** | `.claude/commands/<name>.md` (or a skill) | When you type `/name` | Actions with side effects whose timing you want to control |
| **Subagent** | `.claude/agents/<name>.md` | When delegated a task | Scoped work you want done in a *separate* context window |

All three exist at both **project** scope (`.claude/…`, committed) and **personal** scope
(`~/.claude/…`).

---

## 3.1 Skills

A skill is a directory with a `SKILL.md` file: YAML frontmatter plus markdown
instructions. The directory name becomes the command (`/pr-description`).

```
.claude/skills/
└── pr-description/
    ├── SKILL.md          # required
    └── template.md       # optional supporting file, loaded only if referenced
```

### Why skills matter for efficiency

`CLAUDE.md` content is loaded **every session** and stays in context every turn — a
permanent cost. A skill's body loads **only when used**. So a 300-line deployment
checklist costs ~nothing as a skill and is dead weight in `CLAUDE.md`.

> Rule of thumb: if a section of `CLAUDE.md` reads like a *procedure* rather than a
> *fact*, move it to a skill.

### Minimal `SKILL.md`

```markdown
---
description: >
  Writes a PR description from the staged diff. Use when the user asks for a PR
  description, PR summary, or "write the PR".
---

## Diff

!`git diff --staged`

## Instructions

Using the diff above, fill in the template in [template.md](template.md):
- One-sentence summary
- Bullet list of changes grouped by area
- "Testing" section describing how it was verified
- Call out anything risky (schema changes, config, migrations)
```

Notes:

- **`description` is the one field that matters.** It's how Claude decides whether to auto-
  load the skill. Put the primary use case first; add trigger phrases.
- **`` !`command` ``** runs a shell command and injects its output *before* Claude reads
  the skill — the instructions arrive with live data already inlined.
- **`$ARGUMENTS`, `$1`, `$2`…** interpolate arguments you pass: `/pr-description HOTFIX`.
- Keep `SKILL.md` **under ~500 lines**; move bulky reference material into sibling files
  and link to them so they load only when needed.
- Keep the body concise — once loaded, it stays in context across turns, so every line is
  a recurring cost. Same discipline as `CLAUDE.md`.

### Useful frontmatter fields

| Field | Effect |
| --- | --- |
| `description` | When to use the skill (recommended). |
| `disable-model-invocation: true` | Only *you* can run it (`/name`); Claude won't trigger it on its own. Use for anything with side effects — `/deploy`, `/commit`, `/send-slack`. |
| `user-invocable: false` | Only *Claude* can load it; hidden from the `/` menu. Use for pure background knowledge. |
| `allowed-tools: Read Grep` | Tools pre-approved for the turn that invokes the skill (no permission prompt). |
| `context: fork` | Run the skill in its own subagent context instead of the main one. |
| `model` / `effort` | Override model or reasoning effort while the skill runs. |
| `paths: ["src/api/**"]` | Only auto-load when working with matching files. |
| `argument-hint: "[ticket-id]"` | Autocomplete hint. |

### Where skills live and how conflicts resolve

| Location | Applies to |
| --- | --- |
| `~/.claude/skills/<name>/` | All your projects |
| `.claude/skills/<name>/` | This project |
| Plugin | Wherever the plugin is enabled (namespaced `plugin:name`) |

Personal overrides project; a local skill can also override a bundled one of the same
name. Nested `.claude/skills/` directories (e.g. in a monorepo package) load when Claude
touches files in that subtree.

### Bundled skills

Claude Code ships skills like `/code-review`, `/debug`, `/loop`, `/run`. Some run
automatically when relevant; others only when you invoke them, precisely so *you* decide
when to spend the tokens. Type `/` to see what's available.

---

## 3.2 Slash commands

A file at `.claude/commands/new-endpoint.md` creates `/new-endpoint`. Custom commands have
been folded into the skill system — a command file and a `SKILL.md` do the same thing.
Prefer a skill when you want a supporting-files directory or auto-invocation; a plain
command file is fine for a quick one-shot prompt.

```markdown
---
description: Scaffold a new REST endpoint with handler, route, and test.
argument-hint: "[resource-name]"
---

Create a new endpoint for `$1`:

1. Handler in `src/api/handlers/$1.ts` following the pattern in `users.ts`.
2. Register the route in `src/api/router.ts`.
3. A test in `src/api/handlers/$1.test.ts` covering the happy path and a 400.
4. Run `pnpm test src/api/handlers/$1.test.ts` and fix failures.
```

The same string substitutions (`$ARGUMENTS`, `$1`, `` !`cmd` ``) work here. See
[../examples/.claude/commands/new-endpoint.md](../examples/.claude/commands/new-endpoint.md).

**When to use a command instead of just typing the prompt:** when you'll run it more than
twice, when the exact wording matters, or when you want side effects (`/commit`,
`/deploy`) to happen only on your say-so — set `disable-model-invocation: true`.

---

## 3.3 Subagents

A subagent is defined by a markdown file in `.claude/agents/` with frontmatter naming it,
describing when to use it, and optionally restricting its tools:

```markdown
---
name: test-runner
description: >
  Runs the test suite, diagnoses failures, and reports back. Use proactively
  after code changes that could affect tests.
tools: Bash, Read, Grep, Glob
---

You run and triage tests. Steps:
1. Run the full suite with `pnpm test`.
2. For each failure, read the relevant source and test files and determine the cause.
3. Report a concise summary: what failed, why, and the smallest fix. Do NOT fix
   code yourself unless asked.
```

### Why subagents save context

A subagent runs in **its own context window**. It can read twenty files, run the tests
three times, and think hard — and your main session only receives its final summary. The
noisy middle never touches your main context.

Good candidates: test triage, log/trace analysis, "search the codebase and tell me where
X is handled", dependency audits, multi-file reviews.

### The trade-off

A subagent starts **cold** — it doesn't inherit your conversation. You pay to re-establish
context it needs. So delegate work that is *separable* and *summary-shaped*, not a task
that needs everything you've discussed so far. For that, a `context: fork` skill (which
inherits the conversation) is the better tool.

See [../examples/.claude/agents/test-runner.md](../examples/.claude/agents/test-runner.md).

---

## 3.4 Choosing between them

```
Is it a fact true every session?            -> CLAUDE.md
Is it a procedure / reference doc?           -> skill
 └ with side effects, timing matters?        -> skill w/ disable-model-invocation, or a command
Is it separable work with a summary output?  -> subagent
Does it need the current conversation?       -> context: fork skill
Must it happen deterministically every time? -> hook (see doc 04)
```

---

Next: **[04 — Permissions and safety](04-permissions-and-safety.md)**
