# Go Style Guide: Language Features Reference

Source: https://google.github.io/styleguide/go/decisions and https://google.github.io/styleguide/go/best-practices

---

## Interfaces

**Define interfaces in the consuming package, not the providing package:**

```go
// Package transport (consumer) — defines the interface it needs
package transport

type Sender interface {
    Send(ctx context.Context, msg *Message) error
}

// Package email (provider) — returns a concrete type
package email

type Client struct { ... }
func (c *Client) Send(ctx context.Context, msg *Message) error { ... }
// Client satisfies transport.Sender without knowing about it
```

**Return concrete types from functions:**

```go
// YES — return concrete type
func NewClient() *Client { return &Client{} }

// NO — return interface prematurely
func NewClient() Sender { return &Client{} }
```

**Do not define interfaces before they are used.** Design for testability via public API, not test doubles.

---

## Goroutine Lifetimes

**Always know when and whether a goroutine exits.** Never start a goroutine without knowing how it will stop.

```go
// YES — clear shutdown via context and WaitGroup
func process(ctx context.Context, items <-chan Item) {
    var wg sync.WaitGroup
    for item := range items {
        wg.Add(1)
        go func(item Item) {
            defer wg.Done()
            select {
            case <-ctx.Done():
                return
            default:
                handle(item)
            }
        }(item)
    }
    wg.Wait()
}

// NO — goroutine with no exit path
func start() {
    go func() {
        for {
            poll()  // runs forever, no way to stop
        }
    }()
}
```

---

## Panics

**Do not use `panic` for normal error handling.**

```go
// YES — return errors
func Parse(s string) (Config, error) {
    if s == "" {
        return Config{}, errors.New("empty input")
    }
    ...
}

// NO — panic on user input
func Parse(s string) Config {
    if s == "" {
        panic("empty input")  // never panic on user input
    }
    ...
}
```

**Appropriate uses of panic:**
- API misuse that mirrors stdlib behavior (e.g., index out of bounds)
- Internal `recover` exists in the same call chain (never crosses package boundaries)
- After `log.Fatal` to satisfy the compiler
- Package `init` when fatal logging is unavailable

**Naming convention for panic-on-misuse functions:**
```go
// MustXYZ — only call at startup/init, never on user input
var re = regexp.MustCompile(`^[a-z]+$`)
db := sql.MustOpen("postgres", dsn)
```

---

## Contexts

- Always the **first parameter**, named `ctx`.
- Never store in a struct field.
- Pass to each method that needs it.

```go
// YES
func (c *Client) Fetch(ctx context.Context, id string) (*Item, error) { ... }
func Process(ctx context.Context, data []byte) error { ... }

// NO
type Worker struct {
    ctx context.Context  // never store context in struct
}

func Fetch(id string) (*Item, error) {  // missing context
    ...
}
```

**Exceptions:**
- HTTP handlers: use `req.Context()` instead of a ctx parameter
- Streaming RPCs: context comes from the stream
- Entrypoints: create with `context.Background()` or `tb.Context()` (Go 1.24+)

**Context is immutable** — safe to pass the same instance to concurrent calls. Do not create custom context types or use interfaces other than `context.Context`.

---

## Receiver Types

| Situation | Use |
|---|---|
| Method mutates receiver | Pointer `*T` |
| Type contains `sync.Mutex` or similar | Pointer `*T` |
| Large struct | Pointer `*T` |
| Small plain value type | Value `T` |
| Maps, functions, channels, slices (unless reslicing) | Value `T` |
| Concurrent access concerns | Pointer `*T` |

**Correctness wins over performance or simplicity.**

**Prefer all pointer or all value receivers for a given type — do not mix:**

```go
// YES — all pointer receivers
func (c *Counter) Increment() { c.n++ }
func (c *Counter) Value() int { return c.n }

// NO — mixed receivers
func (c *Counter) Increment() { c.n++ }  // pointer
func (c Counter) Value() int { return c.n }  // value — inconsistent
```

**Never copy types with pointer-based internals:**
```go
// NO — sync.Mutex must never be copied
mu := sync.Mutex{}
mu2 := mu  // BUG: copies lock state

// NO — bytes.Buffer slice may alias internal array
buf := bytes.Buffer{}
buf2 := buf  // BUG: both share internal slice
```

---

## Variable Declarations

Prefer `:=` for non-zero initialization:
```go
// YES
config := Config{Host: "localhost"}
count := len(items)

// Use var for zero-value initialization where variable is filled later
var result []Item
json.Unmarshal(data, &result)

// Use new() for pointer types, especially protobufs
msg := new(pb.MyMessage)
```

**Preallocate when final size is known:**
```go
// YES
out := make([]string, 0, len(in))
for _, s := range in {
    out = append(out, transform(s))
}

index := make(map[string]int, len(keys))

// NO — over-allocating wastes memory
buf := make([]byte, 0, 1<<20)  // 1MB when 100B is needed
```

**Specify channel direction:**
```go
// YES — directional prevents accidental closure bugs
func producer(out chan<- int) { out <- 42 }
func consumer(in <-chan int) { v := <-in }

// NO — bidirectional when only one direction is used
func producer(out chan int) { out <- 42 }
```

---

## Switch Statements

Go auto-breaks at the end of each `case` — no `break` needed:

```go
// YES
switch status {
case http.StatusOK:
    handleOK(resp)
case http.StatusNotFound:
    handleNotFound(resp)
default:
    handleOther(resp)
}

// NO — redundant break
switch status {
case http.StatusOK:
    handleOK(resp)
    break  // unnecessary
```

Use `fallthrough` for intentional C-style fall-through. Use labeled `break` to exit a `for` loop from inside a `switch`:

```go
loop:
for _, item := range items {
    switch item.Type {
    case TypeDone:
        break loop  // exits the for loop
    case TypeSkip:
        continue loop
    }
}
```

Empty `case` clauses must have a comment:
```go
switch event.Type {
case EventStart:
    start()
case EventStop:
    // intentionally no-op during shutdown
}
```

---

## Synchronous Functions

Prefer synchronous over asynchronous — callers can add concurrency if needed:

```go
// YES — synchronous, easy to test and reason about
func FetchItem(ctx context.Context, id string) (*Item, error) {
    return db.Get(ctx, id)
}

// Caller adds concurrency only when needed
results := make(chan *Item, len(ids))
for _, id := range ids {
    go func(id string) {
        item, _ := FetchItem(ctx, id)
        results <- item
    }(id)
}
```

---

## Generics

Allowed where fulfilling real requirements. Avoid premature use:

```go
// YES — genuine need for type-parameterized container
func Map[S, T any](s []S, f func(S) T) []T { ... }

// NO — premature generics when a concrete type would do
func Process[T any](items []T) {}  // if T is always string, just use string
```

Do not use generics to build DSLs or for algorithms that are type-indifferent.

---

## Flags

- Flag names use **underscores**: `poll_interval`.
- Corresponding Go variables use **camelCase**: `pollInterval`.
- Define flags **only in `package main`**.
- Library packages configured via APIs, not flags.
- Group flag variables in their own `var` block after imports.

```go
// main.go
var (
    pollInterval = flag.Duration("poll_interval", time.Minute, "how often to poll")
    serverAddr   = flag.String("server_addr", ":8080", "server listen address")
)
```

---

## Cryptography

**Never use `math/rand` for secrets or keys:**

```go
// YES
import "crypto/rand"
import "encoding/hex"

func generateToken() (string, error) {
    b := make([]byte, 32)
    if _, err := rand.Read(b); err != nil {
        return "", err
    }
    return hex.EncodeToString(b), nil
}

// NO — predictable, not cryptographically secure
import "math/rand"
token := rand.Int63()
```

---

## `any` vs `interface{}`

Use `any` in new code (Go 1.18+):

```go
// YES
var m map[string]any
func Accept(v any) {}

// Outdated
var m map[string]interface{}
func Accept(v interface{}) {}
```
