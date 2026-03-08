---
name: langgraph-python-core
description: Core LangGraph Python architecture, setup, and API selection guidance for building stateful, tool-using AI agents. Use when creating or refactoring LangGraph agents, choosing between Graph API and Functional API, implementing foundational graph structure, or grounding implementation details in official OSS Python docs.
---

# LangGraph Python Core

## Use This Workflow

1. Confirm the target stack is OSS Python LangGraph.
2. Read `references/foundations.md` for setup and API-selection guardrails.
3. Read `references/docs-index.md` and load only the section needed for the user request.
4. Implement minimal working graph first, then add persistence, interrupts, and deployment concerns via companion skills.

## Core Build Sequence

1. Install and verify environment from the install guide.
2. Choose Graph API or Functional API before writing runtime logic.
3. Build a small end-to-end quickstart graph.
4. Expand node/state design incrementally.
5. Hand off to `langgraph-python-patterns`, `langgraph-python-production`, or `langgraph-python-troubleshooting` as needed.

## Boundaries

- Focus on foundations and API selection.
- Do not overload context with production or error-detail docs unless required.
- Use the official docs URLs in references as source of truth.
