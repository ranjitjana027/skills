---
name: skeino
description: >-
  Serve, configure, embed, and call a skeino server — the embeddable,
  open-source replacement for the `langgraph dev` HTTP server that exposes a
  LangGraph-Platform-compatible REST API (threads, runs, streaming/SSE,
  assistants) over any compiled LangGraph graph. TRIGGER when: code imports
  `skeino` or uses `create_app` / `from_langgraph_json` / `SkeinoSettings`; the
  user wants to expose, serve, run, or deploy a LangGraph graph over HTTP, stand
  up a langgraph-dev-compatible API, configure its persistence (in-memory,
  SQLite, Postgres, MongoDB, or Redis checkpointer backends), embed it in an
  existing FastAPI app, or call/stream its threads & runs endpoints. SKIP when
  authoring the LangGraph graph itself (use the langgraph skills) or working on
  LangGraph Cloud/Platform managed infrastructure.
---

# Serving a LangGraph graph with skeino

`skeino` turns any compiled LangGraph graph into a running HTTP server that
speaks the LangGraph Platform REST dialect — so LangGraph Studio and the
`langgraph_sdk` client work against it. It is the embeddable counterpart to
`langgraph dev`: a library you call, not a managed service.

The public surface is small: **`create_app`**, **`SkeinoSettings`**,
**`from_langgraph_json`**, **`GraphRegistry`** (all importable from `skeino`).

> Targets skeino **3.0.0+** — **3.0.1+** for graphs that pause on `interrupt()`
> (see Human-in-the-loop, below). **3.0.0 is breaking:** `POST /threads/{id}/runs` no
> longer runs to completion — it now starts the graph in a background task and
> returns immediately with a `pending`/`running` run. Get the old blocking
> behavior from the new `POST /threads/{id}/runs/wait` (see Runs, below).
> 2.0.0 made streaming standards-faithful and **removed the `agent_nodes` /
> `status_field` settings** (see Live progress streaming). 1.0.0 made
> persistence *scheme-authoritative* and moved database drivers behind extras
> (see Persistence); on 0.x the selector was `postgres_uri`.

## Install

```bash
pip install skeino                 # core only — ships the in-memory backend
pip install 'skeino[postgres]'     # + PostgreSQL backend
pip install 'skeino[sqlite]'       # + SQLite backend
pip install 'skeino[mongodb]'      # + MongoDB backend
```

The default install pulls fastapi / langgraph / uvicorn but **no** database
drivers — each durable backend is an optional extra, imported lazily.

## Quickstart — serve a graph

```python
import uvicorn
from skeino import create_app, SkeinoSettings
from my_app.graph import graph  # a compiled LangGraph graph (graph = builder.compile())

app = create_app(
    graphs={"my_agent": graph},          # assistant_id -> graph
    settings=SkeinoSettings(
        default_assistant_id="my_agent",
        assistant_name="My Agent",
    ),
)

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=2024)
```

`graphs` values may be either a precompiled `CompiledStateGraph` **or** a builder
`(checkpointer) -> CompiledStateGraph` (sync or async). Use a builder when the
graph must be compiled with skeino's resolved checkpointer:

```python
from my_app.graph import builder  # the StateGraph, *before* .compile()

def build(checkpointer):
    return builder.compile(checkpointer=checkpointer)

app = create_app(
    graphs={"my_agent": build},
    settings=SkeinoSettings(
        checkpointer_scheme="postgres",
        checkpointer_uri="postgresql://user:pass@localhost/db",
    ),
)
```

v1 routes a **single** assistant: the default is `default_assistant_id` (must be
a key in `graphs`) or the first key.

## From a `langgraph.json` manifest

Mirrors how `langgraph dev` boots — loads `env`, resolves `graphs[name]` as
`path:attribute`, applies `http.cors`, and maps `store.uri` to `checkpointer_uri`
(the scheme is derived from the URI prefix):

```python
from skeino import from_langgraph_json
app = from_langgraph_json("langgraph.json")  # optional settings= overrides
```

## Configuration — `SkeinoSettings`

| Field | Purpose |
| --- | --- |
| `checkpointer_scheme` | **Authoritative** backend selector (default `"memory"`): `memory`, `postgres`/`postgresql`, `sqlite`/`sqlite3`, `mongodb`/`mongo`, `redis`. Drives *both* the checkpointer and the metadata store. |
| `checkpointer_uri` | Connection string for that scheme (e.g. `postgresql://…`, a SQLite file path, `mongodb://…`). Ignored if it doesn't match the scheme. |
| `checkpointer_options` | Extra params passed to the checkpointer builder. |
| `allow_ephemeral_metadata` | Opt out of the startup guard that rejects a durable checkpointer paired with the in-memory metadata store (see Persistence). |
| `default_assistant_id` | The served assistant (key in `graphs`). |
| `assistant_name` / `assistant_description` / `assistant_namespace` | Assistant identity (the namespace derives the assistant's deterministic UUID). |
| `server_title` / `server_description` / `server_version` / `welcome_message` | Presentation. |
| `cors_origins` / `cors_methods` / `cors_headers` | CORS. |

`SkeinoSettings` is a plain (frozen) pydantic `BaseModel` — settings live in your
code. To read from the environment, use `pydantic-settings` in *your* project and
pass the resulting values in.

## Persistence

skeino keeps two stores, **both** selected by `checkpointer_scheme`: LangGraph's
*checkpointer* (graph state/history) and skeino's *metadata store* (thread/run
rows). The scheme is authoritative — `checkpointer_uri` is only the connection
string for it, and a URI that doesn't match the scheme is ignored (e.g.
`checkpointer_scheme="memory"` with a Postgres URI still uses in-memory).

| `checkpointer_scheme` | Backend | Durable | Install |
| --- | --- | --- | --- |
| `memory` (default) | in-process | No (ephemeral) | bundled |
| `postgres` / `postgresql` | `AsyncPostgresSaver` + native metadata store | Yes | `skeino[postgres]` |
| `sqlite` / `sqlite3` | `AsyncSqliteSaver` + native metadata store | Yes (file) | `skeino[sqlite]` |
| `mongodb` / `mongo` | `MongoDBSaver` + native metadata store | Yes | `skeino[mongodb]` |
| `redis` | lazy `redis` checkpointer builder | checkpointer only | `pip install langgraph-checkpoint-redis` |

**Fail-loud guard.** A durable checkpointer scheme that has **no native metadata
store** (e.g. `redis`, or a custom scheme) is rejected at startup — otherwise
graph state would persist while the thread/run list silently evaporated. Opt out
with `allow_ephemeral_metadata=True`.

**Custom backend** — register a builder and select it via `checkpointer_scheme`.
The builder is an **async context manager** taking a `CheckpointerSpec`
(`.uri`, `.options`) and yielding a `BaseCheckpointSaver`, so resources are
released on shutdown:

```python
from contextlib import asynccontextmanager
from skeino.persistence import register_checkpointer, CheckpointerSpec

@register_checkpointer("redis")
@asynccontextmanager
async def _build_redis(spec: CheckpointerSpec):
    saver = make_redis_saver(spec.uri, **spec.options)  # a BaseCheckpointSaver
    try:
        yield saver
    finally:
        await saver.aclose()
```

A custom durable scheme has no native metadata store, so pair it with a
supported metadata scheme or set `allow_ephemeral_metadata=True` (see below).

## Live progress streaming

skeino forwards each requested `stream_mode` faithfully, like a real LangGraph
server: `values` (full state per super-step), `updates` (per-node deltas),
`custom` (graph-emitted events). `values` and `updates` are passed through a
fail-closed output-key filter so internal state never leaks.

For low-bandwidth live UIs, have clients request `["updates", "custom"]` instead
of `["values"]` (each node's new message only, no full-history re-send), and emit
progress from your graph nodes via LangGraph's `get_stream_writer()`:

```python
from langgraph.config import get_stream_writer

def my_node(state):
    get_stream_writer()({"type": "status", "message": "Working…"})
    ...
```

> Before 2.0.0 this was configured with the removed `agent_nodes` / `status_field`
> settings; skeino now emits standard LangGraph modes instead.

## Embed in an existing FastAPI app

`create_app` returns a normal `FastAPI` instance — mount or include it:

```python
parent.mount("/agent", create_app(graphs={...}, settings=...))
```

## Calling the server

It speaks the LangGraph Platform dialect, so the official client works:

```python
import asyncio
from langgraph_sdk import get_client

async def main():
    client = get_client(url="http://localhost:2024")
    thread = await client.threads.create()
    async for chunk in client.runs.stream(
        thread["thread_id"], "my_agent",
        input={"messages": [{"role": "user", "content": "hi"}]},
    ):
        print(chunk.event, chunk.data)

asyncio.run(main())  # in a notebook, just `await main()`
```

**Stateless runs** (no thread lifecycle to manage): `POST /runs[/wait|/stream]`
and `POST /runs/batch` run against a thread created and deleted inside the
request — no thread id in, none out. `/runs` and `/runs/batch` block until
done (skeino has no background executor for these); `checkpoint` is rejected
(400) since there's no history to resume. Use the thread-scoped routes for
anything you need to inspect, resume, or continue.

Or raw HTTP. Key endpoints:

- **Threads:** `POST /threads`, `POST /threads/search`, `GET /threads/{id}`,
  `PATCH /threads/{id}` (update metadata), `DELETE /threads/{id}`,
  `POST /threads/{id}/copy` (fork into an independent thread).
- **State & time travel:** `GET /threads/{id}/state`, `POST /threads/{id}/state`
  (human-in-the-loop edit → new checkpoint), `GET /threads/{id}/state/{checkpoint_id}`
  and `POST /threads/{id}/state/checkpoint` (read at a checkpoint),
  `GET|POST /threads/{id}/history`.
- **Runs (background):** `POST /threads/{id}/runs` starts the graph in a
  background task and returns immediately with a `pending`/`running` run.
  `POST /threads/{id}/runs/wait` runs to completion and returns the final
  graph state values (the old synchronous behavior). `GET
  /threads/{id}/runs/{run_id}/join` waits for an in-flight run to finish and
  returns its output. `POST /threads/{id}/runs/{run_id}/cancel?action=interrupt|rollback`
  cancels it (`rollback` also deletes the run row); `DELETE
  /threads/{id}/runs/{run_id}` removes a terminal run (409 if still active).
  `POST /threads/{id}/runs/stream` streams SSE (`event:`/`data:` frames).
  `GET /threads/{id}/runs` lists them.
- **Assistants / meta:** `POST /assistants/search`,
  `GET /assistants/{id}/schemas`, `GET /api/health`, `GET /info`.

### Human-in-the-loop (`interrupt()`)

A graph node that calls LangGraph's `interrupt(value)` ends the run cleanly and
parks the thread waiting for a decision (3.0.1+; earlier versions dropped the
pause on the way to the client):

- The pending request streams on the reserved `__interrupt__` channel of
  `values` and `updates` events as `[{"value": ..., "id": ...}]` — the shape
  `useStream().interrupt` reads.
- The thread's `status` becomes `interrupted`, and the request stays readable in
  the thread's `interrupts` and in its state's `tasks[].interrupts`.
- Resume with another run on the same thread carrying
  `{"command": {"resume": <decision>}}`; `interrupt()` returns `<decision>` when
  the node replays.

Output-schema filtering never strips `__interrupt__` — reserved dunder channels
are protocol, not graph state.

Run options on `POST /runs[/wait|/stream]`: `input` **or** `command` (resume),
`stream_mode` (`values`/`updates`/`messages`/`events`/…), `multitask_strategy`
(`enqueue` default; `reject` → 409 when busy; `interrupt` cancels the active
run; `rollback` cancels **and deletes** it), `if_not_exists: "create"` to
auto-create the thread.

## v1 scope & gotchas

- **Single graph** per app; assistant CRUD/versioning is not exposed.
- `after_seconds` (scheduled runs) and `webhook` are **rejected** (400).
- A `langgraph.json` `store.uri` maps to `checkpointer_uri`; `auth`/`ui`/`http.app`
  are ignored (warned). There is no Store API, auth, or cron support yet.
- Concurrency is **one run per thread**, enforced with in-process locks — correct
  for single-process deployments only.
- Background runs (3.0.0+) live in the server process's own `asyncio` event
  loop, not a durable job queue — a graceful shutdown marks in-flight runs
  `interrupted`, but a crash loses them. Not a fit for a multi-process/worker
  deployment expecting cross-process run recovery.
- In-memory persistence (`checkpointer_scheme="memory"`, the default) is **not
  durable** — pick a durable scheme (`postgres`/`sqlite`/`mongodb`) for anything
  real, and install its extra.

For full request/response shapes see the skeino docs and the OpenAPI schema the
server serves at `/openapi.json` (and Swagger UI at `/docs`).
