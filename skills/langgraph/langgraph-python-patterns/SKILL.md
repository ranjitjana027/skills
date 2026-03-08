---
name: langgraph-python-patterns
description: Reusable LangGraph Python design patterns for multi-agent workflows, memory, retrieval, subgraphs, and general agent architectures. Use when implementing non-trivial graph behavior, selecting architecture patterns, or mapping functional requirements to official LangGraph examples.
---

# LangGraph Python Patterns

## Use This Workflow

1. Read `references/patterns.md` to select a matching architecture pattern.
2. Start from the simplest matching pattern and preserve state schema clarity.
3. Add one pattern at a time: subgraphs, memory, retrieval, or specialization.
4. Verify behavior with deterministic test prompts before composing patterns.

## Pattern Selection Heuristics

- Use `application-structure` for package/module layout decisions.
- Use `workflows-agents` and `thinking-in-langgraph` to model orchestration.
- Use `use-subgraphs` for composability and modular graph sections.
- Use `add-memory` when long-lived conversational or task state is required.
- Use `agentic-rag` and `sql-agent` for retrieval/data-backed agent flows.

## Boundaries

- Keep focus on architecture patterns.
- Delegate deployment/runtime concerns to `langgraph-python-production`.
- Delegate runtime failures to `langgraph-python-troubleshooting`.
