# Foundations (OSS Python LangGraph)

## Scope

Use these pages for setup, API selection, and baseline architecture:

- Overview: https://docs.langchain.com/oss/python/langgraph/overview
- Install: https://docs.langchain.com/oss/python/langgraph/install
- Quickstart: https://docs.langchain.com/oss/python/langgraph/quickstart
- Choosing APIs: https://docs.langchain.com/oss/python/langgraph/choosing-apis
- Graph API concept: https://docs.langchain.com/oss/python/langgraph/graph-api
- Functional API concept: https://docs.langchain.com/oss/python/langgraph/functional-api
- Use Graph API: https://docs.langchain.com/oss/python/langgraph/use-graph-api
- Use Functional API: https://docs.langchain.com/oss/python/langgraph/use-functional-api

## API Selection Rules

- Choose Graph API when explicit nodes/edges and deterministic control flow are required.
- Choose Functional API when task-style composition is clearer than explicit graph wiring.
- Prefer consistency within one feature area; avoid mixing APIs in the same module unless necessary.

## Minimal Acceptance Checklist

- Graph compiles and executes on a deterministic input.
- State schema is explicit and stable.
- Routing conditions are testable.
- A migration path exists to persistence and interrupts.
