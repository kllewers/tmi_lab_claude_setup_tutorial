# Recording script — "Setting up Claude Code in VS Code"

A full narration + demo script for recording a screencast walkthrough of this repo for
lab mates. Written to be **spoken aloud**. Target run time **~40 minutes**; it compresses
to ~25 if you trim the live demos and keep the narration.

- `NARRATION:` — say this, roughly as written. Contractions and all; it's meant to sound
  spoken.
- `[ON SCREEN]` / `[DEMO]` — what to show or do.
- `[BEAT]` — pause, let the point land, or wait for a command to finish.
- Timings are cumulative and approximate.

The whole script uses **one running example**: a fictional TypeScript API service — the
same "orders service" the files in [`examples/`](../examples/) describe (handlers in
`src/api/handlers/`, a legacy `src/domain/billing/v1/`, a Postgres dependency). Keeping one
project in view the whole time is what makes the pieces feel connected instead of like a
feature list.

---

## Before you record (prep checklist)

- [ ] Terminal / editor font at ~16–18pt. Nobody can read 11pt on a shared screen.
- [ ] A throwaway demo repo cloned locally that matches the `examples/` layout — even a
      near-empty repo with `src/api/handlers/users.ts`, `package.json`, and a couple of
      tests is enough. You'll run `/init`, edit `CLAUDE.md`, and trigger a hook on camera.
- [ ] This tutorial repo open in a second editor window or tab, so you can show the
      `examples/` files.
- [ ] `claude` logged in already. Don't record the auth flow — just talk over it.
- [ ] A clean shell history and a neutral prompt. Hide anything private.
- [ ] Decide your billing story up front: say which one your lab uses (subscription or
      Console API) so the cost section is concrete for your audience.
- [ ] If your lab writes Python, not TypeScript: either swap the demo repo's commands
      (`pytest`, `ruff`, `mypy`) and say so once at the top, or just tell viewers "wherever
      I say `pnpm test`, you say `pytest`." Keep the `examples/` files as-is on screen so
      the repo and the video match.
- [ ] Close Slack / mail / notifications.

---

## 0:00 — Cold open

`[ON SCREEN]` Title card, or just your editor with the repo README open.

NARRATION:
> Hey. This is a walkthrough of how we set up Claude Code — the AI coding agent — inside
> VS Code, in a way that's actually efficient. Not "how to install it," although we'll do
> that. The point of this video is the *setup around it*: the handful of files and habits
> that decide whether this tool saves you time and credits, or quietly burns both.
>
> Here's the honest version of what happens when people start using an AI agent with no
> setup. The first day feels like magic. By the second week you've got a tool that
> re-asks you the same questions every session, goes rummaging through the whole codebase
> to find things you could've pointed it at, occasionally tries to run something you
> really didn't want it to run, and — if you're on metered billing — costs way more than
> it should for what you're getting.
>
> Every one of those problems has a fix, and the fixes are small. That's what we're
> covering. By the end you'll have a checklist you can apply to any repo in about ten
> minutes.

`[BEAT]`

NARRATION:
> Everything I show is in a repo — I'll put the link at the end — including a ready-made
> `examples/` folder you can copy straight into a project. Let's start with the one idea
> that everything else hangs off.

---

## 1:30 — The mental model (the one idea)

`[ON SCREEN]` The "60-second mental model" section of the README, or a simple slide with
the four-bucket diagram.

NARRATION:
> Claude Code is an agent. It runs in your terminal, and through the VS Code extension it
> runs inside the editor. Every time you start a session, it begins with an empty **context
> window** — think of that as its working memory for the conversation.
>
> Here's the mechanical detail that matters, and honestly if you only remember one thing,
> remember this: **every turn, the model is re-sent the entire conversation so far.**
> Every message, every file it read, every command's output. Picture a whiteboard where,
> before each new question, someone photographs the whole board and mails the photo. The
> board only gets more crowded. The 40th message in a session is expensive even if it's a
> tiny question, because it's dragging the previous 39 along with it.

`[BEAT]`

NARRATION:
> So a good setup is basically an exercise in *what goes on the whiteboard, and when.*
> There are four buckets.
>
> One: **stable facts** — things true in every session. Your build command, where the code
> lives, your conventions. Those go in a file called `CLAUDE.md`, which loads every
> session. You want that file small, because it's on the whiteboard permanently.
>
> Two: **situational detail** — a deployment procedure, the rules for writing tests, API
> design guidelines. Stuff that matters sometimes. Those go in **skills** and **rules**,
> which load *on demand* — they cost basically nothing until something actually needs
> them.
>
> Three: **repeatable actions** — "write the PR description," "scaffold a new endpoint."
> Those become **slash commands** you trigger by name.
>
> Four: **hard rules** — "never touch the generated folder," "run the formatter after
> every edit." Those you do *not* leave to the model's judgment. You enforce them
> mechanically, with **settings** and **hooks**.
>
> That's the whole philosophy. Front-load the stable stuff once, keep the situational
> stuff out of context until it's needed, and enforce the non-negotiables instead of
> hoping. Everything else in this video is just applying that.

---

## 4:00 — Prerequisites and first run

`[ON SCREEN]` [`docs/01-prerequisites.md`](01-prerequisites.md). Then switch to a terminal.

NARRATION:
> Setup. This part is once per machine. You need Node 18 or newer, and then it's one
> install command.

`[DEMO]` Type, don't run if already installed — just show it:
```
node --version
npm install -g @anthropic-ai/claude-code
claude --version
```

NARRATION:
> That `claude` binary is the same program whether you run it in a plain terminal, in
> VS Code's integrated terminal, or in the JetBrains one. The VS Code *extension* adds a
> UI layer on top — inline diffs in the editor, your text selection gets passed as
> context, permission prompts show up as clickable cards. But it reads the exact same
> config files. So nothing in this video is extension-only.

`[DEMO]` In VS Code: open Extensions, show "Claude Code" installed. Open the integrated
terminal, run `claude`, then inside it run `/ide`.

NARRATION:
> Install the extension from the marketplace, open the integrated terminal, run `claude`,
> and it connects automatically. `/ide` confirms the editor link is live. You can also
> just hit Cmd-Escape — Ctrl-Escape on Windows — to launch it.

`[BEAT]`

NARRATION:
> One real decision here: how you authenticate. There are two paths. A **Claude
> subscription** — Pro or Max — is a flat monthly fee, and you're working against usage
> limits that reset on a rolling window. The **Anthropic Console API** is pay-as-you-go,
> per token, with spend limits you set yourself. Individuals doing steady daily work
> usually want the subscription. Teams, CI, anything where you need to attribute cost to a
> project — that's the API.
>
> Why do I care which one *you* pick? Because it changes what "waste" means. On a
> subscription, a sloppy session burns your *window* — you hit a limit and you're stuck
> waiting. On the API, a sloppy session burns *dollars*. The habits I'll show at the end
> help with both, but it's worth knowing which number you're watching. [Say which one our
> lab uses here.]

`[DEMO]` Start a fresh `claude` session in the demo repo.

NARRATION:
> Last thing before we scaffold: defaults. Inside a session, `/model` picks which model
> new sessions start on, and `/effort` sets how much reasoning it does per turn. Set these
> for your *common* case, not your worst case. Default to a mid-tier model — Sonnet-class —
> for everyday work. Reach for the big model on genuinely hard problems, and a small fast
> one for mechanical edits. If you set your default to "the most expensive model, maximum
> effort," you're paying a premium to rename a variable.

`[DEMO]` Run `/model`, show the picker, pick the mid-tier option, close it.

---

## 8:00 — CLAUDE.md: the file that matters most

`[ON SCREEN]` [`docs/02-project-scaffolding.md`](02-project-scaffolding.md), the scaffolding
tree. Then open [`examples/CLAUDE.md`](../examples/CLAUDE.md) beside it.

NARRATION:
> Now the per-project setup. This is what makes a new session — or a new lab mate —
> productive without you re-explaining the project every time.
>
> Here's the full picture: a `CLAUDE.md` at the repo root, an optional `.mcp.json` for
> external tools, and a `.claude/` directory with settings, skills, commands, subagents,
> and rules. We'll go through each. Start with `CLAUDE.md`, because it's the most
> important and the easiest to get wrong.

`[BEAT]`

NARRATION:
> `CLAUDE.md` is a markdown file of standing instructions. It loads into context at the
> start of every session. Remember the whiteboard — this file is written on it
> permanently, so its size is a tax you pay on *every single message* for the life of the
> project.
>
> The test I use for what goes in it: **would a new hire need this written down, and is it
> still true next month?** Build and test commands — yes. Where the API handlers live —
> yes. "We use two-space indent, no semicolons" — yes. Those are the onboarding-doc
> facts.
>
> What does *not* go in it: anything that's really a *procedure* — that's a skill.
> Anything that only matters for one corner of the codebase — that's a scoped rule.
> And anything Claude can just *read the code and figure out* — a directory dump, the list
> of dependencies, a paragraph re-explaining how React works. That stuff is pure context
> tax with no upside.

`[DEMO]` Scroll through `examples/CLAUDE.md`. Stop on the "Gotchas" section.

NARRATION:
> Let me show you the kind of thing that earns its place. Look at this gotchas section.
> "`src/domain/billing/v1/` is the legacy billing path — leave it alone unless the task is
> explicitly about it." That is not something Claude can infer. The code looks fine. But
> we know that directory is a minefield, and without this line, an agent asked to
> "clean up the billing code" will happily wade in. One sentence here saves you a bad
> afternoon. *That's* the bar: non-obvious, and it'll bite you if it's missing.
>
> Same with "`pnpm test` needs a local Postgres on 5432 — run `docker compose up -d db`
> first." Claude would discover that by running the tests and watching them fail with a
> connection error, then figuring it out. Or you tell it once, here, for free.

`[BEAT]`

NARRATION:
> Size rule: aim for under 200 lines. Longer files cost more context *and* — this is
> measured, not vibes — the model follows them *less* reliably. A wall of text gets
> skimmed, by models the same as by people. Keep it structured with headers and bullets.
>
> You don't write this from scratch. Run `/init`.

`[DEMO]` In the demo repo session, run `/init`. Let it work. Show the generated file.

NARRATION:
> `/init` reads your codebase and generates a starting `CLAUDE.md` — build commands, test
> setup, conventions it can detect. If one already exists, it suggests improvements
> instead of overwriting. Then you refine it by hand with the stuff it *couldn't* know —
> the billing gotcha, the Postgres dependency.
>
> Two commands to know. `/context` — run it after a session starts and check your
> `CLAUDE.md` actually shows up under "Memory files." If it's not there, Claude can't see
> it. And `/memory` — opens a browser for all your memory files.

`[DEMO]` Run `/context`. Point at the "Memory files" line.

NARRATION:
> When do you add to `CLAUDE.md`? Three triggers. Claude makes the same mistake twice.
> A code review catches something it should've known. Or you catch yourself typing the
> same correction you typed last session. Any of those — that's a line for `CLAUDE.md`.

`[BEAT]`

NARRATION:
> One more tool for big projects: `.claude/rules/`. Same idea as `CLAUDE.md`, but you can
> scope a rule file to file paths.

`[ON SCREEN]` Open [`examples/.claude/rules/testing.md`](../examples/.claude/rules/testing.md).

NARRATION:
> Look at the frontmatter — `paths: "**/*.test.ts"`. This rule only enters context when
> Claude is actually working on a test file. So all your testing conventions — how to name
> a test, use the factories, cover the 400 case — they're there when you're writing tests,
> and they're *not* riding along on the whiteboard when you're doing a database migration.
> That's context you got back for free.

---

## 15:00 — settings.json and the four scopes

`[ON SCREEN]` [`examples/.claude/settings.json`](../examples/.claude/settings.json).

NARRATION:
> `settings.json` is where you change how Claude Code *behaves* — permissions, hooks,
> environment, model. Two quick things about the file itself: it's **strict JSON**, so no
> comments and no trailing commas, a stray one is a syntax error. And keep that `$schema`
> line at the top — it gives you autocomplete and validation right in VS Code.

`[BEAT]`

NARRATION:
> There are four places settings can live, and this trips people up, so here's the mental
> model. **User settings**, in your home directory — that's you, across every project.
> **Shared project settings**, `.claude/settings.json` in the repo — that's committed, so
> it's everyone who clones it. **Project local settings**, `.claude/settings.local.json` —
> that's you, this one project, and it's gitignored. And **managed settings**, which your
> org can push down and which win over everything.
>
> When the same key is set in two places, the more specific one wins — *except* for lists
> like the permission allowlist, which **merge**. So your personal allowlist and the
> project's committed allowlist both apply. That's deliberate: the team can share a
> baseline, and you can add your own safe commands on top without a commit.
>
> And that local file? When Claude Code first writes it in a git repo, it adds it to your
> global git excludes automatically. So the approvals you click "don't ask again" on don't
> end up in a commit.

`[DEMO]` Walk down the `examples/.claude/settings.json` — the `allow` list, the `ask` list,
the `deny` list, the `hooks` block. Don't explain them yet — just orient.

NARRATION:
> We'll come back to what's actually *in* here in the safety section. For now: this is the
> file, these are the four scopes, `/status` tells you which ones loaded.

`[BEAT]`

NARRATION:
> Quick word on `.mcp.json` — that's for MCP servers, external tools like a database
> connection or an issue tracker. Two things. It's committed and shared, like the project
> settings. And — this matters for cost — **every MCP server you enable adds its tool
> definitions to context on every turn.** Two or three focused ones, fine. Ten "might be
> handy someday" servers is a standing tax on every message you send. Enable what the
> project uses; turn the rest off.

---

## 19:00 — Skills, commands, and subagents

`[ON SCREEN]` [`docs/03-skills-commands-subagents.md`](03-skills-commands-subagents.md), the
comparison table.

NARRATION:
> Three ways to extend Claude Code. They look similar and people mush them together, so
> let's be precise. A **skill** is knowledge or a procedure that loads on demand. A
> **slash command** is an action you trigger by name. A **subagent** is a whole separate
> agent with its own context window that goes off, does a job, and reports back.
>
> The biggest setup mistake I see is cramming all of this into `CLAUDE.md`. Let's do it
> right.

### Skills

`[ON SCREEN]` Open [`examples/.claude/skills/pr-description/SKILL.md`](../examples/.claude/skills/pr-description/SKILL.md).

NARRATION:
> A skill is a folder with a `SKILL.md` inside. The folder name becomes the command — this
> one's `pr-description`. Here's *why* skills matter for efficiency: `CLAUDE.md` content
> is on the whiteboard every session forever. A skill's body loads **only when it's used.**
> So a 300-line deployment checklist as a skill costs nothing until the day you deploy. As
> a section of `CLAUDE.md`, it costs on every message for months.
>
> Rule of thumb: if a chunk of your `CLAUDE.md` reads like a *procedure* — step one, step
> two — it should be a skill.

`[DEMO]` Walk through the `SKILL.md`:

NARRATION:
> Look at the pieces. The `description` in the frontmatter is the one field that really
> matters — that's how Claude decides whether to auto-load the skill when it's relevant,
> so it names the trigger phrases: "PR description," "PR summary," "write the PR."
>
> These lines with the backtick-bang — `` !`git diff --staged` `` — that's a shell command
> that runs *before* Claude reads the skill, and its output gets injected right there. So
> when the skill fires, the instructions arrive with the actual diff already inlined. The
> skill is grounded in the real state of the repo, not whatever Claude can guess.
>
> `$1` is an argument — you can call `/pr-description ORDERS-1234` and it drops the ticket
> number in.
>
> And notice `template.md` sits in the same folder but only loads when `SKILL.md` links to
> it. Bulky reference material goes in sibling files so it's there when needed and
> invisible when not — same principle as the scoped rules.

`[DEMO]` In the demo repo: stage a small change (`git add` something), then run
`/pr-description` and let it produce a description.

NARRATION:
> There it is. It ran the diff, read the template, and wrote a real description with a
> risk section. And that whole procedure — the template, the grouping rules, the risk
> checklist — was never in my context until I typed the command.

`[BEAT]`

NARRATION:
> One frontmatter field to call out: `disable-model-invocation: true`. That means *only
> you* can run the skill — Claude won't trigger it on its own. You want that on anything
> with side effects. You do not want Claude deciding on its own that the code "looks
> ready" and running your `/deploy` skill.

### Commands

`[ON SCREEN]` Open [`examples/.claude/commands/new-endpoint.md`](../examples/.claude/commands/new-endpoint.md).

NARRATION:
> Slash commands are the simpler cousin. A file in `.claude/commands/`, and the filename
> becomes the command. This one's `new-endpoint`. Honestly, commands and skills have
> merged under the hood — a command file is just a skill without the folder. Use a plain
> command file when it's a quick one-shot prompt; use a skill when you want supporting
> files or you want Claude to be able to invoke it automatically.
>
> This one takes a resource name and scaffolds a handler, a route registration, and a
> test — all following the patterns already in the repo. When would you make one of these
> instead of just typing the prompt? When you'll run it more than twice, or when the exact
> wording matters and you don't want to re-improvise it each time.

### Subagents

`[ON SCREEN]` Open [`examples/.claude/agents/test-runner.md`](../examples/.claude/agents/test-runner.md).

NARRATION:
> Subagents are different in kind. A subagent runs in its **own** context window. Think of
> it as sending a colleague off to go investigate something and come back with a one-page
> summary — you don't get their entire afternoon of digging, just the conclusion.
>
> This `test-runner` agent runs the suite, reads the failures, works out the cause of
> each, and reports back: what failed, why, smallest fix. Here's the payoff — it might
> read fifteen files and run the tests three times doing that. In a subagent, *your* main
> session never sees any of that. It gets the summary. The noisy middle stays out of your
> whiteboard entirely.
>
> Good candidates: test triage, digging through logs, "search the codebase and tell me
> where we handle X," dependency audits. Anything that's separable and ends in a summary.
>
> The trade-off: a subagent starts *cold*. It doesn't inherit your conversation, so you
> pay a bit to get it the context it needs. Don't delegate something that depends on
> everything you've discussed so far — for that there's a "forked" skill that *does*
> inherit the conversation. But for separable grunt work, a subagent is how you keep a
> long task from bloating your context.

`[ON SCREEN]` The decision tree at the bottom of the doc.

NARRATION:
> So the decision tree. Fact true every session — `CLAUDE.md`. A procedure or reference
> doc — skill. Side effects, and *you* want to control timing — skill with model
> invocation disabled, or a command. Separable work with a summary output — subagent. Must
> happen deterministically every time — and that's the next section.

---

## 27:00 — Permissions and safety

`[ON SCREEN]` [`docs/04-permissions-and-safety.md`](04-permissions-and-safety.md).

NARRATION:
> Two ideas run through this whole section. First: **`CLAUDE.md` is guidance, not
> enforcement.** It shapes what the model tends to do. It does not *guarantee* anything.
> If something truly must or must not happen, you encode it in settings or a hook, which
> the client enforces no matter what the model decides. Second — and this is the part
> people miss — **your allowlist is also a speed and cost tool.** Every permission prompt
> is a round-trip that stalls the agent and costs a turn. Pre-approving your safe, boring
> commands makes sessions smoother *and* cheaper.

`[BEAT]`

NARRATION:
> Permission modes first. You cycle these with Shift-Tab. **Default** prompts for anything
> not on the allowlist. **Plan mode** is read-only — Claude investigates and proposes an
> approach and changes nothing. **Accept-edits** auto-accepts file edits but still asks
> before running commands. And **bypass-permissions** asks for nothing at all.
>
> That last one — bypass — is the dangerous one. It'll run any command without asking:
> `rm -rf`, force-push, pipe-curl-to-shell, anything. Only use it in a container or a VM
> you can throw away. Never on your actual working copy. Some orgs disable it outright, and
> that's a reasonable call.

`[DEMO]` Press Shift-Tab a couple times in the session to show the mode indicator changing.
Land back on default (or plan).

`[ON SCREEN]` The `permissions` block in `examples/.claude/settings.json`.

NARRATION:
> Here's the allowlist from our example. Three lists. **Allow** — runs with no prompt.
> **Ask** — always prompts, even if an allow rule would've matched; it's a deliberate
> speed bump. **Deny** — blocked outright.
>
> Look at what's in `allow`: `pnpm test`, `pnpm lint`, `pnpm typecheck`, `git status`,
> `git diff`. Safe, read-mostly, run fifty times a day. The syntax — `Bash(pnpm test:*)`
> — the `:*` means "with any arguments." So `pnpm test` and `pnpm test src/foo.test.ts`
> both match.
>
> Now look at what's in `ask`: `git push` and `git commit`. Not blocked — we do need them
> — but we want eyes on them every time. And `deny`: reads of `.env`, anything under
> `secrets/`, any `.pem` file. Plus edits to `src/generated/` — that's the code-gen
> directory from our `CLAUDE.md` gotcha, and here we're not just *asking* Claude nicely to
> avoid it, we're making it impossible.

`[BEAT]`

NARRATION:
> The thing to internalize: **what stays off the allowlist.** Anything that changes the
> world outside your working tree. Deploys. `terraform apply`. Database migrations,
> `DROP`, destructive SQL. `rm -rf`, anything with `sudo`. `npx` of a package you don't
> recognize. If it's irreversible or it reaches outside the repo, it should cost a
> deliberate human "yes."

`[ON SCREEN]` The secrets section, then the hooks section.

NARRATION:
> Secrets, quickly: `deny` the reads, so the content can't even reach context indirectly.
> Don't put real credentials in `settings.json` or `CLAUDE.md` or a skill — the `env`
> block in committed settings is for *non-secret* config only. Keep real secrets in your
> shell environment or a secrets manager. And remember gitignored is not encrypted —
> `CLAUDE.local.md` staying out of git doesn't mean it's safe to put your production keys
> in it.

`[BEAT]`

NARRATION:
> Hooks. This is how you handle every "from now on, always do X" or "never let it do Y"
> requirement. A hook is a shell command Claude Code runs at a lifecycle event — before a
> tool call, after a tool call, when a session starts, and so on. It runs regardless of
> what the model intended.
>
> Analogy: putting "please format your code" in `CLAUDE.md` is like a line in the style
> guide. A `PostToolUse` hook that runs Prettier is like a CI check that *fails the build*
> if it's not formatted. One is advice; the other is a fact of the world.

`[DEMO]` Show the `hooks` block in `examples/.claude/settings.json`. Then, in the demo repo,
have Claude make a deliberately badly-formatted edit to a file, and show Prettier fixing it
right after the edit lands.

NARRATION:
> There — Claude wrote the edit, the hook fired on `Edit`, Prettier reformatted it, done.
> I didn't ask, and the model didn't decide to; it just happens now.
>
> The docs also show a `PreToolUse` hook that blocks edits to a protected directory by
> exiting with an error — a `PreToolUse` hook that exits non-zero *vetoes* the tool call.
> That's your "never" enforcement. Note that hooks in a committed settings file run on
> every teammate's machine, so Claude Code makes them wait for the workspace-trust step
> before they'll run.

---

## 34:00 — Credit efficiency: making it go further

`[ON SCREEN]` [`docs/05-credit-efficiency.md`](05-credit-efficiency.md).

NARRATION:
> Okay. This is the section that pays for the rest. Whether you're on the subscription or
> metered API, the same things waste your budget, and now that you've seen the pieces, the
> fixes will make sense.
>
> Recap the cost model. Every turn, the model gets: the system prompt plus every enabled
> MCP server's tool definitions, your `CLAUDE.md` and any active rules, the *entire*
> conversation so far, and your new message. Then it generates a response — and output
> tokens cost more than input, and higher reasoning effort means more of them.
>
> Two consequences. One: **context is cumulative** — long session, everything re-sent
> every turn. Two: **prompt caching helps, but only the stable prefix.** Claude Code
> automatically caches the unchanging front of the request — system prompt, `CLAUDE.md`,
> the early conversation — and cached tokens are much cheaper. But the cache only covers
> the part that doesn't change, and it's *per model*. Switch models mid-task and the next
> request re-reads everything uncached.
>
> So the whole game: keep the prefix small and stable, keep sessions short, don't make the
> model work harder than the task needs.

`[BEAT]`

NARRATION:
> The habits, most important first.
>
> **One: one session per task. `/clear` between unrelated work.** This is the biggest
> lever by far. Do not debug a test failure in the same session where you just spent an
> hour designing a feature. That hour is now riding along on every turn of your debugging.
> Finish a task, `/clear`, start clean.
>
> **Two: use plan mode before anything non-trivial.** Shift-Tab into plan mode, let Claude
> investigate read-only and propose an approach, correct it *before* any code is written.
> Cheap to fix a plan. Expensive to let it build the wrong thing, then re-read the mess,
> then rebuild.
>
> **Three: point at files. Don't make it search from zero.** "The bug's in
> `src/auth/session.ts`, probably `refreshToken`" costs a few hundred tokens. "There's a
> bug somewhere in auth" costs a whole exploration phase — greps, a dozen file reads, dead
> ends — and all of that stays in context for the rest of the session. You usually know
> roughly where things are. Say so.
>
> **Four: give the whole brief up front.** Every clarifying round-trip re-sends the entire
> context. One good paragraph beats five terse messages.
>
> **Five: watch the window.** `/context` shows what's using it. When it's filling up,
> decide deliberately — `/compact` if you still need the thread, `/clear` if you don't.
>
> **Six: right-size the model.** Mechanical edits, small model. Everyday work, mid model.
> Genuinely hard problem, big model or high effort. Set your default to the everyday case.
>
> **Seven: delegate noisy work to a subagent** — like we saw with test-runner. The
> fifteen file reads happen in its context, not yours.
>
> **Eight: let it finish.** Interrupting mid-turn to add "oh also—" throws away partial
> work and re-sends context on the restart.

`[ON SCREEN]` The "worked example" section of the doc — show it side by side if you can.

NARRATION:
> Let me make this concrete. Task: "add rate limiting to the login endpoint."
>
> The wasteful way. You stay in the session where you spent the morning refactoring
> something unrelated — huge context already. You say "add rate limiting somewhere in
> auth." Claude greps the codebase, reads fifteen files to orient. It writes an
> implementation using a library you don't use. You say "no, we use rate-limiter-flexible."
> It re-reads, rewrites. And the entire time, every single turn is re-sending the morning's
> refactor, plus fifteen file reads, plus two implementations of the thing.
>
> The efficient way. `/clear` first. Shift-Tab into plan mode. Then one message: "Add rate
> limiting to the login handler in `src/api/handlers/auth.ts`. Use rate-limiter-flexible —
> it's already a dependency, there's a wrapper in `src/lib/ratelimit.ts`. Five attempts per
> fifteen minutes per IP, return a 429 with our standard error envelope. Add a test."
> Review the plan, approve. Claude edits two files, adds a test, runs it. Done in a handful
> of turns, on a small context.
>
> Same code at the end. A fraction of the tokens. The difference is entirely setup and
> habits — nothing about the task changed.

`[DEMO]` Run `/context` and, if on API, `/cost`. Briefly show the numbers.

NARRATION:
> `/context` for what's in the window, `/cost` for spend this session on the API, `/usage`
> for your subscription window. Glance at these. You'll learn fast which of your workflows
> are the expensive ones.

---

## 40:00 — Putting it together

`[ON SCREEN]` The "appropriate setup checklist" in the [README](../README.md).

NARRATION:
> Let's tie it off. "An appropriate setup" just means three things: it's scoped to the
> work, it's safe by default, and it's cheap to run. Here's the checklist — it's in the
> repo.
>
> Machine: CLI and extension installed and connected, auth path chosen deliberately,
> default model set for the common case.
>
> Scaffolding: a `CLAUDE.md` under 200 lines with only always-true facts; a committed
> `settings.json` with an allowlist for your safe commands; procedures living in skills,
> not `CLAUDE.md`; only the MCP servers you actually use.
>
> Safety: `deny` on secret files; destructive commands off the allowlist; anything that
> must happen every time is a hook; bypass mode stays off.
>
> Running: new session per task, plan mode for real changes, point at files, check
> `/context` when it feels heavy.

`[BEAT]`

NARRATION:
> You don't have to do all of this on day one. The realistic path: run `/init`, add a
> `deny` for your `.env`, allowlist your test command. That's ten minutes and it's already
> a better setup than most people have. Then, over the next few weeks, every time you
> catch yourself re-explaining something — that's a `CLAUDE.md` line or a skill. Every time
> a prompt annoys you — that's an allowlist entry. It compounds.
>
> The repo has all of this written up, plus an `examples/` folder you can copy straight
> into a project and trim. Link's in the description. Go set up one repo properly this
> week — pick the one you work in most — and you'll feel the difference immediately.
>
> Thanks for watching.

`[ON SCREEN]` Repo URL / end card.

---

## Appendix A — lines worth landing cleanly

If you re-record any part, these are the sentences that carry the tutorial. Get them
crisp:

- "Every turn, the model is re-sent the entire conversation so far."
- "A good setup is an exercise in what goes on the whiteboard, and when."
- "`CLAUDE.md` size is a tax you pay on every message for the life of the project."
- "If a chunk of your `CLAUDE.md` reads like a procedure, it should be a skill."
- "`CLAUDE.md` is guidance, not enforcement. If it must happen, it's a hook."
- "Your allowlist is also a speed and cost tool."
- "Keep the prefix small and stable, keep sessions short, don't make the model work
  harder than the task needs."
- "Same code at the end. A fraction of the tokens."

## Appendix B — shot list / B-roll

- The four-bucket diagram (slide or drawn).
- `npm install -g @anthropic-ai/claude-code` in a terminal.
- VS Code Extensions panel with "Claude Code" installed; `/ide` output.
- `/init` running and the generated `CLAUDE.md`.
- `/context` output with "Memory files" highlighted.
- Scroll of `examples/CLAUDE.md`, holding on "Gotchas."
- `examples/.claude/rules/testing.md` frontmatter.
- `examples/.claude/settings.json` permissions block.
- `/pr-description` running end to end on a staged change.
- `examples/.claude/agents/test-runner.md`.
- Shift-Tab cycling the permission mode indicator.
- A Prettier hook reformatting a file right after an edit.
- `/context` and `/cost` numbers.
- The README checklist.

## Appendix C — if you only have 15 minutes

Cut to: the mental model (§1), `CLAUDE.md` what-goes-in / what-doesn't (§ first half of the
CLAUDE.md section) with the gotchas example, the skills-vs-commands-vs-subagents table and
the `/pr-description` demo, the allowlist + `bypassPermissions` warning, and the credit-
efficiency habits list with the rate-limiting worked example. Skip prerequisites (link the
doc), `.mcp.json`, rules, and hooks detail.
