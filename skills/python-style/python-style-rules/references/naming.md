# Python Style Guide: Naming Conventions Reference

Source: https://google.github.io/styleguide/pyguide.html (Section 3.16)

---

## 3.16 Naming

### Naming Table

| Type | Public | Internal |
|---|---|---|
| Packages | `lower_with_under` | — |
| Modules | `lower_with_under` | `_lower_with_under` |
| Classes | `CapWords` | `_CapWords` |
| Exceptions | `CapWords` | — |
| Functions | `lower_with_under()` | `_lower_with_under()` |
| Global/Class Constants | `CAPS_WITH_UNDER` | `_CAPS_WITH_UNDER` |
| Global/Class Variables | `lower_with_under` | `_lower_with_under` |
| Instance Variables | `lower_with_under` | `_lower_with_under` |
| Method Names | `lower_with_under()` | `_lower_with_under()` |
| Parameters | `lower_with_under` | — |
| Local Variables | `lower_with_under` | — |

---

### Examples

```python
# Module: my_module.py or _internal_module.py

MAX_RETRIES = 3              # global constant
_DEFAULT_TIMEOUT = 30.0      # internal constant


class HttpClient:            # CapWords class
    """An HTTP client."""

    base_url: str            # public instance variable
    _session: object         # internal instance variable

    def get(self, url: str) -> bytes:          # public method
        return self._fetch(url)

    def _fetch(self, url: str) -> bytes:       # internal method
        ...


class AuthenticationError(RuntimeError):      # Exception — CapWords, ends in Error
    """Raised when authentication fails."""


def parse_response(raw_data: bytes) -> dict:  # function — lower_with_under
    ...


def _normalize_headers(headers: dict) -> dict:  # internal function
    ...
```

---

### 3.16.1 Names to Avoid

**Single-character names** — except for:
- Loop counters: `i`, `j`, `k`, `v`
- Exception variable: `e`
- File handle: `f`
- Private type variables: `_T`, `_P`
- Mathematical notation with a cited source

```python
# YES
for i, item in enumerate(items):
    ...

try:
    ...
except ValueError as e:
    ...

with open(path) as f:
    ...

# NO
def process(x, y, z):  # unclear single chars in function signatures
    ...
```

**Dashes in package/module names:**
```python
# YES
my_module.py
my_package/

# NO
my-module.py
my-package/
```

**`__double_underscore__` dunder names** — reserved for Python language:
```python
# NO — do not create new dunder names
class Foo:
    def __my_magic__(self):  # reserved pattern
        ...
```

**Offensive or confusing terms.**

**Type information embedded in names:**
```python
# YES
id_to_name = {}
user_map = {}

# NO — type suffix is redundant with type annotations
id_to_name_dict = {}
user_mapping_dict = {}
```

---

### 3.16.2 Naming Conventions

**Single underscore `_`** = protected/internal:
```python
class Foo:
    _internal_value = 42       # accessible but signaled as internal
```

**Double underscore `__`** = name mangling (avoid):
```python
class Foo:
    __private = 'hidden'  # becomes _Foo__private — avoid, impacts testability
```

Prefer single underscore `_` for all internal/private members.

**Unit test files:** `test_<method>_<state>` pattern:
```python
# test files
test_parser_empty_input.py
test_client_connection_refused.py

# test methods
def test_parse_returns_none_on_empty():
    ...

def test_connect_raises_on_timeout():
    ...
```

---

### 3.16.3 File Naming

- Always use `.py` extension — enables importing and unit testing
- No dashes in filenames — use underscores
- For executable scripts without `.py`: use a symlink or bash wrapper:

```bash
#!/bin/bash
exec "$0.py" "$@"
```

---

### 3.16.5 Mathematical Notation

Short names allowed when matching a published algorithm or paper.

```python
# YES — matching paper notation
def gaussian_pdf(x, mu, sigma):
    """Compute Gaussian PDF.

    Notation follows Bishop (2006) Pattern Recognition and Machine Learning, p.78.
    """
    # pylint: disable=invalid-name
    return (1 / (sigma * sqrt(2 * pi))) * exp(-0.5 * ((x - mu) / sigma) ** 2)
```

- Cite the source in a comment or docstring.
- Use `# pylint: disable=invalid-name` to suppress lint warnings.
