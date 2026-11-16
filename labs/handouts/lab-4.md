# Lab 4 · Ship a feature through brainstorm, plan, subagents and review

**40 minutes · start tag `lab-4-start` · Day 2, after M6**

> **Minute-15 rule.** Still brainstorming at minute 15? Stop. Use the pre-written spec
> **`docs/superpowers/specs/lab-4-tags-design.md`** (it ships with `lab-4-start`) and go straight to step 5, the plan.

## Goal

Write a read-only `security-reviewer` subagent by hand. Then take one small feature, **task tags with a tag filter**, through the superpowers flow: brainstorming → design spec → plan (2–3 tasks) → subagent-driven development. Review the result with your subagent and `/code-review`, fix the findings, and merge.

## Prerequisites

- Lab 0 done: `origin` points to your own GitHub repo.
- Lab 3 done: the core toolkit is installed, including the **superpowers** plugin. Check that `/superpowers:` shows up in the `/` menu.
- Claude Code ≥ v2.1.287 ⚠ verify before delivery (`claude --version`).
- Missed Lab 3? Starting from `lab-4-start` is enough; it already contains the Lab 3 reference.

## Steps

1. **Start from the tag and check the suites.**

   ```bash
   git switch -c lab-4 lab-4-start
   uv run --directory api pytest -q
   npm --prefix web test
   ```

   Expected: `11 passed` (API) and `12 passed` (web).

2. **Write the subagent.** Create `.claude/agents/security-reviewer.md` by hand, or ask Claude to write it. The `/agents` wizard is gone: `/agents` now only prints a reminder ⚠ verify before delivery.

   > Write `.claude/agents/security-reviewer.md`: a read-only security reviewer for this FastAPI + TypeScript app. Tools Read, Grep, Glob, Bash only; model sonnet; effort high. Checklists for FastAPI/Python and the TypeScript UI; output ordered by severity with file:line and a concrete fix.

   Check it against the model answer below. What matters: the `tools` line has **no Write or Edit** (so it can't edit files) and **no `Agent`** (so it can't nest). Bash calls still go through your permission rules.

   <details>
   <summary>Model answer: <code>.claude/agents/security-reviewer.md</code></summary>

   ````markdown
   ---
   name: security-reviewer
   description: Security reviewer for the Taskboard FastAPI + TypeScript app. Use proactively after changes to api/ or web/, and before /code-review or opening a PR. Read-only; reports findings, never edits.
   tools: Read, Grep, Glob, Bash
   model: sonnet
   effort: high
   color: red
   ---

   You are a senior application-security reviewer for a FastAPI (Python) API in `api/`
   and a Vite + TypeScript UI in `web/`. You review; you never modify files.

   ## Scope
   1. Run `git diff --stat` and `git diff` (or the range the caller gives you) to see what changed.
   2. Read each changed file in full, plus the code it calls when a risk depends on it.
   3. Use Bash only for read-only commands: `git diff`, `git log`, `git show`, `ls`.
      Never run installers, servers, network calls, or anything that writes.

   ## Checklist: FastAPI / Python
   - Injection: raw SQL built with f-strings or `%`; use bound parameters. Also `subprocess` with `shell=True`, `eval`/`exec`, `pickle`/`yaml.load` on input.
   - AuthN/AuthZ: every route that reads or changes data has the auth dependency;
     object-level checks (a user can't fetch or delete another user's task by id).
   - Validation: request bodies use Pydantic models with constraints; no `dict`/`Any` passthrough;
     response models don't leak internal fields (password hashes, tokens).
   - Secrets: no keys, tokens, or passwords in code, tests, or logs; config comes from env.
     Never read or print `.env`.
   - CORS: no `allow_origins=["*"]` together with `allow_credentials=True`.
   - Errors: no stack traces or SQL errors returned to clients; `debug` is not on.
   - Files and paths: user input never reaches `open()`/paths without normalising them (path traversal).

   ## Checklist: TypeScript UI
   - XSS: user data assigned to `innerHTML`/`outerHTML`/`insertAdjacentHTML` or built into HTML strings;
     prefer `textContent` and DOM APIs.
   - Tokens in `localStorage`; secrets or API keys bundled into front-end code or `import.meta.env`.
   - `fetch` calls send credentials only to the API origin; no `eval`/`new Function`.

   ## Output
   Return a list ordered by severity (Critical, High, Medium, Low). For each finding give:
   `file:line` · category · what an attacker could do · a concrete fix (code sketch is fine).
   Report only issues you can point to in the code; say "Not verifiable from diff" when you can't confirm.
   If nothing is found, say so and list what you checked. Keep the report under 60 lines.
   ````

   </details>

3. **Restart Claude** in the repo (or start it, if you wrote the file by hand) so it loads the new agent. Commit the file on `lab-4` once you've reviewed it.

   ```bash
   claude
   ```

4. **Brainstorm the feature.** Use a natural request so the superpowers `brainstorming` skill triggers on its own:

   > Let's brainstorm task tags with a tag filter. Follow the tag rules in README.md.

   If the skill doesn't trigger, invoke it directly: `/superpowers:brainstorming` ⚠ verify before delivery (the name depends on how the plugin was installed; check the `/` menu). Answer its questions one by one. Keep the scope small: tags on tasks, filter by one tag, tests on both sides. It writes a design spec to `docs/superpowers/specs/<date>-<topic>-design.md`.

   > **⏱ Minute 15 and still brainstorming?** Stop the session and switch to the pre-written spec:
   >
   > > Use docs/superpowers/specs/lab-4-tags-design.md as the approved spec and write the implementation plan.
   >
   > This is the planned path, not a failure: the point of the lab is the rest of the flow.

5. **Plan with 2–3 tasks.** Brainstorming hands off to `writing-plans` ⚠. Review the plan before you approve it. It should look like this:
   1. API: `tags: list[str]` on create, update and read; `GET /tasks?tag=<name>` (exact match); pytest for round-trip, validation and filtering.
   2. Web: tags input parsed by `parseTags`, chips, a `#tag-filter` select; Vitest.
   3. Optional: an end-to-end check through Playwright MCP.

   > The plan has more than three tasks. Merge them into: (1) API with pytest, (2) web with Vitest, (3) optional Playwright check.

6. **Run subagent-driven development.** Choose subagent-driven execution when asked, or invoke it: `/superpowers:subagent-driven-development` ⚠. What to watch for:
   - It works in an isolated git worktree and keeps a ledger (`progress.md`).
   - Per task: a fresh implementer subagent (test, implement, commit), then one task reviewer with two verdicts: spec compliance, then code quality. Fixes loop with a cap.
   - It runs without check-ins. It stops only for destructive or security-sensitive actions, side effects outside the worktree (such as a merge), or a broken plan.
   - Background subagents: Ctrl+B and `/tasks` show what's running ⚠.

7. **Review with your subagent, then `/code-review`.**

   > @agent-security-reviewer review the branch diff

   Only its summary lands in your main context. Then run the bundled correctness review:

   ```text
   /code-review high
   ```

   It runs as a background forked subagent; the findings arrive when it finishes ⚠.

8. **Address the findings and merge into `lab-4`.** Decide on each finding yourself: fix it, or say why not.

   > Fix the High and Medium findings, each with a failing test first. Leave the Low ones in a list for me.

   Let `finishing-a-development-branch` merge the work into `lab-4` (it asks before a merge). Then, from the repo root:

   ```bash
   uv run --directory api pytest -q
   npm --prefix web test
   ```

   The reference solution (`lab-5-start`) ends at `16 passed` on each side; your counts may differ.

## Done when

- The `security-reviewer` and `/code-review` findings are fixed or consciously dismissed.
- The tags feature is merged into `lab-4`, and both suites pass.

## Stretch goals

- `/security-review`: vulnerability review of your branch against `origin`'s default branch (needs the `origin` remote from Lab 0) ⚠.
- Add `memory: project` to the subagent so it builds knowledge across sessions in `.claude/agent-memory/security-reviewer/`.
- Run a whole session as the reviewer: `claude --agent security-reviewer`.
- Compare with the reference: `git diff lab-4 lab-5-start`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `/superpowers:brainstorming` not in the `/` menu, or names differ | The plugin's marketplace entry may lag superpowers v7.0.2 ⚠ verify before delivery. Check the `/` menu for the installed names; reinstall from Lab 3 if superpowers is missing. |
| The subagent says it can't edit a file | By design: its `tools` list has no Write or Edit. Fix findings in the main session. |
| `@agent-security-reviewer` isn't found | Check the path `.claude/agents/security-reviewer.md` and the `name:` line, then restart `claude`. |
| Brainstorming keeps asking questions | Minute-15 rule: switch to `docs/superpowers/specs/lab-4-tags-design.md`. |
| `no such column: tags` | An old database: delete `api/taskboard.db`. |
| `/code-review` uses an unexpected level | With no level typed it reuses the last one you typed. Type `high` explicitly. |
| `/security-review` fails | It needs an `origin` remote: `git remote -v` (Lab 0). |
