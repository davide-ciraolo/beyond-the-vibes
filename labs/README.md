# Beyond the Vibes · Labs

Hands-on labs for the *Beyond the Vibes* agentic-coding course. Labs 1–5 work on **Taskboard**, a small task app: a FastAPI + SQLite API in `api/` and a Vite + TypeScript page (no framework) in `web/`.

## What's here

- `taskboard.bundle`: the starter app as a single git file. It holds `main` (the clean starting point, the same as `lab-1-start`), a tag for the start of each lab (`lab-1-start` … `lab-5-start`) and `lab-5-done`, the finished reference. Download it from the `labs/` folder of [github.com/davide-ciraolo/beyond-the-vibes](https://github.com/davide-ciraolo/beyond-the-vibes).
- `docsearch.bundle`: the DocSearch app for the Day 2 RAG labs (8.1–8.3), see [DocSearch](#docsearch-day-2-rag-edition).
- `handouts/`: one handout per lab. Each has the same sections: Goal · Prerequisites · Steps · Done when · Stretch goals · Troubleshooting.

## Get the starter

Install the tools first with [Lab 0](handouts/lab-0-setup.md) (Node 24, Claude Code, `uv`, git, `jq`). Getting the starter is Part A of [Lab 1](handouts/lab-1.md). In short, from the folder where you saved `taskboard.bundle`:

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
cd web
npm install    # inside web/: older npm ignores --prefix for install
cd ..
uv run --directory api pytest -q
npm --prefix web test
```

## Labs

| Lab | Day | Time | Start tag | Handout |
|---|---|---|---|---|
| 0 · Tools and account | Before Day 1 (checked at the welcome) | ~15 min | none | [lab-0-setup.md](handouts/lab-0-setup.md) |
| 1 · Get the starter, CLAUDE.md and permissions | Day 1 | 30 min | `lab-1-start` | [lab-1.md](handouts/lab-1.md) |
| 2 · Plan mode, test-first feature, seeded bug | Day 1 | 45 min | `lab-2-start` | [lab-2.md](handouts/lab-2.md) |
| 3 · Toolkit, Playwright MCP, a project Skill | Day 2 | 40 min | `lab-3-start` | [lab-3.md](handouts/lab-3.md) |
| 4 · Security-reviewer subagent, brainstorm → plan → build | Day 2 | 40 min | `lab-4-start` | [lab-4.md](handouts/lab-4.md) |
| 5 · Claude PR review in GitHub Actions | Day 2 | 30 min | `lab-5-start` | [lab-5.md](handouts/lab-5.md) |
| 8.1 · Stream OCR from your own Claude (DocSearch) | Day 2 RAG | 30 min | `lab-8.1-start` | [lab-8-1.md](handouts/lab-8-1.md) |
| 8.2 · Queue, chunk, embed, store (DocSearch) | Day 2 RAG | 35 min | `lab-8.2-start` | [lab-8-2.md](handouts/lab-8-2.md) |
| 8.3 · Give Claude Code your knowledge base (DocSearch) | Day 2 RAG | 25 min | `lab-8.3-start` | [lab-8-3.md](handouts/lab-8-3.md) |

Each lab's finished state is the next lab's start tag (Lab 5's is `lab-5-done`, Lab 8.3's is `lab-8-done`), so you can always pick up where the class is.

## DocSearch (Day 2, RAG edition)

The RAG edition of Day 2 replaces Labs 3–5 with three labs on **DocSearch**, a small document-search app: drop scanned PDFs or images, OCR them with Claude, chunk and embed the text, store it in Qdrant, and let Claude Code search it over MCP. It runs in Docker Compose: `meridian` (Claude through your subscription), `api` (FastAPI), `web` (Vite + TypeScript), and from Lab 8.2 `redis`, `worker` and `qdrant`, from Lab 8.3 `mcp`.

> ⚠ Meridian runs Claude through your personal Claude subscription. Anthropic's terms say Agent SDK use should go through API keys, and Anthropic may enforce this without notice. Use it at your own risk, and use an API key for anything beyond this exercise.

You need Docker Desktop, Node 24, `uv` and Claude Code. Download `docsearch.bundle` and `example.pdf` from the `labs/` folder of [github.com/davide-ciraolo/beyond-the-vibes](https://github.com/davide-ciraolo/beyond-the-vibes), then, from the folder where you saved them:

```bash
git clone docsearch.bundle docsearch
cd docsearch
git switch -c lab-8.1 lab-8.1-start
docker compose build                              # a few minutes: start it early
docker compose run --rm meridian claude login     # once
docker compose up -d meridian
curl http://localhost:3456/health                 # must show "loggedIn": true
```

| Tag | Contents |
|---|---|
| `lab-8.1-start` | `meridian`, `api` and `web`; OCR and stream endpoint stubbed, failing tests for Lab 8.1 |
| `lab-8.2-start` | Lab 8.1 solution, plus `redis`, `worker` and `qdrant`; chunker, store, queue, worker, new API routes and web pages stubbed, failing tests for Lab 8.2 |
| `lab-8.3-start` | Lab 8.2 solution, plus the `mcp` service; MCP tools stubbed, failing tests for Lab 8.3 |
| `lab-8-done` | The full reference solution; `main` points here |

Missed a DocSearch lab? Start the next one from its tag, for example `git switch -c lab-8.2 lab-8.2-start`. The Docker images and the Meridian login carry over.

## Missed a lab?

Start the next one from its tag. From the repo root:

```bash
git switch -c lab-N lab-N-start
```

Replace `N` with the lab number, for example `git switch -c lab-3 lab-3-start`.
