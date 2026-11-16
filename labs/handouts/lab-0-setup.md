# Lab 0 · Setup (tools and account)

About 15 minutes. Install the tools the labs use and check that Claude Code signs in with the right account. We check this together at the start of Day 1. Getting the Taskboard starter is the first part of Lab 1.

## Goal

Install Claude Code and the tools the labs use, switch to Node 24, and confirm that `claude` uses the account you expect.

## Prerequisites

- A Claude account you can sign in with (the one you will use in the course).
- A GitHub account.
- macOS 13+, Windows 10 1809+, or Linux (for example Ubuntu 20.04+). 4 GB+ RAM. ⚠

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
   - Node 24 LTS ([nodejs.org](https://nodejs.org/), or with nvm: see step 4)
   - [GitHub CLI](https://cli.github.com/) `gh`, signed in with `gh auth login` (optional: you can create the repo on github.com instead)
   - `jq` (used by hook examples on Day 2):

   ```bash
   brew install jq            # macOS
   sudo apt install jq        # Debian / Ubuntu / WSL
   winget install jqlang.jq   # Windows
   ```

### Check

4. **Switch to Node 24.** The web tests need Node 22.13 or later; the course uses Node 24.

   ```bash
   node -v
   ```

   It must print `v24.…`. If it doesn't and you use nvm (or nvm-windows), switch:

   ```bash
   nvm install 24
   nvm use 24
   node -v
   ```

   Without nvm, install Node 24 LTS from [nodejs.org](https://nodejs.org/) and open a new terminal.

5. Check the Claude Code version, update, and run the diagnostics. You need **v2.1.287 or later**. ⚠

   ```bash
   claude --version
   claude update
   claude doctor
   ```

6. Check which account Claude Code will use:

   ```bash
   claude auth status --text
   ```

   It must show the account you expect. If you haven't signed in yet, run `claude auth login`.

   If it shows an **API key** instead of your account, a stray `ANTHROPIC_API_KEY` in your shell is winning over your login. Remove it for this shell, then check again:

   ```bash
   unset ANTHROPIC_API_KEY
   ```

   ```powershell
   Remove-Item Env:ANTHROPIC_API_KEY
   ```

   If it comes back in new terminals, remove it from your shell profile or system environment variables too.

7. Check the rest of the tools:

   ```bash
   git --version
   uv --version
   jq --version
   gh auth status     # only if you use gh
   ```

## Done when

- `node -v` prints `v24.…`.
- `claude --version` is v2.1.287 or later ⚠, and `claude auth status --text` shows your account.
- `git`, `uv` and `jq` answer with a version.

## Stretch goals

- Install the Claude Code extension for VS Code (it uses the CLI you just installed).
- Start `claude` in any folder and run `/status` and `/doctor` to see what they report.

## Troubleshooting

- **`claude`, `uv` or `node` not found right after installing:** open a new terminal so it picks up the new PATH.
- **`node -v` still shows the old version after `nvm use 24`:** nvm only switches the current terminal. Run `nvm alias default 24` (nvm) so new terminals use 24 too; on nvm-windows, `nvm use 24` is global but needs a new terminal.
- **`claude --version` is older than v2.1.287:** run `claude update`. For Homebrew or WinGet installs use `brew upgrade claude-code` or `winget upgrade Anthropic.ClaudeCode`. ⚠
- **Windows: Claude Code can't find Git Bash:** add this to the `env` block of `~/.claude/settings.json`, with the path to your `bash.exe`:

  ```json
  { "env": { "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe" } }
  ```

- **`claude auth status --text` shows the wrong account:** see step 6 (`ANTHROPIC_API_KEY`), or run `/logout` then `/login` inside `claude`.
