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

> Targets skeino **1.0.0+**. 1.0.0 made persistence *scheme-authoritative* and
> moved database drivers behind extras (see Persistence); on 0.x the selector was
> `postgres_uri`.

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
| `agent_nodes` / `status_field` | Enable token-level message streaming (see below). |
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

## Token-level streaming

To stream assistant tokens incrementally (not just whole-state snapshots), tell
skeino which graph nodes emit assistant messages, and optionally a status field:

```python
SkeinoSettings(agent_nodes=frozenset({"chatbot"}), status_field="pipeline_status")
```

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

Or raw HTTP. Key endpoints:

- **Threads:** `POST /threads`, `POST /threads/search`, `GET /threads/{id}`,
  `PATCH /threads/{id}` (update metadata), `DELETE /threads/{id}`,
  `POST /threads/{id}/copy` (fork into an independent thread).
- **State & time travel:** `GET /threads/{id}/state`, `POST /threads/{id}/state`
  (human-in-the-loop edit → new checkpoint), `GET /threads/{id}/state/{checkpoint_id}`
  and `POST /threads/{id}/state/checkpoint` (read at a checkpoint),
  `GET|POST /threads/{id}/history`.
- **Runs:** `POST /threads/{id}/runs` (run to completion),
  `POST /threads/{id}/runs/stream` (SSE: `event:`/`data:` frames),
  `GET /threads/{id}/runs`.
- **Assistants / meta:** `POST /assistants/search`,
  `GET /assistants/{id}/schemas`, `GET /api/health`, `GET /info`.

Run options on `POST /runs[/stream]`: `input` **or** `command` (resume),
`stream_mode` (`values`/`updates`/`messages`/`events`/…), `multitask_strategy`
(`enqueue` default, or `reject`/`rollback`/`interrupt` → 409 when busy),
`if_not_exists: "create"` to auto-create the thread.

## v1 scope & gotchas

- **Single graph** per app; assistant CRUD/versioning is not exposed.
- `after_seconds` (scheduled runs) and `webhook` are **rejected** (400).
- A `langgraph.json` `store.uri` maps to `checkpointer_uri`; `auth`/`ui`/`http.app`
  are ignored (warned). There is no Store API, auth, or cron support yet.
- Concurrency is **one run per thread**, enforced with in-process locks — correct
  for single-process deployments only.
- In-memory persistence (`checkpointer_scheme="memory"`, the default) is **not
  durable** — pick a durable scheme (`postgres`/`sqlite`/`mongodb`) for anything
  real, and install its extra.

For full request/response shapes see the skeino docs and the OpenAPI schema the
server serves at `/openapi.json` (and Swagger UI at `/docs`).
