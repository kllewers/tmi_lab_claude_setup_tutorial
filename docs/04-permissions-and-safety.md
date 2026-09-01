# 4. Permissions and safety

Two ideas run through this doc:

1. **`CLAUDE.md` is guidance; it is not enforcement.** If something *must* happen or *must
   not* happen, encode it in `settings.json` or a hook, which the client enforces
   regardless of what the model decides.
2. **An allowlist is also an efficiency tool.** Every permission prompt is a round-trip
   that stalls the agent and costs a turn. Pre-approving your safe, frequent commands
   makes sessions both smoother and cheaper — as long as the list stays genuinely safe.

---

## 4.1 Permission modes

Claude Code runs in one of these modes. Cycle with **Shift+Tab**, or set with
`--permission-mode`.

| Mode | Behavior | When |
| --- | --- | --- |
| **default** | Prompts for anything not on the allowlist | Normal work |
| **plan** | Read-only: Claude investigates and proposes a plan, makes no changes | Start here for anything non-trivial |
| **acceptEdits** | Auto-accepts file edits, still prompts for commands | Bulk mechanical edits you'll review in the diff |
| **bypassPermissions** | No prompts at all | Rarely. Sandboxed throwaway environments only |

**`bypassPermissions` is the dangerous one.** It lets the agent run any command without
asking — `rm -rf`, `git push --force`, `curl | sh`. Use it only in a container or VM you
can throw away, never on your real working copy. Some organizations disable it entirely.

Plan mode is the one to build a habit around — see
[05 — Credit efficiency](05-credit-efficiency.md).

---

## 4.2 Permission rules: allow / ask / deny

In `settings.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test:*)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Read(src/**)"
    ],
    "ask": [
      "Bash(git push:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

- **`allow`** — runs without a prompt.
- **`ask`** — always prompts, even if an `allow` rule would otherwise match. Use it to put
  a speed bump in front of risky-but-sometimes-needed commands.
- **`deny`** — blocked outright. `deny` and `ask` take effect immediately, even before a
  teammate has "trusted" the folder; `allow` rules wait for that trust step.

### Rule syntax

| Pattern | Matches |
| --- | --- |
| `Bash(npm run test)` | Exactly that command |
| `Bash(npm run test:*)` | That command with any arguments |
| `Bash(git diff:*)` | `git diff` with any arguments |
| `Read(src/**)` / `Edit(src/**)` | File reads/edits under `src/` |
| `Read(./.env)` | A specific file (relative to the project) |
| `WebFetch(domain:docs.example.com)` | Fetches from that domain |
| `mcp__postgres__query` | A specific MCP tool |

Lists **merge across scopes** (user + project + local), so your personal `~/.claude/`
allowlist and the project's committed one both apply.

Manage rules interactively with `/permissions`.

### What to allow

Safe, read-mostly, frequently-used commands:

- `lint`, `test`, `typecheck`, `build` (if it has no side effects)
- `git status`, `git diff`, `git log`, `git branch`
- Package-manager *read* operations

### What to keep OFF the allowlist

Anything that changes the world outside your working tree:

- `git push`, `git commit` (many people want to eyeball these — use `ask` or leave to a prompt)
- Deploys, `terraform apply`, `kubectl apply`
- Database migrations, `DROP`, destructive SQL
- `rm -rf`, `chmod -R`, anything with `sudo`
- `curl … | sh`, `npx` of unknown packages

---

## 4.3 Protecting secrets

- **`deny` reads of secret files** — `.env`, `.env.*`, `secrets/**`, key files, `*.pem`.
  A `deny` on `Read` also stops the content reaching context indirectly.
- **Don't put secrets in `settings.json`, `CLAUDE.md`, or a skill.** The `env` block in
  committed settings is for non-secret config only.
- **Keep real credentials in your shell environment or a secrets manager**, and let the
  app read them at runtime.
- **`CLAUDE.local.md` and `.claude/settings.local.json` are gitignored** — but gitignored
  is not encrypted. Don't rely on them to hold anything you'd mind leaking locally.

---

## 4.4 Hooks — deterministic automation

A hook is a shell command Claude Code runs at a lifecycle event. Hooks are the answer to
every "from now on, always/never/before/after…" requirement, because they run whatever the
model intended.

Common events:

| Event | Fires | Typical use |
| --- | --- | --- |
| `PreToolUse` | Before a tool call | Block edits to protected paths; veto dangerous commands |
| `PostToolUse` | After a tool call | Run the formatter / linter after every edit |
| `UserPromptSubmit` | When you send a message | Inject dynamic context (branch, ticket) |
| `SessionStart` | Session begins | Print environment info, warm a cache |
| `Stop` / `SubagentStop` | Turn / subagent ends | Notify, log, run a final check |
| `PreCompact` | Before compaction | Persist notes you don't want summarized away |

Example — format TypeScript after every edit (`settings.json`):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "prettier --write \"$CLAUDE_FILE_PATHS\" 2>/dev/null || true" }
        ]
      }
    ]
  }
}
```

Example — block edits to a generated directory (`PreToolUse`, exit non-zero to veto):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "case \"$CLAUDE_FILE_PATHS\" in *\"/generated/\"*) echo 'generated/ is off limits' >&2; exit 2 ;; esac" }
        ]
      }
    ]
  }
}
```

A `PreToolUse` hook that exits non-zero blocks the call. Because hooks in a committed
`settings.json` execute shell on every teammate's machine, Claude Code makes them wait for
the workspace-trust step before they run.

> This repo has a skill, `update-config`, that is designed for editing `settings.json`
> hooks safely — `/update-config`. In your own projects, use the docs above.

---

## 4.5 Sandboxing

Claude Code can run bash tool calls inside an OS sandbox (`sandbox-exec` on macOS,
namespaces on Linux), limiting filesystem and network access even for allowed commands.
Configure via the `sandbox` settings. Worth turning on if you run in `acceptEdits` a lot
or let the agent execute broadly. Organizations can enforce it via managed settings.

---

## 4.6 A minimal safe baseline

Commit this to `.claude/settings.json` and adjust the command names to your stack:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test:*)",
      "Bash(npm run typecheck)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./**/*.pem)"
    ]
  }
}
```

The full commented example is at
[../examples/.claude/settings.json](../examples/.claude/settings.json).

---

Next: **[05 — Credit efficiency](05-credit-efficiency.md)**
