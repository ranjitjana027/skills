---
name: go-language-rules
description: Enforce Go language feature best practices when writing or reviewing Go code. Use when handling errors, using interfaces, goroutines, panics, contexts, receiver types, generics, variable declarations, struct copying, switch statements, synchronous vs asynchronous design, or passing values vs pointers. Trigger when the user asks about idiomatic Go patterns, error handling, goroutine lifecycle, context usage, interface design, pointer vs value receivers, or any Go language feature question.
license: Complete terms in LICENSE.txt
---

# Go Style Guide: Language Rules

## Workflow

1. Read `references/errors.md` for error handling, wrapping, logging, and in-band error rules.
2. Read `references/language-features.md` for interfaces, goroutines, panics, contexts, generics, receivers, and variable declarations.
3. When suggesting fixes, explain the principle violated (Clarity, Simplicity, Correctness, Maintainability).
4. Pair with `go-style-rules` for naming/formatting and `go-testing` for test-specific patterns.

## Quick Checklist

**Errors**
- [ ] `error` is the final return parameter; return `nil` for success
- [ ] Exported functions return `error` type, not concrete error types
- [ ] Error strings: not capitalized, no ending punctuation
- [ ] No sentinel values (`-1`, `""`, `nil`) — use `(value, ok bool)` multiple returns
- [ ] Error flow indented first; normal code follows flat after `if err != nil`
- [ ] `%w` to preserve error chain for `errors.Is`/`errors.As`; `%v` at external boundaries

**Language Features**
- [ ] Interfaces defined in the consuming package, not the providing package
- [ ] Return concrete types from functions, not interfaces
- [ ] No `panic` for normal error handling
- [ ] Goroutines always have a known exit strategy (context cancellation, `sync.WaitGroup`)
- [ ] `context.Context` is the first parameter, named `ctx`, never a struct field
- [ ] Never use `math/rand` for secrets — use `crypto/rand`
- [ ] Pointer receivers for: mutating methods, types with `sync.Mutex`, large structs
- [ ] Do not mix pointer and value receivers on the same type
- [ ] Do not copy structs with `sync.Mutex` or pointer-based internals (`bytes.Buffer`)
- [ ] Prefer synchronous functions over async; callers add concurrency if needed
- [ ] `any` instead of `interface{}` in new code (Go 1.18+)
- [ ] Flags defined only in `package main`, never in library packages

## Boundaries

- Focus on language correctness and idiomatic patterns — not formatting/naming (see `go-style-rules`).
- Source of truth: https://google.github.io/styleguide/go/decisions and https://google.github.io/styleguide/go/best-practices
