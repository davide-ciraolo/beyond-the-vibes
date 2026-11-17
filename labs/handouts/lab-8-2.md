# Lab 8.2 · Queue, chunk, embed, store

**Time:** 35 minutes · **Start tag:** `lab-8.2-start` · **Module:** M8 RAG with Claude (Day 2, RAG edition)

> ⚠ This lab still uses Meridian from Lab 8.1, which runs Claude through your personal subscription. The warning at the top of [Lab 8.1](lab-8-1.md) applies.

## Goal

Turn the one-off OCR stream into an ingestion pipeline. An upload now becomes a task on a Redis queue; a `worker` service OCRs the pages, splits the Markdown into section-aware chunks, embeds them on your CPU (`BAAI/bge-small-en-v1.5`, 384 dimensions) and stores them in Qdrant. The web page shows a task list with live statuses, and clicking a document opens the original on the left and the extracted Markdown on the right.

## Prerequisites

- Lab 8.1 is done, or at least steps 1–4 (bundle cloned, images built, Meridian logged in, `curl http://localhost:3456/health` shows `"loggedIn": true`).
- Two files to ingest: `example.pdf` and one more PDF, PNG or JPEG of your own.

## Steps

1. **Start a branch at the tag.** From the repo root:

   ```bash
   git switch -c lab-8.2 lab-8.2-start
   ```

   The tag contains a working Lab 8.1 solution, so you can start here even if 8.1 isn't finished. Your 8.1 work stays on the `lab-8.1` branch. (You can also stay on your branch and run `git merge lab-8.2-start`; where you and the tag changed the same file, keep the tag's version if in doubt.)

2. **Read the chunking tests first.** Open `backend/tests/test_chunking.py`. The tests are the spec: chunks follow Markdown headings and carry the heading path as `section` (`Guide > Install`); `<!-- page N -->` markers set the page; long sections are split under ~450 tokens with ~15% overlap; a table or a `$$` maths block is never split; the text to embed starts with the section path.

3. **Have Claude make the chunker pass.** Prompt:

   > Read CLAUDE.md. Make `tests/test_chunking.py` pass by implementing `chunk_markdown` in `docsearch/chunking.py`. Don't change the tests. Run only that test file.

   Check the result against the rules in step 2 before you accept: the tests are small, so read how overlap and the table/maths rule are done.

4. **Then the rest of the pipeline, one test file at a time.** Prompt:

   > Now make the remaining backend tests pass, in this order: `test_store.py` (Qdrant collection `chunks`), `test_tasks.py` (the Redis queue), `test_worker.py` (OCR → chunk → embed → store, updating the status at each step), then the new routes in `test_api.py`. Run the full suite at the end.

   The tests use fakes and an in-memory Qdrant; nothing calls Meridian or downloads a model.

5. **Have Claude finish the web pages.** Prompt:

   > Finish `refresh()` in `web/src/tasks.ts` (the task list, polled every 2 seconds) and `renderViewer` in `web/src/viewer.ts` (PDF in an iframe or image on the left, extracted Markdown on the right, `/doc/<id>#page=N`). Show the Markdown rendered by default with `renderMarkdown` from `web/src/render.ts`, which is already written and sanitises its output, and add a button that switches to the raw text. Use `textContent` for anything else that comes from a file. Run the web tests.

6. **Run the stack.**

   ```bash
   docker compose up -d --build
   ```

   This adds `redis`, `qdrant` and `worker` to the Lab 8.1 services.

7. **Upload two files and open one.** Open <http://localhost:5173> and drop both files. Watch the status go `queued` → `ocr` → `chunking` → `embedding` → `done` (in another terminal: `docker compose logs -f worker`). Click a `done` row. Then check that Qdrant has the chunks:

   ```bash
   curl http://localhost:6333/collections/chunks
   ```

## Done when

- The task list shows `done` for both files, and clicking one opens the original on the left and the Markdown on the right.
- `curl http://localhost:6333/collections/chunks` shows `points_count` greater than 0.
- The backend and web tests pass.

## Stretch goals

- **Live text in the viewer:** have the worker publish OCR tokens on a Redis pub/sub channel and stream them to the viewer, as in Lab 8.1.
- **Hybrid search:** add a BM25 (sparse) vector next to the dense one in Qdrant and fuse the two rankings.
- **Multilingual:** switch to `intfloat/multilingual-e5-small` and re-ingest a document in another language. Mind the query/passage prefixes that model expects, and recreate the collection if the dimension changes.

## Troubleshooting

- **The first document sits in `embedding` for a while:** the first run downloads the embedding model (~130 MB) into the `models` volume. Later runs reuse it.
- **A status stays `queued`:** the worker isn't running or can't reach Redis. Run `docker compose ps` and `docker compose logs worker`.
- **A row shows `failed`:** the status shows the error message. The usual causes are a Meridian problem (see [Lab 8.1](lab-8-1.md) Troubleshooting) or a file the OCR couldn't read.
- **`points_count` is 0 but the status is `done`:** the document produced no chunks (empty transcription). Open `data/extracted/<id>.md` to check.
- **The viewer says "Not extracted yet.":** the document isn't `done` yet. Go back to the list and wait.
- **The PDF doesn't show on the left:** some browsers block PDFs in an iframe. Open `http://localhost:8000/documents/<id>/file` directly.
