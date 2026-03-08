---
name: langgraph-python-troubleshooting
description: Systematic troubleshooting and remediation for LangGraph Python runtime errors, graph recursion problems, checkpointing failures, invalid state updates, and multi-subgraph edge cases. Use when a LangGraph run fails, produces unstable behavior, or returns framework-level error messages.
---

# LangGraph Python Troubleshooting

## Use This Workflow

1. Capture full error text, stack trace, and triggering input.
2. Read `references/errors.md` and locate exact matching error class/page.
3. Validate state shape, checkpoint configuration, and recursion controls.
4. Reproduce with minimal graph fixture.
5. Apply fix, then run regression test for the failing branch.

## Diagnosis Priorities

1. Structural mismatch in node return/state updates.
2. Missing or misconfigured checkpointer.
3. Invalid chat history mutation.
4. Concurrent graph update conflicts.
5. Excessive recursion or subgraph composition issues.

## Boundaries

- Focus on root-cause isolation and narrow fixes.
- Defer broader redesign to `langgraph-python-patterns` once stable.
