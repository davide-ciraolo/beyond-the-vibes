# Lab 8.3 · Give Claude Code your knowledge base

**Time:** 25 minutes · **Start tag:** `lab-8.3-start` · **Module:** M8 RAG with Claude (Day 2, RAG edition)

## Goal

Close the loop: expose DocSearch to Claude Code as an MCP server, so Claude can ingest files you give it, search your documents and answer with links that open the right page in the viewer. Claude Code becomes the "generation" half of your RAG system, and DocSearch the retrieval half.

## Prerequisites

- Lab 8.2 is done, or the stack runs from `lab-8.2-start`'s solution: `docker compose ps` shows `meridian`, `api`, `web`, `redis`, `qdrant` and `worker` up, and at least one document is `done` in the task list.
- A file to ingest through Claude: `example.pdf` or any PDF, PNG or JPEG.

## Steps

1. **Start a branch at the tag.** From the repo root:

   ```bash
   git switch -c lab-8.3 lab-8.3-start
   ```

   The tag contains a working Lab 8.2 solution and the `mcp` service, whose tools aren't written yet.

2. **Have Claude make the MCP tests pass.** Prompt:

   > Read CLAUDE.md and `tests/test_mcp.py`. Implement the `Tools` methods in `docsearch/mcp_server.py` (`ingest_file`, `search`, `get_document`, `list_documents`) and register them on the MCP server in `build_server`. `ingest_file` takes a file name in `data/inbox/` only: no paths outside it, no URLs. Run the backend tests.

   Check the tool descriptions before you accept: Claude reads them to decide when to call each tool, so `search` should say that each result has a link to cite.

3. **Start the MCP service.**

   ```bash
   docker compose up -d --build mcp
   ```

4. **Add it to Claude Code.**

   ```bash
   claude mcp add --transport http docsearch http://localhost:8765/mcp
   ```

5. **Check it's connected.** Start `claude` in the repo root and run:

   ```text
   /mcp
   ```

   `docsearch` should be listed as connected, with four tools.

6. **Let Claude ingest a file.** Copy a file into the inbox, then ask Claude to ingest it:

   ```bash
   mkdir -p data/inbox
   cp ../example.pdf data/inbox/    # use the path where you saved it
   ```

   > Use the docsearch tools to ingest example.pdf and tell me when list_documents shows it done.

   Approve the tool calls. The new document appears in the task list at <http://localhost:5173>.

7. **Ask a question and click a link.** For example:

   > Using docsearch, what loss does DPO optimise? Answer with links to the pages you used.

   Click a link in the answer: it should open the viewer on that document and page.

## Done when

- Claude answers from your documents, with links that open the right page in the viewer.
- `/mcp` shows `docsearch` connected, and the backend tests pass.

## Stretch goals

- **Rerank:** retrieve ~50 chunks, rerank them with a cross-encoder (bge-reranker through fastembed's `TextCrossEncoder`) and return the best 5.
- **Query rewrite:** add a tool that turns a question into two or three search queries, and merge the results.
- **Read-only mode:** an environment variable that leaves `ingest_file` out, for a server that only searches.

## Troubleshooting

- **The tools aren't listed:** run `claude mcp list` to see whether `docsearch` is registered and connects. `claude mcp add` registers it for this project only (local scope), so start `claude` from the repo root.
- **`docsearch` shows as failed:** check `docker compose ps mcp` and `docker compose logs mcp`, then `/mcp` to reconnect. The URL ends in `/mcp`.
- **`ingest_file` returns an error:** the file must sit directly in `data/inbox/` under the repo root, and you pass only its name (`example.pdf`), not a path.
- **Links open the document list instead of the page:** the links come from `WEB_URL` (default `http://localhost:5173`). Check it in `compose.yaml` under `mcp:`, and that the link looks like `http://localhost:5173/doc/<id>#page=N`.
- **`search` returns nothing:** no document is `done` yet, or Qdrant is empty (`curl http://localhost:6333/collections/chunks`). Wait for the worker, see [Lab 8.2](lab-8-2.md) Troubleshooting.
- **You want to remove the server afterwards:** `claude mcp remove docsearch`.
