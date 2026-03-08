# Error References

## Error Index

- Common errors landing page: https://docs.langchain.com/oss/python/langgraph/common-errors

## Canonical Error Pages

- GRAPH_RECURSION_LIMIT: https://docs.langchain.com/oss/python/langgraph/errors/GRAPH_RECURSION_LIMIT
- INVALID_CHAT_HISTORY: https://docs.langchain.com/oss/python/langgraph/errors/INVALID_CHAT_HISTORY
- INVALID_CONCURRENT_GRAPH_UPDATE: https://docs.langchain.com/oss/python/langgraph/errors/INVALID_CONCURRENT_GRAPH_UPDATE
- INVALID_GRAPH_NODE_RETURN_VALUE: https://docs.langchain.com/oss/python/langgraph/errors/INVALID_GRAPH_NODE_RETURN_VALUE
- MISSING_CHECKPOINTER: https://docs.langchain.com/oss/python/langgraph/errors/MISSING_CHECKPOINTER
- MULTIPLE_SUBGRAPHS: https://docs.langchain.com/oss/python/langgraph/errors/MULTIPLE_SUBGRAPHS

## Legacy Aliases (Also Present In Docs)

- https://docs.langchain.com/oss/python/langgraph/GRAPH_RECURSION_LIMIT
- https://docs.langchain.com/oss/python/langgraph/INVALID_CHAT_HISTORY
- https://docs.langchain.com/oss/python/langgraph/INVALID_CONCURRENT_GRAPH_UPDATE
- https://docs.langchain.com/oss/python/langgraph/INVALID_GRAPH_NODE_RETURN_VALUE
- https://docs.langchain.com/oss/python/langgraph/MISSING_CHECKPOINTER
- https://docs.langchain.com/oss/python/langgraph/MULTIPLE_SUBGRAPHS

## Fast Triage Checklist

- Confirm error class exactly matches one documented page.
- Verify node return payload matches expected state schema.
- Verify checkpointer is configured for flows requiring persistence.
- Verify concurrent writes are serialized or conflict-aware.
- Verify recursion limits and termination conditions are explicit.
- Reduce to minimal reproducible graph before broad refactors.
