# Production References

## Runtime Reliability

- Durable execution: https://docs.langchain.com/oss/python/langgraph/durable-execution
- Persistence: https://docs.langchain.com/oss/python/langgraph/persistence
- Interrupts: https://docs.langchain.com/oss/python/langgraph/interrupts

## Runtime Visibility

- Streaming: https://docs.langchain.com/oss/python/langgraph/streaming
- Observability: https://docs.langchain.com/oss/python/langgraph/observability

## Validation and Tooling

- Testing: https://docs.langchain.com/oss/python/langgraph/test
- Local server: https://docs.langchain.com/oss/python/langgraph/local-server
- Studio: https://docs.langchain.com/oss/python/langgraph/studio
- UI: https://docs.langchain.com/oss/python/langgraph/ui

## Deployment and Internals

- Deploy: https://docs.langchain.com/oss/python/langgraph/deploy
- Pregel runtime internals: https://docs.langchain.com/oss/python/langgraph/pregel
- Python changelog: https://docs.langchain.com/oss/python/langgraph/changelog-py

## Production Readiness Checklist

- Checkpointing strategy defined and tested.
- Resume behavior validated after forced interruption.
- Traces/metrics/logs connected to observability stack.
- Test matrix includes happy path, branch path, and failure path.
- Deployment runbook documented with rollback steps.
