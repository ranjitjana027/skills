---
name: skeino
description: >-
  Serve, configure, embed, and call a skeino server — the embeddable,
  open-source replacement for the `langgraph dev` HTTP server that exposes a
  LangGraph-Platform-compatible REST API (threads, runs, streaming/SSE,
  assistants) over any compiled LangGraph graph. TRIGGER when: code imports
  `skeino` or uses `create_app` / `from_langgraph_json` / `SkeinoSettings`; the
  user wants to expose, serve, run, or deploy a LangGraph graph over HTTP, stand
  up a langgraph-dev-compatible API, configure its persistence (Postgres or
  in-memory checkpointer), embed it in an existing FastAPI app, or call/stream
  its threads & runs endpoints. SKIP when authoring the LangGraph graph itself
  (use the langgraph skills) or working on LangGraph Cloud/Platform managed
  infrastructure.
---

# Serving a LangGraph graph with skeino

`skeino` turns any compiled LangGraph graph into a running HTTP server that
speaks the LangGraph Platform REST dialect — so LangGraph Studio and the
`langgraph_sdk` client work against it. It is the embeddable counterpart to
`langgraph dev`: a library you call, not a managed service.

The public surface is small: **`create_app`**, **`SkeinoSettings`**,
**`from_langgraph_json`**, **`GraphRegistry`** (all importable from `skeino`).

## Install

```bash
pip install skeino   # brings in fastapi, langgraph, uvicorn, etc.
```

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
def build(checkpointer):
    return builder.compile(checkpointer=checkpointer)

app = create_app(graphs={"my_agent": build}, settings=SkeinoSettings(postgres_uri=...))
```

v1 routes a **single** assistant: the default is `default_assistant_id` (must be
a key in `graphs`) or the first key.

## From a `langgraph.json` manifest

Mirrors how `langgraph dev` boots — loads `env`, resolves `graphs[name]` as
`path:attribute`, applies `http.cors` and `store.uri`:

```python
from skeino import from_langgraph_json
app = from_langgraph_json("langgraph.json")  # optional settings= overrides
```

## Configuration — `SkeinoSettings`

| Field | Purpose |
| --- | --- |
| `postgres_uri` | When set, uses the Postgres checkpointer **and** metadata store; absent → both in-memory (ephemeral). |
| `checkpointer_scheme` / `checkpointer_options` | Pick/parametrise the checkpointer backend explicitly. |
| `default_assistant_id` | The served assistant (key in `graphs`). |
| `assistant_name` / `assistant_description` / `assistant_namespace` | Assistant identity (the namespace derives the assistant's deterministic UUID). |
| `agent_nodes` / `status_field` | Enable token-level message streaming (see below). |
| `server_title` / `server_description` / `server_version` / `welcome_message` | Presentation. |
| `cors_origins` / `cors_methods` / `cors_headers` | CORS. |

`SkeinoSettings` is a `pydantic-settings` model, so every field is also readable
from the environment.

## Persistence

- **No `postgres_uri`** → in-memory checkpointer + metadata store. Great for dev;
  nothing persists across restarts.
- **`postgres_uri` set** → Postgres-backed checkpointer and metadata store.
- **Custom backend** → register one and select it via `checkpointer_scheme`:

```python
from skeino.persistence import register_checkpointer

@register_checkpointer("redis")
def _build_redis(spec):  # returns an async context manager yielding a BaseCheckpointSaver
    ...
```

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
from langgraph_sdk import get_client
client = get_client(url="http://localhost:2024")
thread = await client.threads.create()
async for chunk in client.runs.stream(thread["thread_id"], "my_agent",
                                       input={"messages": [{"role": "user", "content": "hi"}]}):
    print(chunk.event, chunk.data)
```

Or raw HTTP. Key endpoints: `POST /threads`, `GET /threads/{id}`,
`GET /threads/{id}/state`, `GET|POST /threads/{id}/history`,
`POST /threads/{id}/runs` (run to completion), `POST /threads/{id}/runs/stream`
(SSE: `event:`/`data:` frames), `GET /threads/{id}/runs`,
`POST /assistants/search`, `GET /assistants/{id}/schemas`,
`GET /api/health`, `GET /info`.

Run options on `POST /runs[/stream]`: `input` **or** `command` (resume),
`stream_mode` (`values`/`updates`/`messages`/`events`/…), `multitask_strategy`
(`enqueue` default, or `reject`/`rollback`/`interrupt` → 409 when busy),
`if_not_exists: "create"` to auto-create the thread.

## v1 scope & gotchas

- **Single graph** per app; assistant CRUD/versioning is not exposed.
- `after_seconds` (scheduled runs) and `webhook` are **rejected** (400).
- A `langgraph.json` `store` provides only the Postgres URI; `auth`/`ui`/`http.app`
  are ignored (warned). There is no Store API, auth, or cron support yet.
- Concurrency is **one run per thread**, enforced with in-process locks — correct
  for single-process deployments only.
- In-memory persistence is **not durable** — set `postgres_uri` for anything real.

For full request/response shapes see the skeino docs and the OpenAPI schema the
server serves at `/openapi.json` (and Swagger UI at `/docs`).
