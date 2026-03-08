# Go Style Guide: Formatting & Naming Reference

Source: https://google.github.io/styleguide/go/guide and https://google.github.io/styleguide/go/decisions

---

## Overarching Principles (priority order)

1. **Clarity** — Code purpose and rationale are clear to the reader. Prioritize reading ease over writing ease.
2. **Simplicity** — Accomplish goals in the simplest way. Code reads top-to-bottom without assumed context.
3. **Concision** — High signal-to-noise ratio. Eliminate repetition, opaque names, unnecessary abstraction.
4. **Maintainability** — APIs that grow gracefully; comprehensive tests; avoids unnecessary coupling.
5. **Consistency** — Alignment with the broader codebase. Breaks ties but does not override other principles.

---

## Formatting

All Go source must match `gofmt` output — presubmit checks enforce this.

```bash
gofmt -w ./...
```

- Generated code must also use `format.Source`.
- **No fixed line length limit.** Refactor long lines rather than splitting arbitrarily.
- Do not split lines just because they are long; avoid splits before indentation changes or for long strings (e.g., URLs, error messages).
- Use `MixedCaps` / `mixedCaps` (camelCase) everywhere — **no underscores** in names.

**Exceptions where underscores are allowed:**
- Generated code package names
- Test function names: `TestMyFunc_EdgeCase`
- Low-level OS/cgo interop

---

## Naming

### Package Names

- Lowercase, concise, no underscores.
- Name should clarify call-site usage.

```go
// YES
tabwriter, k8s, oauth2, httputil

// NO
tab_writer, k8s_client, httpUtility
util, common, helper, misc  // too generic
```

### Receiver Names

- Short (1–2 letters), an abbreviation for the type.
- Consistent across all methods on that type.

```go
// YES
type Client struct { ... }
func (c *Client) Send() {}
func (c *Client) Close() {}

// NO
func (client *Client) Send() {}    // too long
func (cl *Client) Close() {}       // inconsistent
func (this *Client) Send() {}      // never use this/self
```

### Constants

- Use `MixedCaps` — no `ALL_CAPS`, no `K`-prefix.
- Name explains what the value **denotes**, not the value itself.

```go
// YES
const MaxRetries = 3
const DefaultTimeout = 30 * time.Second
const StatusOK = 0

// NO
const MAX_RETRIES = 3        // ALL_CAPS
const kMaxRetries = 3        // K-prefix
const Thirty = 30            // describes the value, not the concept
```

### Initialisms and Acronyms

Keep uniform case — never mix:

```go
// YES
URL, url
ID, id
HTTP, http
API, api
RPC, rpc

// NO
Url, Id, Http, Api, Rpc   // mixed case initialisms
```

In names:
```go
// YES
type UserID int64
func ParseURL(s string) (*URL, error)
var httpClient *http.Client

// NO
type UserId int64
func ParseUrl(s string) (*Url, error)
var httpClient *Http.Client
```

### Getters

No `Get`/`get` prefix unless the concept requires it:

```go
// YES
func (u *User) Name() string { return u.name }
func (c *Cache) Count() int  { return c.count }

// NO
func (u *User) GetName() string { return u.name }
func (c *Cache) GetCount() int  { return c.count }
```

### Variable Names

Length proportional to scope, inversely proportional to frequency of use:

```go
// Short scope — short name fine
for i, v := range items { ... }

// Large scope — descriptive name required
var retryDelay = 5 * time.Second

// YES — omit type info
userCount := len(users)
bytesWritten := n

// NO — redundant type suffix
numUsers := len(users)
byteCount := n
```

### Repetition Avoidance

Avoid redundancy between package name and exported symbol:

```go
// Package: config
// YES
config.Load()
config.Parse()

// NO
config.LoadConfig()    // "config" repeated
config.ParseConfig()   // "config" repeated
```

Avoid repeating receiver type in method names:

```go
// YES
func (b *Buffer) WriteTo(w io.Writer) {}

// NO
func (b *Buffer) WriteBufferTo(w io.Writer) {}
```

Avoid repeating parameter names in function names:

```go
// YES
func Override(dest, src *Config) {}

// NO
func OverrideFirstWithSecond(dest, src *Config) {}
```

### Test Double Packages

Append `test` to the production package name:

```go
// Production package: creditcard
// Test double package: creditcardtest

// Single double — simple name
type Stub struct { ... }

// Multiple behaviors — name by behavior
type AlwaysCharges struct { ... }
type AlwaysDeclines struct { ... }

// Multiple types — explicit names
type StubService struct { ... }
type StubStoredValue struct { ... }
```

---

## Commentary

### Doc Comments

- All top-level exported names **require** doc comments.
- Begin with the name of the object being described.
- Complete sentences (capitalized, ending with punctuation).

```go
// Client manages connections to the remote service.
type Client struct { ... }

// Send transmits the request and returns the response.
// It blocks until the response is received or ctx is cancelled.
func (c *Client) Send(ctx context.Context, req *Request) (*Response, error) { ... }
```

Non-doc inline comments may be fragments:
```go
// retry on transient errors
for i := 0; i < maxRetries; i++ {
```

### Package Comments

Appears immediately above the `package` clause — no blank line between:

```go
// Package tabwriter implements a write filter (tabwriter.Writer) that
// translates tabbed columns in input into properly aligned text.
package tabwriter
```

- Exactly **one file** per package should contain the package comment.
- For `main` packages, use the binary name.

### Comment Philosophy

- Explain **why**, not **what**.
- Avoid comments that restate the code.

```go
// YES — explains why
// Use a mutex here instead of atomic to avoid ABA problems
// when the counter wraps around.
mu.Lock()

// Negate the condition for clarity — we want to proceed when there is NO error.
if err == nil {
    process(data)
}

// NO — restates the code
// Lock the mutex
mu.Lock()

// Check if err is nil
if err == nil {
```

### Named Result Parameters

Use when returning multiple same-type values or when clarity requires naming:

```go
// YES — clarifies ambiguous multiple returns
func divide(a, b float64) (quotient, remainder float64) { ... }

// NO — naked returns obscure what is returned
func stats(data []float64) (min, max, mean float64) {
    ...
    return  // unclear
}
```

### Godoc Formatting

- Blank line = new paragraph.
- Two-space indent = verbatim/code block.
- Capitalized line with no punctuation followed by a paragraph = section header.
- Runnable examples go in `_test.go` files as `ExampleFoo` functions.
