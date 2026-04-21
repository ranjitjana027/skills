---
name: python-style-rules
description: Enforce Python formatting, naming, and documentation best practices when writing or reviewing Python code. Use when checking line length, indentation, whitespace, blank lines, naming conventions, docstrings, comments, string formatting, import ordering, function length, or the main guard pattern. Trigger when the user asks to format Python code, write docstrings, name variables/functions/classes, review Python for style issues, or apply Python style conventions for any formatting concern.
license: Complete terms in LICENSE.txt
---

# Python Style Guide: Style Rules

## Workflow

1. Read `references/formatting.md` for line length, indentation, whitespace, and blank-line rules.
2. Read `references/naming.md` for naming conventions and the full naming table.
3. Read `references/docstrings.md` for docstring format, comment style, and TODO format.
4. Apply rules while writing or reviewing. When flagging issues, cite the rule number and explain the rationale.
5. Pair with `python-language-rules` for language-level correctness and `python-type-annotations` for type hint style.

## Quick Checklist

When reviewing Python code style:

**Formatting**
- [ ] Lines ≤ 80 characters (no backslash continuations — use implicit joining)
- [ ] 4-space indentation, never tabs
- [ ] 2 blank lines between top-level definitions
- [ ] 1 blank line between method definitions within a class
- [ ] No semicolons to end lines or combine statements
- [ ] No unnecessary parentheses in `return`/`if`

**Naming**
- [ ] Classes: `CapWords`; Functions/methods/variables: `lower_with_under`
- [ ] Constants: `CAPS_WITH_UNDER`; Modules: `lower_with_under`
- [ ] Internal: prefix with single underscore `_`; avoid double underscore `__`
- [ ] No single-char names except `i/j/k/v/e/f` and math notation with citation

**Docstrings & Comments**
- [ ] Triple double-quotes `"""` for all docstrings
- [ ] First line: one-line summary ending in `.`, `?`, or `!`
- [ ] Public functions have `Args:`, `Returns:`, `Raises:` sections where applicable
- [ ] Inline comments explain *why*, not *what*
- [ ] TODO format: `# TODO: <url> - <explanation>`

**Imports**
- [ ] Grouped: `__future__` → stdlib → third-party → local, with blank lines between groups
- [ ] Alphabetical within each group

**Structure**
- [ ] Executable scripts use `if __name__ == '__main__':` guard
- [ ] Logic in `main()` function, not at module level
- [ ] Functions ≤ ~40 lines; prefer small, focused functions

## Boundaries

- Focus on formatting, naming, documentation — not language rules (see `python-language-rules`).
- Source of truth: https://google.github.io/styleguide/pyguide.html sections 3.1–3.18
