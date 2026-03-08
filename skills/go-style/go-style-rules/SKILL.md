---
name: go-style-rules
description: Enforce Go formatting, naming, commentary, and import style best practices when writing or reviewing Go code. Use when checking Go code for naming conventions, package names, receiver names, constants, initialisms, comment format, import grouping, doc comments, or gofmt compliance. Trigger when the user asks to write idiomatic Go, review Go code for style, name Go types/functions/packages/variables, format Go imports, or write Go doc comments.
---

# Go Style Guide: Style Rules

## Workflow

1. Read `references/formatting-naming.md` for formatting, naming, and commentary rules.
2. Read `references/imports.md` for import grouping, renaming, and blank import rules.
3. Apply rules while writing or reviewing. Cite the relevant principle (Clarity, Simplicity, Concision, Maintainability, Consistency) when explaining a decision.
4. Pair with `go-language-rules` for language feature and error handling rules, and `go-testing` for test style.

## Quick Checklist

**Formatting**
- [ ] Code passes `gofmt` — all Go source must match `gofmt` output
- [ ] No arbitrary line splits; refactor long lines instead
- [ ] `MixedCaps` / `mixedCaps` everywhere — no underscores in names (except test funcs, generated code, cgo)

**Naming**
- [ ] Package names: lowercase, no underscores, no generic names (`util`, `common`, `helper`)
- [ ] Receiver names: short (1–2 letters), consistent across all methods on the type
- [ ] Constants: `MixedCaps` — no `ALL_CAPS`, no `K`-prefix
- [ ] Initialisms/acronyms: uniform case (`URL`/`url`, `ID`/`id`, never `Url`/`Id`)
- [ ] No `Get`/`get` prefix on getters: `Counts()` not `GetCounts()`
- [ ] No type-like words in variable names: `userCount` not `numUsers`
- [ ] No redundancy between package name and exported symbol

**Commentary**
- [ ] All top-level exported names have doc comments
- [ ] Doc comments begin with the name of the thing being described
- [ ] Comments are complete sentences (capitalized, punctuated)
- [ ] Comments explain *why*, not *what*
- [ ] Package comment appears immediately above `package` clause with no blank line

**Imports**
- [ ] Groups: stdlib → other/project → protobuf → side-effect (`_`) imports
- [ ] Rename imports only to avoid collisions
- [ ] No dot (`.`) imports
- [ ] Blank (`_`) imports only in `main` packages or tests

## Boundaries

- Focus on formatting, naming, commentary, imports — not language/error rules (see `go-language-rules`).
- Source of truth: https://google.github.io/styleguide/go/guide and https://google.github.io/styleguide/go/decisions
