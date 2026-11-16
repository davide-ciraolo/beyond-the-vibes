# Lab 1 · Make Claude productive in Manual mode

**Time:** 30 minutes · **Start tag:** `lab-1-start` · **Module:** M3 Claude Code Fundamentals

## Goal

Get the Taskboard starter into your own private GitHub repo, then give it a lean `CLAUDE.md` and a shared `.claude/settings.json`, so that Claude can run both test suites in Manual mode without asking you first. The allow rules should be what removes the prompts, not auto mode.

## Prerequisites

- Lab 0 is done: `node -v` prints `v24.…`, Claude Code is v2.1.287 or later ⚠, and `claude auth status --text` shows your account.
- `taskboard.bundle`: download it from the `labs/` folder of [github.com/davide-ciraolo/beyond-the-vibes](https://github.com/davide-ciraolo/beyond-the-vibes).

## Steps

### Part A · Get the starter

A1. **Clone the bundle and push it to a new private repo.** In the folder where you saved `taskboard.bundle`:

   ```bash
   git clone taskboard.bundle taskboard
   cd taskboard
   git remote rename origin course
   gh repo create taskboard --private --source . --remote origin   # or create it on github.com and: git remote add origin <url>
   git push -u origin main --tags
   ```

   `course` now points at the bundle, and `origin` at your GitHub repo. The `--tags` push matters: every lab starts from a tag (`lab-1-start`, `lab-2-start`, …). Lab 5 opens a pull request in this repo.

A2. **Install the dependencies,** from the repo root:

   ```bash
   uv sync --directory api
   cd web
   npm install    # inside web/: older npm ignores --prefix for install
   cd ..
   ```

A3. **Run both test suites,** also from the repo root:

   ```bash
   uv run --directory api pytest -q
   npm --prefix web test
   ```

   Each suite should report `6 passed`. Check `git remote -v` too: `origin` must point to your GitHub repo. From here on, every command runs from the repo root.

### Part B · CLAUDE.md and permissions

1. **Create a branch at the start tag.** Use a branch, not a detached HEAD.

   ```bash
   git switch -c lab-1 lab-1-start
   ```

   `lab-1-start` has no `CLAUDE.md` and no `.claude/`. That's expected.

2. **Start Claude in Manual mode.**

   ```bash
   claude --permission-mode default
   ```

   Auto mode is the default since v2.1.283 ⚠. In auto mode, "no permission prompts" is true without any setup, which would defeat the point of this lab. Check that the status bar shows `⏸ manual mode on`. If you leave Manual mode by accident, press Shift+Tab (Alt+M on some Windows terminals) until you're back.

3. **Generate a first draft, then trim it.** Run:

   ```text
   /init
   ```

   Approve the file write when Claude asks. Then prompt:

   > Trim CLAUDE.md to under 40 lines: keep commands, architecture and workflow rules; drop anything the code already says.

4. **Use the root-relative test commands.** Open `CLAUDE.md` and check that the test commands are exactly these:

   ```bash
   uv run --directory api pytest -q
   npm --prefix web test
   ```

   If `/init` wrote `cd api && …` or `cd web && npm test`, prompt:

   > In CLAUDE.md, list the test commands as `uv run --directory api pytest -q` and `npm --prefix web test`, run from the repo root, never with `cd`.

   You need this because an allow rule matches a command, and a `cd web && …` compound needs every part allowed (see Troubleshooting). Also check that the workflow rules include "never read `.env`" and "don't commit; propose a message".

   <details><summary>Model answer: CLAUDE.md</summary>

   ```markdown
   # Taskboard

   Task CRUD app: FastAPI + SQLite API in `api/`, Vite + TypeScript (no framework) UI in `web/`.

   ## Commands (run from the repo root)
   - API tests: `uv run --directory api pytest -q`
   - Web tests: `npm --prefix web test`
   - API dev server: `uv run --directory api fastapi dev app/main.py`
   - Web dev server: `npm --prefix web run dev`
   - Add a Python dependency: `uv add --directory api <pkg>` (never pip install)

   ## Architecture
   - `api/app/routes/` holds HTTP handlers only; all SQL lives in `api/app/repo.py`, with bound parameters.
   - Pydantic models in `api/app/schemas.py` are the API contract; `web/src/api.ts` mirrors them.
   - The UI renders with `textContent`, never `innerHTML`.

   ## Conventions
   - Python: type hints everywhere, 4-space indent.
   - TypeScript: `strict` mode, no `any`, 2-space indent, named exports.
   - Every bug fix starts with a failing test.

   ## Workflow
   - Run the relevant test suite before saying a task is done, and show its output.
   - Never read or print `.env`; ask for config values instead.
   - Don't commit; propose a commit message and wait for review.

   See @README.md for setup and the lab feature rules.
   ```

   </details>

5. **Have Claude write the permission rules.** Prompt:

   > Create `.claude/settings.json` with the `$schema` line and permission rules: allow `uv run --directory api pytest *` and `npm --prefix web test *` as Bash rules; ask before `git push *`; deny reading `.env`, `.env.*` and anything under `secrets/`.

   Review the file before you approve the write. Rules in `.claude/settings.json` are committed and shared with the team. Rules you accept with "Yes, and don't ask again" go to `.claude/settings.local.json`, which stays on your machine.

   <details><summary>Model answer: .claude/settings.json</summary>

   ```json
   {
     "$schema": "https://json.schemastore.org/claude-code-settings.json",
     "permissions": {
       "allow": [
         "Bash(uv run --directory api pytest *)",
         "Bash(npm --prefix web test *)"
       ],
       "ask": ["Bash(git push *)"],
       "deny": ["Read(./.env)", "Read(./.env.*)", "Read(**/.env)", "Read(./secrets/**)"]
     }
   }
   ```

   A trailing ` *` also matches the bare command, so `Bash(npm --prefix web test *)` covers `npm --prefix web test` too.

   </details>

6. **Restart in Manual mode and run both suites.** Exit Claude (Ctrl+C twice), then:

   ```bash
   claude --permission-mode default
   ```

   > Run both test suites

   Both suites should run with no permission prompt (6 passed each at `lab-1-start`).

7. **Check where the rules come from.**

   ```text
   /permissions
   ```

   Both allow rules, the `git push` ask rule and the four deny rules should be listed from `.claude/settings.json`.

## Done when

- `git remote -v` shows `origin` pointing to your GitHub repo, and both suites passed in Part A.
- The status bar shows Manual mode, and Claude runs `uv run --directory api pytest -q` and `npm --prefix web test` with no permission prompt.
- `/permissions` shows that the rules come from `.claude/settings.json`, not only from `.claude/settings.local.json`.

## Stretch goals

- **Path-scoped rule:** ask Claude to move the TypeScript conventions out of `CLAUDE.md` into a rule under `.claude/rules/` that loads only for files in `web/`. Rules in `.claude/rules/*.md` load only when Claude works on matching files.
- **Personal notes:** create `CLAUDE.local.md` for things only you need (for example your preferred answer style), and add it to `.gitignore` yourself. `/memory` lists it next to `CLAUDE.md`.
- Try `/context all` to see which memory files loaded.

## Troubleshooting

- **`npm ERR! enoent … taskboard\package.json`:** you ran `npm --prefix web install` from the repo root. npm 10 and early npm 11 ignore `--prefix` for `install`. Run `cd web`, `npm install`, `cd ..` as in step A2.
- **`Cannot find native binding` or `Cannot find module '@rolldown/binding-…'` when running the web tests:** you're not on Node 24. Run `nvm use 24` (see Lab 0, step 4), delete the `web/node_modules` folder (keep `package-lock.json`), then run `npm install` in `web/` again.
- **Corporate proxy: `npm install` in `web/` hangs or fails with certificate errors:** npm needs your company's proxy settings. Ask IT for the proxy and certificate details for npm, or use a network without the proxy.
- **`gh repo create` fails:** run `gh auth login` first, or create an empty private repo on github.com and use `git remote add origin <url>`.
- **Odd API test failures, or `no such column` errors:** an old `api/taskboard.db` from an earlier run is in the way. Delete it and run the tests again.
- **A test command still prompts:** look for `cd web && npm test` or `cd api && …`. Compound commands (`&&`, `;`, `|`) need every part allowed, so the rule doesn't match. Use the `--prefix` / `--directory` forms, and make sure `CLAUDE.md` lists them that way.
- **Allow rules seem ignored:** project allow rules apply only after you accept the workspace trust dialog. Deny and ask rules apply even before that. Restart `claude` in the repo root and accept the dialog.
- **No prompts at all, even before you wrote settings.json:** you're probably in auto mode ⚠. Restart with `claude --permission-mode default` and check the status bar.
- **It works, but only because of `settings.local.json`:** if you chose "Yes, and don't ask again" earlier, the rule landed in `.claude/settings.local.json`. `/permissions` shows which file each rule is in. Remove the local entry and check that the shared file is enough on its own.
- **`defaultMode: auto` in project settings does nothing:** that's by design. Project settings files ignore `auto` and `bypassPermissions` ⚠.
- **Shift+Tab doesn't switch modes (Windows):** use Alt+M.
