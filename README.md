# Setting up a Claude Code environment in VS Code

A practical, opinionated tutorial for standing up a **Claude Code** environment that treats
AI as a tool — configured deliberately, scoped tightly, and run so that your usage
(tokens / credits / rate limits) goes as far as possible.

It is written for developers who are comfortable with VS Code and Git but new to Claude
Code specifically. It skips the hand-holding and focuses on the mental model, the files
that make up a project setup, and the habits that keep sessions cheap.

---

## What's in here

| Doc | What it covers |
| --- | --- |
| **[docs/01-prerequisites.md](docs/01-prerequisites.md)** | Everything to install and configure *before* your first session: Node, the CLI, the VS Code extension, authentication, and the API-vs-subscription decision. |
| **[docs/02-project-scaffolding.md](docs/02-project-scaffolding.md)** | The `.claude/` directory, `CLAUDE.md`, `settings.json`, `.mcp.json`, and what to commit vs. gitignore. |
| **[docs/03-skills-commands-subagents.md](docs/03-skills-commands-subagents.md)** | The three extension mechanisms — skills, slash commands, and subagents — what each is for, and how to write them. |
| **[docs/04-permissions-and-safety.md](docs/04-permissions-and-safety.md)** | Permission modes, allow/ask/deny rules, hooks, sandboxing, secrets hygiene, and avoiding destructive actions. |
| **[docs/05-credit-efficiency.md](docs/05-credit-efficiency.md)** | The core of this repo: how the context window and prompt caching actually work, and the concrete habits that reduce spend. |
| **[docs/recording-script.md](docs/recording-script.md)** | A full narration + demo script for recording a ~40-minute screencast walkthrough of this material, with a prep checklist and shot list. |
| **[examples/](examples/)** | A working scaffold you can copy into a real project: a commented `CLAUDE.md`, `settings.json`, `.mcp.json`, a skill, a slash command, and a subagent. |

If you read nothing else, read **[docs/05-credit-efficiency.md](docs/05-credit-efficiency.md)** and the
**[appropriate setup checklist](#the-appropriate-setup-checklist)** below.

---

## The 60-second mental model

Claude Code is an agent that runs in your terminal (and, via the extension, inside VS
Code). Every session starts with an empty **context window**. What you put into that
window — and how much of it you burn re-establishing the same facts every session — is
what determines both quality and cost.

A good setup front-loads the *stable* facts once, keeps the *situational* material out of
context until it's needed, and enforces the *non-negotiable* rules mechanically instead of
hoping the model complies.

```
Stable facts        ->  CLAUDE.md            (loaded every session, keep it small)
Situational detail  ->  Skills / rules       (loaded on demand, cost ~nothing until used)
Repeatable actions  ->  Slash commands       (you invoke them by name)
Isolatable work     ->  Subagents            (run in their own context, return a summary)
Hard rules          ->  settings.json + hooks (enforced by the client, not the model)
```

Everything in this tutorial is an application of that one idea.

---

## Quick start

```bash
# 1. Install (Node 18+ required)
npm install -g @anthropic-ai/claude-code

# 2. From your project directory, start a session
cd path/to/your/project
claude

# 3. Authenticate when prompted (/login), then generate a starting CLAUDE.md
/init

# 4. In VS Code: install the "Claude Code" extension, open the integrated
#    terminal, and run `claude` there — or press Cmd/Ctrl+Esc.
```

Then copy the pieces you want from [examples/](examples/) into your project's `.claude/`
directory. Full detail in [docs/01-prerequisites.md](docs/01-prerequisites.md) and
[docs/02-project-scaffolding.md](docs/02-project-scaffolding.md).

---

## The "appropriate setup" checklist

"An appropriate setup" means the environment is scoped to the work, safe by default, and
economical to run. Concretely:

### Machine & workspace
- [ ] Node 18+ and the CLI installed; `claude --version` works in the VS Code integrated terminal.
- [ ] The VS Code **Claude Code** extension is installed and connected (`/ide`).
- [ ] You've chosen an auth path deliberately — Claude subscription **or** Console API — and know how to check spend (`/cost`, or the Console dashboard).
- [ ] Your default model and effort are set for the *common* case, not the worst case (`/model`, `/effort`).

### Project scaffolding
- [ ] A project `CLAUDE.md` exists, is under ~200 lines, and contains only facts that are true every session (build/test commands, layout, conventions).
- [ ] `.claude/settings.json` is committed with a permission allowlist for your safe, common commands (lint, test, typecheck).
- [ ] `.claude/settings.local.json` is gitignored (Claude Code does this for you the first time it writes the file).
- [ ] Multi-step procedures live in **skills**, not `CLAUDE.md`.
- [ ] Only the MCP servers you actually use are enabled — each one costs context on every turn.

### Safety
- [ ] `permissions.deny` blocks reads of `.env` and secret files.
- [ ] Destructive commands (`git push`, `rm -rf`, deploys, DB migrations) are **not** on the allowlist and are not run in an auto-accept mode without review.
- [ ] Anything that *must* happen at a fixed point (e.g. run the formatter after every edit) is a **hook**, not a line in `CLAUDE.md`.
- [ ] You know how to leave `bypassPermissions` mode off and why.

### Running economically
- [ ] You start a **new session per task** and use `/clear` between unrelated pieces of work.
- [ ] You use **plan mode** for anything non-trivial before letting the agent edit.
- [ ] You point Claude at specific files/paths instead of asking it to search from scratch.
- [ ] You check `/context` when a session feels sluggish, and `/compact` or `/clear` rather than letting it auto-compact repeatedly.

Each item is explained in the linked docs.

---

## A note on terminology

- **CLAUDE.md** — a markdown file of standing instructions, loaded into context at the
  start of every session. Your project's "always known" facts.
- **Skill** — a `SKILL.md` file (plus optional supporting files) describing a procedure or
  body of reference knowledge. Loaded only when invoked or when Claude judges it relevant,
  so it's nearly free until used. Custom slash commands are now a kind of skill.
- **Slash command** — a skill (or a file in `.claude/commands/`) you trigger by typing
  `/name`. Good for actions with side effects you want to control the timing of.
- **Subagent** — a separate agent with its own context window, defined in
  `.claude/agents/`. It does a scoped job and returns a summary, keeping the bulk of its
  work out of your main context.
- **Hook** — a shell command Claude Code runs at a lifecycle event (before/after a tool
  call, on session start, etc.). Enforced by the client regardless of what the model
  decides.
- **MCP server** — an external tool provider (databases, issue trackers, browsers…) that
  Claude Code connects to. Configured in `.mcp.json`.
- **Settings** — JSON keys in `settings.json` that change how Claude Code behaves:
  permissions, model, hooks, environment.

---

## License

This tutorial content is provided as-is for you to adapt. See [LICENSE](LICENSE).
