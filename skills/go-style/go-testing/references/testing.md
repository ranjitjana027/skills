# Go Style Guide: Testing Reference

Source: https://google.github.io/styleguide/go/decisions and https://google.github.io/styleguide/go/best-practices

---

## Test Helpers vs. Assertion Helpers

**Use test helpers for setup/teardown — call `t.Helper()`:**

```go
func setupDB(t *testing.T) *sql.DB {
    t.Helper()
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatalf("setupDB: %v", err)
    }
    t.Cleanup(func() { db.Close() })
    return db
}
```

**Assertion libraries are not idiomatic — avoid them.** Use plain Go with `cmp`:

```go
import "github.com/google/go-cmp/cmp"

// YES
if diff := cmp.Diff(want, got); diff != "" {
    t.Errorf("Process() mismatch (-want +got):\n%s", diff)
}

if !cmp.Equal(want, got) {
    t.Errorf("Process() = %v, want %v", got, want)
}

// NO — assertion library
assert.Equal(t, want, got)
require.NoError(t, err)
```

**For protocol buffers, use `protocmp.Transform()`:**

```go
import "google.golang.org/protobuf/testing/protocmp"

if diff := cmp.Diff(wantProto, gotProto, protocmp.Transform()); diff != "" {
    t.Errorf("mismatch (-want +got):\n%s", diff)
}
```

---

## Test Failure Messages

Format: `FunctionName(inputs) = got, want expected`

```go
// YES
t.Errorf("Reverse(%q) = %q, want %q", input, got, want)
t.Errorf("Add(%d, %d) = %d, want %d", a, b, got, want)

// For diffs — label which direction is got vs want
t.Errorf("ParseConfig(%q) mismatch (-want +got):\n%s", input, diff)

// NO — missing context
t.Errorf("wrong result: %v", got)
t.Errorf("expected %v but got %v", want, got)  // wrong order — got first
```

**Rules:**
- Include the function name even if it seems obvious.
- Include inputs in failure messages if they are short.
- Print **actual (got) before expected (want)**: `"got X, want Y"`.
- Use named test cases for large or opaque inputs.

---

## Full Structure Comparisons

Compare the entire structure — **not field by field:**

```go
// YES
want := &User{Name: "Alice", Age: 30, Active: true}
if diff := cmp.Diff(want, got); diff != "" {
    t.Errorf("CreateUser() mismatch (-want +got):\n%s", diff)
}

// NO — field by field
if got.Name != "Alice" {
    t.Errorf("Name = %q, want %q", got.Name, "Alice")
}
if got.Age != 30 {
    t.Errorf("Age = %d, want %d", got.Age, 30)
}
```

**Do not compare unstable outputs** (e.g., raw JSON serialization with non-deterministic key order). Parse and compare semantic content:

```go
// YES — parse and compare
var gotObj, wantObj map[string]any
json.Unmarshal(gotJSON, &gotObj)
json.Unmarshal(wantJSON, &wantObj)
if diff := cmp.Diff(wantObj, gotObj); diff != "" { ... }

// NO — raw string comparison
if string(gotJSON) != string(wantJSON) { ... }
```

---

## `t.Error` vs `t.Fatal`

**Prefer `t.Error`** — allows the test to continue running and report all failures:

```go
// YES — multiple failures reported
if got.Name != want.Name {
    t.Errorf("Name = %q, want %q", got.Name, want.Name)
}
if got.Age != want.Age {
    t.Errorf("Age = %d, want %d", got.Age, want.Age)
}

// NO — stops at first failure, hiding others
if got.Name != want.Name {
    t.Fatalf("Name = %q, want %q", got.Name, want.Name)
}
```

**Use `t.Fatal` for:**
- Test setup failures (can't proceed without the resource)
- Table-driven test setup **before** the loop starts
- Inside `t.Run` subtests (scoped to that subtest)

```go
// YES — Fatal in setup
db := setupDB(t)
if db == nil {
    t.Fatal("setupDB returned nil")
}

// YES — Fatal inside subtest is scoped
for _, tc := range tests {
    t.Run(tc.name, func(t *testing.T) {
        got, err := parse(tc.input)
        if err != nil {
            t.Fatalf("parse(%q) unexpected error: %v", tc.input, err)
        }
        if diff := cmp.Diff(tc.want, got); diff != "" {
            t.Errorf("mismatch (-want +got):\n%s", diff)
        }
    })
}
```

---

## Goroutines in Tests

Only the **test function's goroutine** may call `t.FailNow`, `t.Fatal`, `t.FailNow`, `t.SkipNow`.

Goroutines spawned during tests must call `t.Error` and return:

```go
// YES
func TestConcurrent(t *testing.T) {
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            result := compute(i)
            if result != expected(i) {
                t.Errorf("compute(%d) = %d, want %d", i, result, expected(i))
                return  // return, not t.Fatal
            }
        }(i)
    }
    wg.Wait()
}

// NO — t.Fatal in goroutine causes panic
go func() {
    if err != nil {
        t.Fatalf("unexpected error: %v", err)  // wrong — panics
    }
}()
```

---

## Table-Driven Tests

Use struct field names in literals. Use `t.Run` for isolated subtest reporting.

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name string
        a, b int
        want int
    }{
        {name: "both positive", a: 2, b: 3, want: 5},
        {name: "negative result", a: 1, b: -4, want: -3},
        {name: "zeros", a: 0, b: 0, want: 0},
    }

    for _, tc := range tests {
        t.Run(tc.name, func(t *testing.T) {
            if got := Add(tc.a, tc.b); got != tc.want {
                t.Errorf("Add(%d, %d) = %d, want %d", tc.a, tc.b, got, tc.want)
            }
        })
    }
}
```

**Rules:**
- Always use `t.Run` for subtests — isolated failures, parallelism-safe.
- Use field names in struct literals for readability (especially with large cases or adjacent same-type fields).
- Keep setup scoped to the tests that need it.
- Do not use `TestMain` for per-test setup.

---

## Test Double Packages

Create a `[package]test` package with a validation function:

```go
// Package creditcardtest provides test utilities for the creditcard package.
package creditcardtest

// CheckProcessor exercises the Processor interface and reports any violations.
// Returns all errors encountered, not just the first.
func CheckProcessor(t *testing.T, p creditcard.Processor) {
    t.Helper()
    // test valid charge
    if err := p.Charge(ctx, validCard, 100); err != nil {
        t.Errorf("Charge(validCard, 100) = %v, want nil", err)
    }
    // test invalid card
    if err := p.Charge(ctx, invalidCard, 100); err == nil {
        t.Errorf("Charge(invalidCard, 100) = nil, want error")
    }
}
```

---

## Using Real Transports

Prefer real HTTP/RPC clients connected to test doubles over hand-implemented clients:

```go
// YES — real HTTP client, fake server
server := httptest.NewServer(fakeHandler)
defer server.Close()
client := &http.Client{}
resp, err := client.Get(server.URL + "/api/item")

// NO — custom fake HTTP client with manual request/response handling
type fakeClient struct { ... }
func (f *fakeClient) Get(url string) (*http.Response, error) { ... }
```
