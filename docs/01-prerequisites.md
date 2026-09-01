# 1. Prerequisites — the machine and workspace setup

Everything here happens **once per machine** (mostly). Get it right and every project
afterwards is just scaffolding.

---

## 1.1 Install the CLI

Claude Code itself is distributed as a Node package, so the quickest path needs
**Node.js 18 or newer**. This has nothing to do with your project's language — the
`examples/` in this repo are a **Python** project, and Claude Code works the same on any
stack.

```bash
node --version          # must be >= v18
npm install -g @anthropic-ai/claude-code
claude --version        # confirm it's on your PATH
```

If you don't manage Node yourself, install it via [nvm](https://github.com/nvm-sh/nvm)
(`nvm install --lts`) or Homebrew (`brew install node`). Avoid the system Node on macOS.

If you'd rather not have Node at all, Claude Code also ships a standalone native
installer — check the official install docs for the current one-line command.

> The `claude` binary is the same program whether you run it in a plain terminal, the VS
> Code integrated terminal, or the JetBrains terminal. The VS Code *extension* adds a UI
> on top; it does not replace the CLI.

Run `claude doctor` any time to check your installation and configuration for problems.

---

## 1.2 Install and connect the VS Code extension

1. In VS Code, open **Extensions** and install **"Claude Code"** (publisher: Anthropic).
2. Open the integrated terminal (`Ctrl+``) and run `claude`. The extension detects the
   session and connects automatically. You can also press **`Cmd+Esc`** (macOS) /
   **`Ctrl+Esc`** (Windows/Linux) to launch it.
3. Inside the session, run `/ide` to confirm the editor connection is active.

What the extension gives you over a bare terminal:

- **Inline diffs** — proposed edits show up in the editor's diff view, not just as text.
- **Selection context** — code you highlight in the editor is passed to Claude as context.
- **Permission prompts as UI cards**, including the option to save an approval to the
  project's shared settings (the CLI only writes to your local settings).
- **Diagnostics** — lint/type errors visible to VS Code are visible to Claude.

Everything else in this tutorial — `CLAUDE.md`, `settings.json`, skills, hooks — is read
from the same files whether you drive Claude Code from the extension or the terminal.

---

## 1.3 Authenticate: subscription vs. API

On first run, `claude` prompts you to `/login`. There are two paths, and the choice
affects how you think about cost.

| | **Claude subscription (Pro / Max)** | **Anthropic Console (API)** |
| --- | --- | --- |
| Billing | Flat monthly fee | Pay-as-you-go, per token |
| Limits | Usage limits reset on a rolling window | Spend limits you set in the Console |
| Check usage | `/usage` in a session | Console dashboard, plus `/cost` in a session |
| Best for | Individuals, steady daily use | Teams, CI, metered/chargeback setups, high-volume |
| Model access | Models included in your plan | Every model, priced individually |

You can switch later with `/login`. If your organization manages Claude Code, the login
method may be enforced for you — `/status` shows what's managed.

**Why it matters for this tutorial:** on a subscription you're optimizing against *rate
limits* (don't burn your window on wasteful sessions); on the API you're optimizing
against a *dollar figure* (every wasted token is money). The habits in
[docs/05-credit-efficiency.md](05-credit-efficiency.md) help with both.

---

## 1.4 Set sensible defaults

Inside a session:

```
/model          # pick the model new sessions start on
/effort         # pick the default reasoning effort (low / medium / high / ...)
/config         # theme, editor mode, verbose output, and other personal options
```

Guidance:

- **Default to a mid-tier model** (Sonnet-class) for everyday work. Reach for the top-tier
  model (Opus-class) on genuinely hard problems, and a small model (Haiku-class) for
  mechanical edits. Don't set your default to the most expensive option "just in case."
- **Match effort to the task, not your anxiety.** Higher effort means more reasoning
  tokens per turn. `high` and above earn their keep on architecture and tricky debugging;
  they're waste on renaming a variable.
- These are saved as your personal defaults in `~/.claude/settings.json`. A project or
  your organization can override the starting model; the startup header tells you when
  that happens.

> **Prompt-cache note:** each model keeps its own prompt cache. Switching models
> mid-session means the next request re-reads the whole conversation uncached. Pick a
> model at the start of a task and stick with it.

---

## 1.5 Where Claude Code keeps its files

| Path | What it is |
| --- | --- |
| `~/.claude/settings.json` | Your personal settings for every project |
| `~/.claude/CLAUDE.md` | Your personal standing instructions for every project |
| `~/.claude/skills/` | Your personal skills |
| `~/.claude/commands/` | Your personal slash commands |
| `~/.claude/agents/` | Your personal subagents |
| `~/.claude.json` | Session/auth state and per-project trust decisions (Claude Code manages this — don't hand-edit) |
| `~/.claude/projects/<project>/memory/` | Auto-memory notes Claude writes for itself, per repo |

Project-scoped equivalents live in `.claude/` inside the repo — that's the next doc.

---

## 1.6 Verify the setup

```
claude doctor       # installation & config health check
claude                # start a session, then:
/status             # which settings sources loaded, what's managed
/context            # what's in the context window right now
/ide                # editor connection status
```

If `claude doctor` is clean, `/status` shows the settings files you expect, and `/ide`
reports connected, you're ready to scaffold a project.

---

Next: **[02 — Project scaffolding](02-project-scaffolding.md)**
