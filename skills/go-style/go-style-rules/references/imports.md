# Go Style Guide: Imports Reference

Source: https://google.github.io/styleguide/go/decisions

---

## Import Grouping

`goimports` enforces the group order. Use **blank lines** between groups.

**Group order:**
1. Standard library packages
2. Other packages (project-internal and third-party)
3. Protocol Buffer imports
4. Side-effect imports (`_` imports)

```go
import (
    "context"
    "fmt"
    "net/http"

    "github.com/myorg/myproject/config"
    "github.com/myorg/myproject/internal/db"
    "golang.org/x/oauth2"

    pb "github.com/myorg/myproject/proto/gen/foo_go_proto"

    _ "github.com/lib/pq"  // register postgres driver
)
```

---

## Import Renaming

**Only rename to avoid collisions.** Renamed imports must follow standard naming rules.

```go
// YES — collision avoidance
import (
    "net/http"

    httpclient "github.com/myorg/internal/http"
)

// YES — protobuf packages use pb suffix
import (
    pb "github.com/myorg/proto/gen/foo_go_proto"
    typespb "google.golang.org/protobuf/types/known/typespb"
)

// NO — renaming for style preference
import (
    h "net/http"   // unnecessary alias
)
```

Be consistent: always use the same local name for the same imported package across the codebase.

---

## Blank (`_`) Imports

Only in `main` packages or test files that require them:

```go
// YES — main package, registering a driver
// main.go
import _ "github.com/lib/pq"

// YES — nogo bypass (rare, documented)
import _ "unsafe"

// YES — embed (requires blank import)
import _ "embed"

// NO — library package
// mylib/reader.go
import _ "github.com/lib/pq"  // side effects don't belong in library code
```

---

## Dot (`.`) Imports

**Never use.** Makes it impossible to tell where identifiers come from.

```go
// NO
import . "math"
sin := Sin(x)  // where does Sin come from?

// YES
import "math"
sin := math.Sin(x)
```
