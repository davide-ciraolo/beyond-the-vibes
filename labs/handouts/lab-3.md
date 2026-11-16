# Lab 3 · Teach Taskboard's agent new tricks

**Time:** 40 minutes · **Start tag:** `lab-3-start` · **Module:** M5 Extending Claude

## Goal

Give Claude three new capabilities in the Taskboard repo:

- the course's **core toolkit** of plugins;
- a **browser**, through the Playwright MCP server;
- your team's **know-how**, as a project Skill `add-endpoint` that fires on its own when you ask for an endpoint in plain words.

## Prerequisites

- Lab 0 done: the starter cloned, both suites run, and Claude Code at least v2.1.287 ⚠ (`claude --version`, `claude update`).
- Node 18+ for Playwright MCP (Lab 0 already installs Node ≥ 22.13).
- You're logged in with your subscription (`claude auth status --text`).
- Labs 1–2 are optional: `lab-3-start` already holds their reference solutions (CLAUDE.md, permissions, due dates and the overdue filter).
- Read the **Overdue** rule in `README.md`. The new endpoint reuses it.

## Steps

1. **Start from the tag and check the baseline.** From the repo root:

   ```bash
   git switch -c lab-3 lab-3-start
   uv sync --directory api
   cd web
   npm install    # inside web/: older npm ignores --prefix for install
   cd ..
   uv run --directory api pytest -q   # 10 passed
   npm --prefix web test              # 12 passed
   ```

2. **Install the core toolkit.** Start `claude` in the repo and run these in the session ⚠:

   ```text
   /plugin install superpowers@claude-plugins-official
   /plugin install security-guidance@claude-plugins-official
   /plugin install context7@claude-plugins-official
   /plugin install skill-creator@claude-plugins-official
   /reload-plugins
   ```

   `/code-review` is bundled: there's nothing to install. Each `/plugin install` opens a details pane where you pick the scope.

   <details>
   <summary>Shell equivalents, and a machine that has never run an interactive session</summary>

   ```bash
   # Only on a machine where no interactive session has run yet:
   claude plugin marketplace add anthropics/claude-plugins-official

   claude plugin install superpowers@claude-plugins-official
   claude plugin install security-guidance@claude-plugins-official   # needs Python 3.8+
   claude plugin install context7@claude-plugins-official
   claude plugin install skill-creator@claude-plugins-official
   claude plugin list                                                # Version / Scope / Status
   ```

   Alternative for superpowers (the author's marketplace, latest release): `/plugin marketplace add obra/superpowers-marketplace`, then `/plugin install superpowers@superpowers-marketplace`.
   </details>

3. **Add Playwright MCP at project scope.** Exit Claude (or use a second terminal) and run from the repo root ⚠:

   ```bash
   # macOS, Linux, WSL, Git Bash
   claude mcp add --scope project playwright -- npx @playwright/mcp@latest

   # Native Windows (PowerShell or CMD), if the line above fails to connect ⚠
   claude mcp add --scope project playwright -- cmd /c npx @playwright/mcp@latest

   claude mcp list   # expect: playwright ... ✔ Connected
   ```

   Everything after `--` is the server's own command line. `--scope project` writes `.mcp.json` at the repo root so the team gets it through git. Restart `claude`, approve the project server once, then run `/mcp` and check that `playwright` shows `✔ Connected`.

4. **Write the Skill.** Ask Claude (or run `/skill-creator` and describe the same thing):

   > Create a project Skill in `.claude/skills/add-endpoint/`. `SKILL.md` must have `name: add-endpoint` and a description that says what it does and when to use it: whenever someone asks to add, create or expose an API endpoint or route. Steps: failing test first in `api/tests/`, models in `api/app/schemas.py`, SQL in `api/app/repo.py`, a thin handler in `api/app/routes/`, mirror the types in `web/src/api.ts`, then run `uv run --directory api pytest -q` until green. Put naming and status-code rules in a separate `conventions.md` and link it from `SKILL.md`.

   Review both files against the model answers below. Then check that the Skill loaded: run `/skills` (type to filter), and ask:

   > What skills are available?

   <details>
   <summary>Model answer: <code>.claude/skills/add-endpoint/SKILL.md</code></summary>

   ```markdown
   ---
   name: add-endpoint
   description: Adds a new REST endpoint to the Taskboard FastAPI app (route, schema, repo function, tests) following project conventions. Use when the user asks to add, create or expose an API endpoint or route.
   argument-hint: "[METHOD /path] [short purpose]"
   ---

   # Add an API endpoint

   Add the endpoint described in: $ARGUMENTS

   ## Steps
   1. Write a failing test first in `api/tests/` that calls the new endpoint.
   2. Add or extend the Pydantic models in `api/app/schemas.py`.
   3. Put DB access in `api/app/repo.py`; keep the handler in `api/app/routes/` thin.
   4. Register the route and mirror the types in `web/src/api.ts`.
   5. Run `uv run --directory api pytest -q` and fix until green.

   ## Rules
   - Return 404 for missing resources and 422 for validation errors, never 500.
   - See [conventions.md](conventions.md) for naming and status-code details.
   ```

   - `name` must equal the folder name.
   - The `description` decides *when* the Skill fires; the body decides *what* it does.
   - `$ARGUMENTS` receives the text after `/add-endpoint` when you call it by name.
   - `argument-hint` is Claude Code-only: remove it before uploading the Skill to claude.ai or the Skills API.
   </details>

   <details>
   <summary>Model answer: <code>.claude/skills/add-endpoint/conventions.md</code></summary>

   ```markdown
   # Taskboard API conventions

   - Paths are plural nouns: `/tasks`, `/tasks/{task_id}`. Fixed sub-paths such as `/tasks/stats` are declared **before** `/tasks/{task_id}`, or FastAPI tries to parse "stats" as an id and returns 422.
   - Handlers return Pydantic models from `schemas.py`, never dicts or rows.
   - Status codes: 200 read/update, 201 create, 204 delete, 404 missing, 422 invalid input.
   - SQL lives only in `repo.py` and uses `?` placeholders; never build SQL from user input.
   - Dates are ISO `YYYY-MM-DD` strings in SQLite and `date` in Python; "today" comes from `app.clock.today_utc()`.
   - Tests use the `client` fixture (temporary database) and patch `app.clock.today_utc` when dates matter.
   ```

   Claude reads this file only when the Skill needs it (progressive disclosure), so detail here costs no context until then.
   </details>

5. **Trigger the Skill with a plain request.** No slash command. Ask:

   > Add an endpoint that returns how many tasks are total, done and overdue.

   Watch the transcript: Claude should load `add-endpoint` before it edits anything, then follow its steps. Check the result:
   - A failing test comes first (for example `api/tests/test_stats.py`), and you see it fail.
   - `GET /tasks/stats` returns `{"total": …, "done": …, "overdue": …}` and applies the README overdue rule.
   - The route is declared before `/{task_id}`; SQL is in `repo.py` with `?` placeholders.
   - `web/src/api.ts` gains a matching `TaskStats` type.
   - Both suites pass (reference: API 11 passed, web 12 passed).

6. **Check the UI through Playwright.** Start both dev servers in two terminals from the repo root:

   ```bash
   uv run --directory api fastapi dev app/main.py   # http://127.0.0.1:8000
   npm --prefix web run dev                         # http://localhost:5173
   ```

   Port 8000 busy? Use `--port 8001` on the API line and `API_PORT=8001 npm --prefix web run dev` for the page (see Troubleshooting).

   Then ask Claude:

   > Use Playwright to open the page at http://localhost:5173, add a task with yesterday's date, and check it shows as overdue. Then tick "Overdue only" and confirm the task is still listed. Take a snapshot after each step and report any console errors.

   A browser window opens (headed is the default) and Claude reports what it saw. Approve the MCP tool calls when asked.

7. **Review and commit.** Read `git diff`, then commit the Skill, `.mcp.json` and the endpoint on your `lab-3` branch. Stop the dev servers.

## Done when

- The `add-endpoint` Skill loads from the plain request in step 5, with no slash command.
- The UI check in step 6 runs through Playwright MCP and reports the task as overdue.

## Stretch goals

- **Package the Skill as a plugin** `taskboard-toolkit`. Create `taskboard-toolkit/.claude-plugin/plugin.json` (only `name` is required) and move the Skill to `taskboard-toolkit/skills/add-endpoint/`. The folders sit next to `.claude-plugin/`, not inside it.

  ```json
  {
    "name": "taskboard-toolkit",
    "version": "0.1.0",
    "description": "Taskboard team skills (add-endpoint)"
  }
  ```

  ```bash
  claude plugin validate ./taskboard-toolkit   # Validation passed / with warnings / failed
  claude --plugin-dir ./taskboard-toolkit      # load for one session; the Skill runs as /taskboard-toolkit:add-endpoint
  ```

- **Tune the trigger** with `/skill-creator`: give it should-trigger prompts ("expose the stats as a route") and should-not-trigger prompts ("fix the CSS"), and let it adjust the description.
- **Call it by name:** `/add-endpoint POST /tasks/{id}/archive archive a task`. `$ARGUMENTS` receives the text.
- **Lock Playwright down** to the dev servers ⚠:

  ```bash
  claude mcp add --scope project playwright -- npx @playwright/mcp@latest --headless \
    --allowed-origins "http://localhost:5173;http://localhost:8000"
  ```

  (Run `claude mcp remove playwright` first. If you moved the API to port 8001, use `http://localhost:8001` in the list so the origins match your dev servers. Playwright MCP is "not a security boundary", and by default it allows every origin.)
- **Check the context cost:** `/context` shows the MCP tools as `(deferred)` and a Skills row; `/skills` sorts by token count when you press `t`.

## Troubleshooting

- **The Skill doesn't fire.**
  - Is it listed in `/skills`? If not, check the path `.claude/skills/add-endpoint/SKILL.md` and that `name` equals the folder name.
  - Put the words people actually say ("add", "create", "expose", "endpoint", "route") into the `description`.
  - Bad YAML frontmatter still lets `/add-endpoint` work, but Claude can't match the description. `claude --debug` shows the parse error; `claude plugin validate .claude/skills` finds broken frontmatter (v2.1.233+) ⚠.
  - Edits apply without a restart; `/reload-skills` forces a re-scan.
- **MCP fails to start on native Windows** (`✘ Failed to connect`): `claude mcp remove playwright`, then add it again with the `cmd /c npx` form from step 3 ⚠. `/mcp reconnect all` retries failed servers (v2.1.284+) ⚠.
- **`⏸ Pending approval` in `/mcp`:** project servers need a one-time approval after you trust the workspace. Restart `claude` and approve. `claude mcp reset-project-choices` asks again.
- **No browser opens, or the first call is slow.** `npx` downloads `@playwright/mcp` on first use, which is slow behind a proxy; check `node --version` (18+). To use a browser you already have, add `--browser chrome` or `--browser msedge` after `@playwright/mcp@latest`. On WSL or a machine without a display, add `--headless`.
- **Playwright sees an empty page or errors on add:** both dev servers must be running; the page proxies `/api` to port 8000.
- **Port 8000 already in use (Docker Desktop and WSL often hold it):** run `uv run --directory api fastapi dev app/main.py --port 8001` and `API_PORT=8001 npm --prefix web run dev`. The page stays on http://localhost:5173, so the Playwright prompt doesn't change.
- **`GET /tasks/stats` returns 422:** the route is declared after `/tasks/{task_id}`, so FastAPI parses "stats" as an id. Move it above (this is why `conventions.md` says so).
- **A plugin doesn't seem active:** run `/reload-plugins`, then check `/plugin` (Installed tab) or `claude plugin list`. security-guidance needs Python 3.8+, and its LLM reviews cost tokens.
