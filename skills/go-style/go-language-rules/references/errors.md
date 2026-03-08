# Go Style Guide: Error Handling Reference

Source: https://google.github.io/styleguide/go/decisions and https://google.github.io/styleguide/go/best-practices

---

## Returning Errors

- `error` is always the **last** return parameter.
- Return `nil` to signal success.
- Exported functions return the `error` interface type — not a concrete error type.

```go
// YES
func ReadFile(path string) ([]byte, error) { ... }
func Connect(addr string) (*Conn, error) { ... }

// NO — exported concrete error type leaks implementation
func ReadFile(path string) ([]byte, *os.PathError) { ... }
```

---

## Error Strings

- **Not capitalized** (unless starting with an exported name, proper noun, or acronym).
- **No ending punctuation.**
- These rules apply to the error string itself; full log messages / UI text may be capitalized.

```go
// YES
fmt.Errorf("invalid argument: %w", err)
errors.New("connection refused")
fmt.Errorf("URL %q not found", url)

// NO
fmt.Errorf("Invalid argument: %w", err)   // capitalized
fmt.Errorf("connection refused.")          // trailing period
```

---

## Handling Errors — Make an Explicit Choice

Every error must be handled. Choose one:
1. **Handle immediately** — log, recover, or take corrective action.
2. **Return to caller** — propagate with context.
3. **Terminate** — `log.Fatal` / `panic` only in exceptional circumstances.

If an error is intentionally ignored:
```go
_ = os.Remove(tmpFile)  // best-effort cleanup; ignore error
// If ignoring, add a comment explaining why it is safe.
```

---

## In-Band Errors

Do not use sentinel values (`-1`, `nil`, `""`) to signal errors — use multiple return values:

```go
// YES
func Lookup(key string) (value string, ok bool) { ... }

val, ok := Lookup("foo")
if !ok {
    // handle missing key
}

// NO — sentinel value
func Lookup(key string) string {
    if notFound {
        return ""  // ambiguous: empty string or not found?
    }
    return value
}
```

---

## Indent Error Flow

Handle errors first. Normal code follows **flat** (not indented in an `else`):

```go
// YES
if err := doSomething(); err != nil {
    return err
}
// normal code continues here, unindented
result := process(data)
return result

// NO — normal code indented in else
if err := doSomething(); err != nil {
    return err
} else {
    // this nesting is unnecessary
    result := process(data)
    return result
}
```

---

## Error Structure

Give errors structure for programmatic inspection — **not string matching**:

```go
// YES — sentinel errors
var ErrNotFound = errors.New("not found")
if errors.Is(err, ErrNotFound) { ... }

// YES — structured error types
type ValidationError struct {
    Field   string
    Message string
}
func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}
var ve *ValidationError
if errors.As(err, &ve) { ... }

// NO — string matching
if strings.Contains(err.Error(), "not found") { ... }
```

---

## Wrapping Errors: `%v` vs `%w`

Use **`%w`** to preserve the error chain (for `errors.Is` / `errors.As`):

```go
// YES — preserving error chain for programmatic inspection
return fmt.Errorf("reading config file: %w", err)
```

Use **`%v`** when:
- Adding context at external system boundaries (RPC, storage, IPC) — creates a fresh independent error.
- The wrapped error is not part of the public API contract.

```go
// YES — external boundary, wrap as string
return fmt.Errorf("rpc call to service failed: %v", err)
```

Place `%w` at the **end** of the format string — error chain reads newest-to-oldest:

```go
// YES
fmt.Errorf("connecting to %s: %w", addr, err)
// Output: "connecting to localhost:8080: connection refused"

// NO — %w in the middle
fmt.Errorf("failed: %w while connecting to %s", err, addr)
```

---

## Error Logging

- Log information **beyond** what the return value conveys.
- Avoid duplicating information — let the caller decide to log or handle.
- Avoid logging PII.

**Log levels:**
```go
log.Info(...)    // general operational info
log.Warning(...) // something unexpected but handled
log.Error(...)   // use sparingly — triggers disk flush

log.V(1).Info(...)  // small extra debugging info
log.V(2).Info(...)  // trace info
log.V(3).Info(...)  // dump internal state

// Guard expensive log calls:
if log.V(2) {
    log.Infof("request dump: %v", expensiveDump(req))
}
```
