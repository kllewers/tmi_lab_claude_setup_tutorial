# 5. Credit efficiency — making your usage go further

Whether you're billed per token (API) or against a usage window (subscription), the same
thing wastes it: **tokens spent re-establishing context, exploring blindly, or re-reading
a bloated context window on every turn.**

This doc is the payoff of the rest of the tutorial. It's organized as: how the cost model
actually works, then the habits, then the configuration.

---

## 5.1 How you actually get billed

Every turn, Claude Code sends the model:

- the system prompt + tool definitions (including **every enabled MCP server's** tools)
- your `CLAUDE.md` and any active rules
- the **entire conversation so far** — every message, every file you've read, every
  command's output
- your new message

Then the model generates a response (output tokens, priced higher than input; more
reasoning effort = more of them).

Two consequences:

1. **Context is cumulative.** A long session re-sends everything every turn. The 40th
   message in a session costs far more than the 4th, even if it's the same question.
2. **Prompt caching softens this.** Claude Code automatically caches the stable prefix of
   the request (system prompt, `CLAUDE.md`, early conversation). Cached input tokens are
   much cheaper. But the cache only helps the **unchanging prefix** — and it's **per
   model**, so switching models mid-task throws it away and the next request re-reads
   everything uncached.

The whole game is: keep the prefix small and stable, keep sessions short, and don't make
the model work harder than the task needs.

---

## 5.2 The habits (highest leverage first)

### 1. One session per task. `/clear` between unrelated work.

The single biggest lever. When you finish a task, `/clear` (or just start a new `claude`
session) before the next one. Don't debug a test failure in the same session where you
just designed a feature and refactored three modules — the model is paying to carry all of
that on every turn of the debugging.

### 2. Use plan mode before non-trivial changes.

Press **Shift+Tab** to enter plan mode. Claude investigates read-only and proposes an
approach; you correct it *before* any code is written. This is dramatically cheaper than
letting it write the wrong thing, then re-reading the mess, then rewriting. Cheap to plan,
expensive to thrash.

### 3. Point at files. Don't make it search from zero.

"The bug is in `src/auth/session.ts`, probably in `refreshToken`" costs a few hundred
tokens. "There's a bug somewhere in auth" costs a whole exploration phase — greps, reads
of a dozen files, dead ends — all of which stay in context for the rest of the session.
You almost always know roughly where things are. Say so.

### 4. Give the whole brief up front.

Each clarifying round-trip re-sends the entire context. One message with the goal,
constraints, the relevant file paths, and what "done" looks like beats five terse ones.

### 5. Watch the context window.

```
/context     # shows what's using the window and how full it is
/compact     # summarize the conversation so far, keep going
/clear       # wipe conversation, keep CLAUDE.md/settings
```

When `/context` shows you're getting full, decide deliberately: `/compact` if you still
need the thread, `/clear` if you don't. Letting it auto-compact repeatedly in a marathon
session is a sign the task should have been several sessions.

### 6. Right-size the model and effort.

- Mechanical edits, renames, boilerplate → small model (Haiku-class), low effort.
- Everyday feature work → mid model (Sonnet-class), medium effort.
- Genuinely hard architecture / subtle bugs → top model (Opus-class) and/or high effort.

Set your **default** to the everyday case (`/model`, `/effort`) and bump up only when a
task warrants it. A default of "most expensive, highest effort" means you pay a premium on
every trivial edit.

### 7. Delegate noisy, separable work to a subagent.

Test triage, log analysis, "find everywhere we do X" — hand these to a subagent (doc 03).
It burns tokens in its own context and returns you a summary; the twenty file reads and
three test runs never enter your main window.

### 8. Let the agent finish before piling on.

Interrupting to add "oh also…" mid-turn discards partial work and re-sends context on the
restart. If it's genuinely going wrong, interrupt and redirect; otherwise let the turn
complete.

### 9. Don't paste what Claude can read.

Pasting a 500-line file into chat puts it in context permanently. Letting Claude `Read`
the relevant span is cheaper and it can re-read if needed. Same for logs — point at the
file.

---

## 5.3 The configuration (set once, benefits every session)

### Keep `CLAUDE.md` under ~200 lines.

It's in context every turn of every session. Cut anything Claude can derive from the code
(directory trees, dependency lists, framework restatements). `/doctor` can propose trims.
Every 100 lines you cut is ~a thousand tokens off *every message you ever send* in the
project.

### Move procedures to skills.

A skill's body costs nothing until invoked. A 300-line checklist in `CLAUDE.md` costs on
every turn. See doc 03.

### Prune MCP servers.

Every enabled server's tool definitions ride along on every turn. Audit with `/mcp`.
Disable the ones this project doesn't use. A lean `.mcp.json` is a permanent discount.

### Turn off bundled skills / features you don't use.

If your team never uses certain bundled skills, `disableBundledSkills` (or per-skill
overrides) trims their listing overhead. Minor, but free.

### Pre-approve safe commands.

An allowlist (doc 04) removes permission round-trips. Each avoided prompt is a turn you
didn't spend.

### Use path-scoped rules instead of one big `CLAUDE.md`.

`.claude/rules/*.md` with `paths:` frontmatter load only when Claude touches matching
files. Front-end rules don't ride along while you work on the backend.

---

## 5.4 Checking your spend

| | Command / place |
| --- | --- |
| API cost this session | `/cost` |
| Subscription usage / window | `/usage` |
| What's in context right now | `/context` |
| API spend over time, limits | Anthropic Console dashboard |

Get in the habit of glancing at `/context` when a session feels heavy and `/cost` (API) at
the end of a big task. You'll quickly learn which of your workflows are expensive.

---

## 5.5 A worked example

**Task:** "Add rate limiting to the login endpoint."

**Wasteful version:**
- Continue in the session where you spent the morning on an unrelated refactor (huge context).
- "Add rate limiting somewhere in auth." Claude greps the codebase, reads 15 files.
- It writes an implementation using a library you don't use.
- "No, we use `rate-limiter-flexible`." It re-reads, rewrites.
- Meanwhile every turn re-sends the morning's refactor + 15 file reads + two implementations.

**Efficient version:**
- `/clear`.
- Plan mode (Shift+Tab): "Add rate limiting to the login handler in
  `src/api/handlers/auth.ts`. Use `rate-limiter-flexible` (already a dependency, see
  `src/lib/ratelimit.ts` for our wrapper). 5 attempts / 15 min per IP, return 429 with our
  standard error envelope. Add a test."
- Review the plan, approve.
- Claude edits two files, adds a test, runs it.
- Done in a handful of turns on a small context.

Same outcome. A fraction of the tokens.

---

Back to the **[README](../README.md)** · Scaffold: **[../examples/](../examples/)**
