# Lab 8.1 · Stream OCR from your own Claude

**Time:** 30 minutes · **Start tag:** `lab-8.1-start` · **Module:** M8 RAG with Claude (Day 2, RAG edition)

> ⚠ Meridian runs Claude through your personal Claude subscription. Anthropic's terms say Agent SDK use should go through API keys, and Anthropic may enforce this without notice. Use it at your own risk, and use an API key for anything beyond this exercise.

## Goal

Get DocSearch running in Docker, with Meridian as a local Claude endpoint, then have Claude Code finish the first step of a RAG pipeline: drop a scanned PDF or an image on a web page, send each page to Claude as an image, and watch the Markdown transcription stream in, token by token. The result is saved to `data/extracted/<id>.md`, ready to be chunked in Lab 8.2.

## Prerequisites

- Docker Desktop is installed and running (`docker compose version` prints a version).
- Node 24 (`node -v` prints `v24.…`) and `uv` (`uv --version`), so Claude can run the tests on your machine.
- Claude Code, logged in with your own account (Lab 0).
- `docsearch.bundle` and `example.pdf`: download both from the `labs/` folder of [github.com/davide-ciraolo/beyond-the-vibes](https://github.com/davide-ciraolo/beyond-the-vibes). Any scanned PDF, PNG or JPEG of your own works too.

## Steps

1. **Clone the bundle and start a branch at the tag.** In the folder where you saved `docsearch.bundle`:

   ```bash
   git clone docsearch.bundle docsearch
   cd docsearch
   git switch -c lab-8.1 lab-8.1-start
   ```

   From here on, every command runs from the repo root. The stack is `meridian` (Claude through your subscription, port 3456), `api` (FastAPI, port 8000) and `web` (Vite, port 5173). All three ports listen on `127.0.0.1` only.

2. **Build the images early.** Start this during the break: the Meridian image installs Claude Code and takes a few minutes.

   ```bash
   docker compose build
   ```

3. **Log Meridian in, once.** This runs `claude login` inside the Meridian container:

   ```bash
   docker compose run --rm meridian claude login
   ```

   Open the URL it prints, approve, and paste the code back if asked. The credentials stay in the `claude-auth` Docker volume, so you only do this once.

4. **Start Meridian and check it.**

   ```bash
   docker compose up -d meridian
   curl http://localhost:3456/health
   ```

   The JSON must show `"loggedIn": true`. A 503 with `"loggedIn": false` means step 3 didn't finish: run it again.

5. **Have Claude make the backend tests pass.** Open Claude Code in the repo root (`claude`) and prompt:

   > Read CLAUDE.md. Then run the backend tests (`uv run pytest -q` in `backend/`) and make them pass: first `docsearch/ocr.py` (render PDF pages to PNG, one streaming Meridian call per page), then the OCR stream endpoint in `docsearch/api.py`. Don't change the tests.

   Check before you accept: the tests use fakes, never the real Meridian, and `api.py` sends server-sent events (`page`, `text`, `done`, `error`). The storage tests pass from the start; the OCR and stream tests should go from failing to passing.

6. **Have Claude finish the drop zone.** Prompt:

   > Finish `handle()` in `web/src/upload.ts`: upload the dropped file, then append the streamed OCR text to the right pane as it arrives, with a page divider for each `page` event and the message for an `error` event. Use `textContent`, not `innerHTML`. Run the web tests.

   The web tests run with `npm test` in `web/` (Claude may need `npm install` there first).

7. **Run the whole stack and drop a file.**

   ```bash
   docker compose up -d --build
   ```

   Open <http://localhost:5173>, drop `example.pdf` on the left, and watch the text appear on the right. Then look in `data/extracted/`: there is one `<id>.md` per document, with `<!-- page N -->` markers.

## Done when

- The text streams into the right pane token by token (or page by page, see Troubleshooting), and `data/extracted/<id>.md` exists.
- The backend tests pass (`uv run pytest -q` in `backend/`) and the web tests pass (`npm test` in `web/`).

## Stretch goals

- **Render the Markdown:** ask Claude to render the streamed text as HTML with a Markdown library (and to sanitise it, since it comes from a model).
- **Cancel button:** stop a long transcription halfway and mark the document as cancelled.
- **Tokens per second:** show a live rate next to the page counter.

## Troubleshooting

- **The login URL doesn't open:** copy it from the terminal into your browser by hand. If the container exits before you paste the code, run step 3 again.
- **Port 3456 (or 8000, 5173) is already in use:** another Meridian or dev server is running. Stop it, or change the left side of the `ports:` entry in `compose.yaml` (for example `127.0.0.1:3457:3456`) and use that port in the `curl` check.
- **`/health` keeps saying `"loggedIn": false`:** run step 3 again from the repo root (the `claude-auth` volume belongs to this Compose project), then `docker compose restart meridian`.
- **"model not found" in the stream:** the model name isn't one Meridian accepts. `docker compose logs meridian` lists the model names it maps; set one as `OCR_MODEL` under `api: environment:` in `compose.yaml` (default `claude-sonnet-5-5` ⚠) and run `docker compose up -d api`.
- **CORS error in the browser console:** the page must call the API at `http://localhost:8000`. Check `VITE_API_URL` in `compose.yaml`, and open the page as `http://localhost:5173`, not `127.0.0.1:5173` (the API allows the `localhost` origin only).
- **Text arrives a page at a time, not token by token:** your Meridian version doesn't stream ⚠. The code still works: each page appears when it's done. Check `docker compose logs meridian` for the version and see the "Notes" in the repo's `README.md`.
- **Upload rejected with 415:** only PDF, PNG and JPEG are accepted, checked by content, not by file name, up to 25 MB.
- **`data/extracted/` is empty:** the stream ended with an error. The last lines in the right pane, and `docker compose logs api`, show why.
