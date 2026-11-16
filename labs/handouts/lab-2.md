# Lab 2 · Plan it, test it first, then let Claude commit

**Time:** 45 minutes · **Start tag:** `lab-2-start` · **Module:** M4 Workflows & Best Practices

## Goal

Use plan mode to add **due dates** and an **overdue filter** to Taskboard, test-first, on both stacks: pytest for `api/` and Vitest for `web/`. Then find and fix the **seeded bug** and review the diff before Claude commits.

The overdue rules, from the starter's `README.md`:

> **Overdue:** a task is overdue only if it has a due date, that date is before today, and it is not done. A task due today is not overdue, and a task with no due date is never overdue. Compare dates in UTC: `datetime.now(UTC).date()` on the API, `new Date().toISOString().slice(0, 10)` on the web.

## Prerequisites

- Lab 0 and Lab 1 Part A are done: Claude Code at least v2.1.287 ⚠, and both test suites run in your Taskboard clone.
- You don't need your own Lab 1 result. `lab-2-start` already has the reference `CLAUDE.md` and `.claude/settings.json`, which allow both test commands.
- Two spare terminals to run the API and the page (needed for the seeded bug):

```bash
uv run --directory api fastapi dev app/main.py   # API on http://127.0.0.1:8000
npm --prefix web run dev                         # page on http://localhost:5173
```

## Steps

1. **Start a branch from the lab tag** (from the repo root):

   ```bash
   git switch -c lab-2 lab-2-start
   ```

2. **Start Claude in plan mode.** Either run `claude --permission-mode plan`, or press Shift+Tab in a running session until you see `⏸ plan mode on`. Then ask for a plan:

   > Plan adding an optional due_date to tasks and an "overdue" filter (GET /tasks?overdue=true returns tasks with due_date before today that are not done). Cover: the model/schema change, the API, the web UI filter toggle, and the tests on both sides (pytest for api/, Vitest for web/). List the edge cases (no due date, due today, done tasks, timezones). Follow the overdue rules in README.md. Don't implement.

3. **Review the plan, then approve it.**
   - Check it against the overdue rules above. Are all four edge cases covered? Does it compare dates in UTC? Does an existing database get the new column?
   - Press Ctrl+G to edit the plan in your editor, or choose "No, keep planning" and say what to change.
   - When it's right, approve it. The choices are ⚠: approve and switch to auto mode, approve and review edits manually, or keep planning. Approving with manual edits lets you watch each change.

4. **Red, then green.** Ask for the failing tests first:

   > Write the failing tests first, and don't implement yet: pytest cases in api/tests for due_date validation and the overdue filter, plus a Vitest test for the overdue logic and the filter toggle. Run both suites and show me the failing output. Don't touch application code yet.

   Check that the tests fail for the right reason (missing code, not a typo). If you like, commit the red tests now so you can see later if they changed. Then:

   > Now implement until all tests pass. Don't modify the tests. Show me the test output.

   Look for the evidence: the commands Claude ran and their output, not only "all tests pass". The commands are `uv run --directory api pytest -q` and `npm --prefix web test`.

5. **Seeded bug.** Start the API and the page (see Prerequisites), open http://localhost:5173, add a task and tick it. It shows as done. **Reload the page: the tick is gone.** Give Claude the symptom, not a guess:

   > Mark a task done, then reload the page: the tick is gone. Find the cause in both stacks and fix it with a failing test first.

   Afterwards, tick a task again and reload: it should stay ticked. Stop both servers.

   <details>
   <summary>Hint</summary>

   No test in the starter catches this, and the API never returns an error. Open the browser's DevTools → Network tab, tick a task, and look at the body of the `PATCH /api/tasks/{id}` request and its status. Then compare it with what the API's update model in `api/app/schemas.py` accepts.

   <details>
   <summary>Answer</summary>

   `setDone` in `web/src/api.ts` sends `{"completed": true}`, but the API field is `done`. The `TaskUpdate` Pydantic model ignores unknown fields by default, so the request is valid, nothing changes, and the API returns **200**. The page updates its own state before the request returns, so the tick looks saved until you reload.

   - **Fix (one line, web):** in `setDone`, change `JSON.stringify({ completed: done })` to `JSON.stringify({ done })`. The API needs no change.
   - **Tests, one on each side** (as in `git diff lab-2-start lab-3-start`):
     - Vitest, `"sends the done field when toggling"` in `web/tests/api.test.ts`: call `setDone(1, true)` with a mocked `fetch` and assert the URL is `/api/tasks/1`, the method is `PATCH` and the body equals `{ done: true }`. It fails before the fix.
     - pytest, `test_patch_done_persists` in `api/tests/test_due_dates.py`: `PATCH /tasks/{id}` with `{"done": true}`, then `GET` the task and assert `done` is `True`. It passes before the fix too: it locks in the API contract that the web side must match.

   The lesson: a bug can sit between two stacks that each pass their own tests. Check what crosses the boundary.

   </details>
   </details>

6. **Review, then commit.** Read the full change yourself:

   ```bash
   git diff
   ```

   When you're happy with it:

   > Commit this with a message that explains why.

   Claude asks for permission to run `git commit`, because the settings only allow the test commands. Read the message before you approve.

## Done when

- The new tests pass in both suites, and Claude shows the output.
- Claude committed the change after you reviewed the diff.
- A ticked task stays ticked after a reload.

<details>
<summary>Model answer, in short (the <code>lab-3-start</code> tag)</summary>

Compare yours with `git diff lab-2-start lab-3-start`. In the reference:

- **API:** `due_date: date | None = None` on `TaskCreate`, `TaskUpdate` and `Task`; a `due_date TEXT` column plus a migration that adds it to old databases; `GET /tasks?overdue=true` filters with `due_date IS NOT NULL AND due_date < today AND done = 0`, where today comes from `app/clock.py` (`today_utc()`, patched in tests). `PATCH` with `"due_date": null` clears the date.
- **Web:** `web/src/overdue.ts` with `todayUtc()` and `isOverdue(task, today)`; a date input in the form; an "Overdue only" checkbox; overdue dates show in red.
- **Bug fix:** `setDone` sends `{ done }`, with a regression test on the PATCH body.
- **Tests:** 10 pass in `api/`, 12 in `web/`.

</details>

## Stretch goals

Homework, if you run out of time.

- **Set a goal instead of a prompt** ⚠: `/goal all tests in api/ and web/ pass and no test file is modified, or stop after 15 turns`. A model checks the condition after every turn by reading the transcript; it doesn't run the tests itself.
- **Fresh-eyes review:** run `/code-review` on your branch and decide which findings are worth fixing.
- **Clear a due date:** make `PATCH /tasks/{id}` with `{"due_date": null}` remove the date, test-first, and add a way to clear it in the page. (The reference API at `lab-3-start` already clears it, but has no test for it and no control in the page.)

## Troubleshooting

- **`no such column: due_date`:** your `api/taskboard.db` was created before the change. Delete it and restart the API, or ask Claude to add a migration for existing databases.
- **Overdue is off by one day:** something compares in local time. Use the UTC dates from the overdue rules on both sides, and patch "today" in tests instead of using the real clock.
- **Claude edits the tests to make them pass:** press Esc and say "Stop. You're changing the tests to make them pass, which isn't what I want. Restore the tests to what we agreed (use /rewind if needed). Keep the tests fixed and change only api/app/ and web/src/ until they pass." After two failed corrections, run `/clear` and write one better prompt.
- **A permission prompt for the tests:** the allow rules match only the root-relative commands. If Claude runs `cd web && npm test`, ask it to use `npm --prefix web test` and `uv run --directory api pytest -q`, as `CLAUDE.md` says.
- **The tick still disappears after your fix:** reload with the dev servers running your branch's code, and check the PATCH body in DevTools again.
