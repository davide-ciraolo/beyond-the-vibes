# Beyond the Vibes · Labs

Hands-on labs for the *Beyond the Vibes* agentic-coding course. Every lab works on **Taskboard**, a small task app: a FastAPI + SQLite API in `api/` and a Vite + TypeScript page (no framework) in `web/`.

## What's here

- `taskboard.bundle`: the starter app as a single git file. It holds `main` (the clean starting point, the same as `lab-1-start`), a tag for the start of each lab (`lab-1-start` … `lab-5-start`) and `lab-5-done`, the finished reference. Download it from the `labs/` folder of [github.com/davide-ciraolo/beyond-the-vibes](https://github.com/davide-ciraolo/beyond-the-vibes).
- `handouts/`: one handout per lab. Each has the same sections: Goal · Prerequisites · Steps · Done when · Stretch goals · Troubleshooting.

## Get the starter

Full instructions, including what to install first, are in [Lab 0](handouts/lab-0-setup.md). In short, from the folder where you saved `taskboard.bundle`:

```bash
git clone taskboard.bundle taskboard
cd taskboard
git remote rename origin course
gh repo create taskboard --private --source . --remote origin   # or create it on github.com and: git remote add origin <url>
git push -u origin main --tags
```

Then, from the repo root:

```bash
uv sync --directory api
npm --prefix web install
uv run --directory api pytest -q
npm --prefix web test
```

## Labs

| Lab | Day | Time | Start tag | Handout |
|---|---|---|---|---|
| 0 · Setup | Before the course | ~20 min | none (`main`) | [lab-0-setup.md](handouts/lab-0-setup.md) |
| 1 · CLAUDE.md and permissions | Day 1 | 30 min | `lab-1-start` | [lab-1.md](handouts/lab-1.md) |
| 2 · Plan mode, test-first feature, seeded bug | Day 1 | 45 min | `lab-2-start` | [lab-2.md](handouts/lab-2.md) |
| 3 · Toolkit, Playwright MCP, a project Skill | Day 2 | 40 min | `lab-3-start` | [lab-3.md](handouts/lab-3.md) |
| 4 · Security-reviewer subagent, brainstorm → plan → build | Day 2 | 40 min | `lab-4-start` | [lab-4.md](handouts/lab-4.md) |
| 5 · Claude PR review in GitHub Actions | Day 2 | 30 min | `lab-5-start` | [lab-5.md](handouts/lab-5.md) |

Each lab's finished state is the next lab's start tag (Lab 5's is `lab-5-done`), so you can always pick up where the class is.

## Missed a lab?

Start the next one from its tag. From the repo root:

```bash
git switch -c lab-N lab-N-start
```

Replace `N` with the lab number, for example `git switch -c lab-3 lab-3-start`.
