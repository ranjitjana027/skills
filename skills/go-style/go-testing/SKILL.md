---
name: go-testing
description: Enforce Go testing best practices when writing or reviewing Go tests. Use when writing table-driven tests, test helpers, assertion logic, failure messages, goroutines in tests, subtests, test doubles, or benchmarks. Trigger when the user asks about Go test structure, how to write Go tests idiomatically, t.Error vs t.Fatal, cmp.Equal vs reflect.DeepEqual, test naming, or test package organization in Go.
---

# Go Style Guide: Testing

## Workflow

1. Read `references/testing.md` for all test style rules: helpers, failure messages, table-driven tests, goroutines, and assertions.
2. Apply rules while writing or reviewing tests.
3. Pair with `go-language-rules` for error handling patterns used in tests and `go-style-rules` for naming.

## Quick Checklist

- [ ] Test helpers call `t.Helper()` — assertion libraries are not idiomatic, avoid them
- [ ] Use `cmp.Equal` / `cmp.Diff` (not `reflect.DeepEqual` or field-by-field)
- [ ] Use `protocmp.Transform()` when comparing protocol buffers
- [ ] Failure messages: include function name, inputs, got, want — e.g., `YourFunc(%v) = %v, want %v`
- [ ] Print actual before expected: `"got X, want Y"`
- [ ] Compare full structures, not field by field
- [ ] Prefer `t.Error` to keep test running; use `t.Fatal` only for fatal setup failures
- [ ] Goroutines in tests must call `t.Error` and return — never `t.Fatal`
- [ ] Table-driven tests: use field names in struct literals, use `t.Run` for subtests
- [ ] Test failure in `t.Run` subtests uses `t.Fatal` — reports cleanly per subtest
- [ ] Test helper packages named `[package]test` (e.g., `creditcardtest`)
- [ ] Do not define flags in tests; do not use `TestMain` for per-test setup

## Boundaries

- Focus on test code style — not production language rules (see `go-language-rules`).
- Source of truth: https://google.github.io/styleguide/go/decisions and https://google.github.io/styleguide/go/best-practices
