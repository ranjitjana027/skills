---
name: langgraph-python-production
description: LangGraph Python production operations guidance for durability, persistence, observability, deployment, local runtime tooling, and release hygiene. Use when preparing LangGraph agents for real workloads, testing runtime behavior, or operating deployed graphs with monitoring and debugging workflows.
---

# LangGraph Python Production

## Use This Workflow

1. Read `references/operations.md` to identify the runtime capability needed.
2. Add durability, checkpoints, and interrupts before scaling complexity.
3. Add observability and streaming instrumentation early.
4. Validate with local server and test strategy before deployment.
5. Track changes against changelog entries during upgrades.

## Production Order Of Operations

1. Persistence and durable execution.
2. Interrupt and resume semantics.
3. Streaming and observability hooks.
4. Test strategy and local runtime.
5. Deployment and studio/UI workflows.

## Boundaries

- Avoid architecture-level redesign here unless production constraints force it.
- Escalate structural changes back to `langgraph-python-patterns`.
