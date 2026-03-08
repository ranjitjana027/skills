---
name: python-type-annotations
description: Enforce Python type annotation best practices when writing or reviewing Python type hints. Use when adding type hints to Python code, reviewing mypy or pytype errors, annotating function signatures, using Optional/Union/Any, defining TypeVar or TypeAlias, annotating generics, handling NoneType, or following PEP 484/526 conventions. Trigger when the user asks about Python type hints, type annotations, static type checking, mypy, pytype, or typing module usage in Python.
---

# Python Style Guide: Type Annotations

## Workflow

1. Read `references/type-annotations.md` for the full rule set.
2. When writing or reviewing annotated code, verify each applicable sub-section.
3. Explain which rule applies and why when flagging issues.
4. Pair with `python-language-rules` and `python-style-rules` for complete coverage.

## Quick Checklist

- [ ] Public APIs are annotated; private/complex code also prioritized
- [ ] `self` and `cls` not annotated (unless using `Self`)
- [ ] `__init__` return type not annotated
- [ ] `Optional[X]` or `X | None` for nullable types — never implicit `None`
- [ ] `Any` used only when type cannot be expressed
- [ ] Generics have explicit type params: `list[int]`, not bare `list`
- [ ] Type aliases use `TypeAlias` annotation and `CapWords` naming
- [ ] TypeVars are private (`_T`, `_K`, `_V`) unless exported
- [ ] Imports from `typing` are direct: `from typing import Any, Optional`
- [ ] Built-in parametric types preferred: `tuple[int, str]` not `typing.Tuple[int, str]`
- [ ] Lists annotated as `list[T]`, tuples as `tuple[T1, T2]` or `tuple[T, ...]`
- [ ] `str` for text, `bytes` for binary — never `typing.Text`
- [ ] Circular imports resolved via `if TYPE_CHECKING:` block with quoted type strings

## Boundaries

- Focus on type annotation correctness and style.
- Source of truth: https://google.github.io/styleguide/pyguide.html section 3.19
