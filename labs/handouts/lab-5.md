# Lab 5 · Put a Claude reviewer on your own pull requests

**Time:** 30 minutes · **Start tag:** `lab-5-start`

## Goal

Add a GitHub Actions workflow, `.github/workflows/claude-review.yml`, that runs Claude Code on every pull request. The workflow uses `--permission-mode default` and a strict `--allowedTools` list. Then open a PR with a planted bug and read Claude's review. Claude comments, and **you** decide whether to merge.

## Prerequisites

- Lab 0 done: your own GitHub repo is `origin`, and `main` plus the tags are pushed (`git remote -v` shows it).
- Admin rights on that repo, so you can install the Claude GitHub App and add secrets.
- One of these:
  - an Anthropic **Console API key** (platform.claude.com), or
  - a **Pro/Max** subscription, to mint a token with `claude setup-token`.
- Optional: the `gh` CLI, logged in (`gh auth login`). `/install-github-app` needs it, and it is handy for `gh secret set`.

## Steps

1. **Put `main` at the lab start.** claude-code-action checks that the workflow file in a PR matches the one on the default branch ⚠ verify before delivery. So the workflow goes on `main` first, and the bug goes on a separate branch later.

   ```bash
   git switch main && git merge --ff-only lab-5-start
   ```

2. **Add the workflow and the review rules on `main`.** Start `claude` and ask:

   > Add `.github/workflows/claude-review.yml`: a PR review with `anthropics/claude-code-action@v1` on pull_request (opened, synchronize, ready_for_review, reopened). Use permissions `contents: read`, `pull-requests: write`, `id-token: write`, `timeout-minutes: 15`, and a `concurrency` group that cancels stale runs. In `claude_args`, use `--permission-mode default`, `--max-turns 20`, and allow only the inline-comment tool and `gh pr comment`, `gh pr diff` and `gh pr view`. The prompt must say "Follow the review rules in CLAUDE.md" and "Do not approve the PR; a human decides whether to merge."

   > Append a "Review rules" section to CLAUDE.md: API/web contract drift, a test for every behaviour change, SQL built from strings, `innerHTML` and secrets. Report findings and never approve a PR.

   Compare the result with the model answers below. The action restores `CLAUDE.md` from the base branch, so the review rules must be on `main` too.

   <details>
   <summary>Model answer: <code>.github/workflows/claude-review.yml</code></summary>

   ```yaml
   # Based on docs/solutions.md "Automatic PR Code Review" (anthropics/claude-code-action@v1).
   # Note: /install-github-app names its generated review file claude-code-review.yml.
   name: Claude PR Review

   on:
     pull_request:
       types: [opened, synchronize, ready_for_review, reopened]

   # A new push cancels the previous, now-stale review
   concurrency:
     group: claude-review-${{ github.event.pull_request.number }}
     cancel-in-progress: true

   jobs:
     review:
       if: github.event.pull_request.draft == false
       runs-on: ubuntu-latest
       timeout-minutes: 15
       permissions:
         contents: read
         pull-requests: write
         id-token: write
       steps:
         - name: Checkout repository
           uses: actions/checkout@v6
           with:
             fetch-depth: 1

         - name: Claude review
           uses: anthropics/claude-code-action@v1
           with:
             anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
             # Pro/Max subscription instead of an API key: run `claude setup-token`, store it as
             # CLAUDE_CODE_OAUTH_TOKEN, and replace the line above with:
             # claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
             prompt: |
               REPO: ${{ github.repository }}
               PR NUMBER: ${{ github.event.pull_request.number }}

               Review this pull request. The repo is a FastAPI (Python) backend and a TypeScript frontend.
               Follow the review rules in CLAUDE.md. Focus on:
               - Correctness bugs and unhandled edge cases
               - Security: input validation, authN/authZ, secrets, injection
               - API contract drift between FastAPI models and the TypeScript client types
               - Missing or weak tests for the changed behaviour

               Note: The PR branch is already checked out in the current working directory.

               Use `gh pr comment` for one top-level summary.
               Use `mcp__github_inline_comment__create_inline_comment` (with `confirmed: true`) for specific code issues.
               Only post GitHub comments - don't submit review text as messages.
               Do not approve the PR; a human decides whether to merge.
             # --permission-mode default: strict allowlist (anything else is denied; nobody can approve in CI)
             claude_args: |
               --permission-mode default
               --max-turns 20
               --allowedTools "mcp__github_inline_comment__create_inline_comment,Bash(gh pr comment:*),Bash(gh pr diff:*),Bash(gh pr view:*)"
   ```

   Versions to check: `@v1`, `actions/checkout@v6` ⚠ verify before delivery.

   </details>

   <details>
   <summary>Model answer: <code>CLAUDE.md</code> review rules</summary>

   ```markdown
   ## Review rules
   - Check that `web/src/api.ts` still matches `api/app/schemas.py` (field names and types).
   - Every behaviour change needs a test on the side that changed.
   - Flag SQL built from strings, `innerHTML`, and any secret in code or logs.
   - Report findings; never approve a PR.
   ```

   </details>

   Optional YAML check:

   ```bash
   uv run --with pyyaml python -c "import yaml,sys; yaml.safe_load(open('.github/workflows/claude-review.yml')); print('ok')"
   ```

   Review `git diff`, then commit and push `main`:

   > Commit the workflow and the review rules with a message that explains why.

   ```bash
   git push origin main
   ```

3. **Install the Claude GitHub App.** Pick one route:
   - **Quick:** in `claude`, run `/install-github-app`. It needs repo admin rights, a logged-in `gh`, and a github.com remote ⚠ verify before delivery. It installs the App and saves the secret (so you can skip step 4). It also offers to push its own `claude.yml` and `claude-code-review.yml` in a PR. You already have a review workflow, so leave that PR unmerged.
   - **Manual:** install https://github.com/apps/claude on your Taskboard repo.

4. **Add the secret** (repo Settings → Secrets and variables → Actions, or `gh`). Use one of these:
   - **Console API key:** name it `ANTHROPIC_API_KEY`.

     ```bash
     gh secret set ANTHROPIC_API_KEY
     ```

   - **Pro/Max:** mint a token, store it as `CLAUDE_CODE_OAUTH_TOKEN`, then switch the workflow to the commented `claude_code_oauth_token` line (commit and push `main` again).

     ```bash
     claude setup-token
     gh secret set CLAUDE_CODE_OAUTH_TOKEN
     ```

   The key never goes in the YAML.

5. **Plant a bug on a PR branch.**

   ```bash
   git switch -c lab-5-pr
   ```

   In `web/src/api.ts`, rename `due_date` to `dueDate` in the request body of `createTask`:

   ```ts
   // before
   body: JSON.stringify({ title, due_date: dueDate, tags })
   // after (the bug)
   body: JSON.stringify({ title, dueDate, tags })
   ```

   What happens: the API ignores the unknown `dueDate` field, so new tasks silently lose their due date. One web test catches it: `npm --prefix web test` reports `1 failed | 15 passed`, because `creates a task with POST and a JSON title` in `web/tests/api.test.ts` expects the body `{ title: "Write tests", due_date: null, tags: [] }`. The API suite still shows `16 passed`, and the review workflow doesn't run the tests.

   Don't fix it, and don't touch the test: the point is to see what the PR review says. Commit it:

   > Commit this change to web/src/api.ts as "Use camelCase for the due date". Don't fix anything else.

   Push the branch, then open a PR against `main` on github.com:

   ```bash
   git push -u origin lab-5-pr
   ```

6. **Read the review.** On the PR, watch the Actions tab: the run takes a few minutes. Claude posts one summary comment and inline comments. Did it catch the field-name drift between `web/src/api.ts` and `api/app/schemas.py`? **You** decide whether to merge. Claude never approves.

## Done when

Opening a PR produces a Claude review comment.

## Stretch goals

- Add `--max-budget-usd` to `claude_args`, leaving headroom because it is a client-side estimate. Note 08 lists it as a raw CLI flag, so check that the action passes it through ⚠ verify before delivery.
- Add an `@claude` workflow (`.github/workflows/claude.yml`) with `--permission-mode acceptEdits` and a tight `--allowedTools` list. Then mention `@claude` in a PR comment and ask it to fix the bug.
- Run `/code-review` locally on `lab-5-pr` before pushing. It is the local twin of the CI review.

## Troubleshooting

- **No comment on the PR.**
  - Open the run in the Actions tab and read the log.
  - Check the secret name: exactly `ANTHROPIC_API_KEY`, or `CLAUDE_CODE_OAUTH_TOKEN` with the matching `claude_code_oauth_token` input in the YAML.
  - Check that the Claude GitHub App is installed on this repo.
  - Draft PRs are skipped (`if: github.event.pull_request.draft == false`). Mark the PR ready for review.
- **"Workflow validation failed".** The workflow file must be on the default branch, identical to the PR's copy ⚠ verify before delivery. Push it to `main` first (step 2), then push a new commit to the PR branch.
- **`git merge --ff-only` refuses.** Your local `main` has commits that aren't in `lab-5-start`. Move that work to its own branch first.
- **`/install-github-app` exits.** It needs a logged-in `gh` (`gh auth login`), admin rights, and a github.com remote. Use the manual route instead.
- **The review ran but posted nothing.** Anything outside `--allowedTools` is denied in `--permission-mode default`. Check the log for denied tools, and that the allowlist still contains `gh pr comment` and the inline-comment tool.
