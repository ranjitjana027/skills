---
name: python-language-rules
description: Enforce Python language best practices when writing, reviewing, or refactoring Python code. Use when checking Python code for correctness, reviewing imports structure, handling exceptions, using lambdas, generators, comprehensions, properties, decorators, default arguments, global state, threading, or any Python language feature. Trigger when the user asks to write Pythonic code, review Python for best practices, fix bad Python patterns, or apply Python style conventions.
license: Complete terms in LICENSE.txt
---

# Python Style Guide: Language Rules

## Workflow

1. Read `references/language-rules.md` for the full rule set.
2. Apply rules while writing or reviewing code — check each applicable section.
3. When suggesting fixes, always explain *which rule* is violated and *why* the rule exists.
4. Pair with `python-style-rules` for formatting/naming and `python-type-annotations` for type hints.

## Quick Checklist

When reviewing Python code, verify:

- [ ] Imports are of modules, not individual symbols (with noted exceptions)
- [ ] No relative imports — full package paths only
- [ ] Exceptions inherit from existing classes, end in `"Error"`, never use bare `except:`
- [ ] No `assert` for runtime validation — only for tests
- [ ] No mutable default arguments (`[]`, `{}`, `set()`)
- [ ] No mutable module-level state (constants `CAPS_WITH_UNDER` are fine)
- [ ] Comprehensions have at most one `for` clause + one optional filter
- [ ] Lambdas are single-line only; use named functions otherwise
- [ ] Properties don't hide side effects or require heavy computation
- [ ] No `@staticmethod` — use module-level functions instead
- [ ] Threading uses `queue.Queue`, not raw atomicity assumptions
- [ ] No power features: metaclasses, `__del__`, dynamic inheritance, bytecode hacks
- [ ] `if __name__ == '__main__':` guard with a `main()` function

## Boundaries

- Focus on language-level correctness and idiom — not formatting (see `python-style-rules`).
- Do not enforce type annotation rules here (see `python-type-annotations`).
- Source of truth: https://google.github.io/styleguide/pyguide.html sections 2.1–2.21
