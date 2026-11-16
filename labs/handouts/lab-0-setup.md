# Lab 0 · Setup (before the course)

About 20 minutes. Do this before Day 1 so we can start coding straight away.

## Goal

Install Claude Code and the tools the labs use. Get the Taskboard starter into your own private GitHub repo, run both test suites, and have a first short conversation with `claude` in the repo.

## Prerequisites

- A Claude account you can sign in with (the one you will use in the course).
- A GitHub account.
- macOS 13+, Windows 10 1809+, or Linux (for example Ubuntu 20.04+). 4 GB+ RAM. ⚠
- `taskboard.bundle`: download it from the `labs/` folder of [github.com/davide-ciraolo/beyond-the-vibes](https://github.com/davide-ciraolo/beyond-the-vibes).

## Steps

### Install

1. **Claude Code** (native installer, recommended; no admin rights needed). Run the line for your shell. ⚠

   macOS, Linux, WSL:

   ```bash
   curl -fsSL https://claude.ai/install.sh | bash
   ```

   Windows PowerShell (prompt shows `PS C:\`):

   ```powershell
   irm https://claude.ai/install.ps1 | iex
   ```

   Windows CMD (prompt shows `C:\` without `PS`):

   ```bat
   curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
   ```

   The installer shows no progress while it downloads. When it finishes, **open a new terminal**.

   Prefer a package manager? `brew install --cask claude-code` (macOS) or `winget install Anthropic.ClaudeCode` (Windows). These don't auto-update, so upgrade them yourself with `brew upgrade claude-code` or `winget upgrade Anthropic.ClaudeCode`. ⚠

2. **Windows only: Git for Windows** ([git-scm.com](https://git-scm.com/)). It gives Claude Code Git Bash for its Bash tool. Without it, Claude Code falls back to PowerShell. If you work in WSL, install and run everything inside the WSL terminal instead.

3. **The other tools:**
   - git (included in Git for Windows)
   - Python ≥ 3.11 and [uv](https://docs.astral.sh/uv/)
   - Node ≥ 22 ([nodejs.org](https://nodejs.org/))
   - [GitHub CLI](https://cli.github.com/) `gh`, signed in with `gh auth login` (optional: you can create the repo on github.com instead)
   - `jq` (used by hook examples on Day 2):

   ```bash
   brew install jq            # macOS
   sudo apt install jq        # Debian / Ubuntu / WSL
   winget install jqlang.jq   # Windows
   ```

### Update and check

4. Check the version, update, and run the diagnostics. You need **v2.1.287 or later**. ⚠

   ```bash
   claude --version
   claude update
   claude doctor
   ```

5. Check which account Claude Code will use:

   ```bash
   claude auth status --text
   ```

   It must show the account you expect. If you haven't signed in yet, run `claude auth login` (or just start `claude` in step 10; it opens the browser login).

   If it shows an **API key** instead of your account, a stray `ANTHROPIC_API_KEY` in your shell is winning over your login. Remove it for this shell, then check again:

   ```bash
   unset ANTHROPIC_API_KEY
   ```

   ```powershell
   Remove-Item Env:ANTHROPIC_API_KEY
   ```

   If it comes back in new terminals, remove it from your shell profile or system environment variables too.

### Get the starter

6. In the folder where you saved `taskboard.bundle`, clone it and push it to a new **private** repo of your own:

   ```bash
   git clone taskboard.bundle taskboard
   cd taskboard
   git remote rename origin course
   gh repo create taskboard --private --source . --remote origin   # or create it on github.com and: git remote add origin <url>
   git push -u origin main --tags
   ```

   `course` now points at the bundle, and `origin` at your GitHub repo. The `--tags` push matters: every lab starts from a tag (`lab-1-start`, `lab-2-start`, …). Lab 5 opens a pull request in this repo.

### Smoke test

7. Install the dependencies, from the repo root:

   ```bash
   uv sync --directory api
   npm --prefix web install
   ```

8. Run both test suites, also from the repo root:

   ```bash
   uv run --directory api pytest -q
   npm --prefix web test
   ```

   Each suite should report `6 passed`.

9. Check the remotes:

   ```bash
   git remote -v
   ```

10. Start Claude Code in the repo:

    ```bash
    claude
    ```

    Sign in if the browser opens, then accept the workspace trust dialog. Ask:

    > What does this project do?

    Then end the session. Don't change any files yet; Lab 1 starts from this clean state.

## Done when

- `claude` answers in the starter repo.
- Both test suites run.
- `git remote -v` shows `origin` pointing to your GitHub repo.

## Stretch goals

- Install the Claude Code extension for VS Code (it uses the CLI you just installed) and open the panel in the repo.
- Run the app. Use two terminals, both from the repo root:

  ```bash
  uv run --directory api fastapi dev app/main.py   # API on http://127.0.0.1:8000 (docs at /docs)
  npm --prefix web run dev                         # page on http://localhost:5173
  ```

  Tick a task as done, then reload the page. Remember what you see for Lab 2.
- Run `/status` and `/doctor` inside a `claude` session and see what they report.

## Troubleshooting

- **`claude`, `uv` or `node` not found right after installing:** open a new terminal so it picks up the new PATH.
- **`claude --version` is older than v2.1.287:** run `claude update`. For Homebrew or WinGet installs use `brew upgrade claude-code` or `winget upgrade Anthropic.ClaudeCode`. ⚠
- **Windows: Claude Code can't find Git Bash:** add this to the `env` block of `~/.claude/settings.json`, with the path to your `bash.exe`:

  ```json
  { "env": { "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe" } }
  ```

- **Corporate proxy: `npm --prefix web install` hangs or fails with certificate errors:** npm needs your company's proxy settings. Ask IT for the proxy and certificate details for npm, or do the setup on a network without the proxy.
- **`claude auth status --text` shows the wrong account:** see step 5 (`ANTHROPIC_API_KEY`), or run `/logout` then `/login` inside `claude`.
- **`gh repo create` fails:** run `gh auth login` first, or create an empty private repo on github.com and use `git remote add origin <url>`.
- **Port 8000 already in use (Docker Desktop and WSL often hold it):** run the API on another port and tell the page about it: `uv run --directory api fastapi dev app/main.py --port 8001`, then `API_PORT=8001 npm --prefix web run dev` (PowerShell: `$env:API_PORT=8001; npm --prefix web run dev`).
- **Odd API test failures, or `no such column` errors:** an old `api/taskboard.db` from an earlier run is in the way. Delete it and run the tests again.
