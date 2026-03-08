# Python Style Guide: Language Rules Reference

Source: https://google.github.io/styleguide/pyguide.html (Sections 2.1–2.21)

---

## 2.1 Lint

Run `pylint` over code using Google's pylintrc config.

- Suppress false positives with `# pylint: disable=rule-name` at the line level.
- Do not suppress at the file level without a documented reason.

---

## 2.2 Imports

**Import packages and modules only — not individual classes, functions, or types.**

```python
# YES
import os
import sys
from package import module
from package import module as alias

# NO
from os.path import join, exists
from package import SomeClass
```

**Allowed exceptions:**
- `from typing import Any, Optional, Union` etc.
- `from collections.abc import Sequence, Mapping`
- `from typing_extensions import ...`
- `from six.moves import ...`

**`from x import y as z` is allowed when:**
- Two modules named `y` conflict
- `y` is overly long or too generic in context
- `y` is not a conventional package suffix

**No relative imports.** Always use full package paths:
```python
# YES
from project.package import module

# NO
from . import module
from ..utils import helper
```

---

## 2.3 Packages

- Import every module by its full pathname location in the package tree.
- Avoids name conflicts and makes modules easier to locate.

---

## 2.4 Exceptions

**Use built-in exceptions for programming mistakes:** `ValueError`, `TypeError`, `RuntimeError`, etc.

**Custom exception classes:**
- Must inherit from an existing exception class
- Name must end in `"Error"`: `class MyValidationError(ValueError): ...`

**Never use bare `except:`:**
```python
# YES
except Exception as e:
    raise
except (TypeError, ValueError):
    ...

# NO
except:
    ...
```

**Never use `assert` to validate preconditions in production code:**
```python
# YES — validate with real checks
if not isinstance(x, str):
    raise TypeError(f'Expected str, got {type(x).__name__}')

# NO — assert can be disabled with -O flag
assert isinstance(x, str), 'x must be a string'
```

**`assert` is for tests and internal invariants only.**

**Minimize code in `try` blocks:**
```python
# YES
try:
    value = collection[key]
except KeyError:
    return key_not_found(key)
else:
    return handle_value(value)

# NO
try:
    # Too much in try block
    value = collection[key]
    result = handle_value(value)
    return result
except KeyError:
    return key_not_found(key)
```

Use `finally` for cleanup (closing files, releasing locks).

---

## 2.5 Mutable Global State

**Avoid mutable global state.**

Module-level **constants** are fine and encouraged:
```python
# YES — constants
MAX_SIZE = 100
_DEFAULT_TIMEOUT = 30.0
ALLOWED_EXTENSIONS: frozenset[str] = frozenset({'jpg', 'png', 'gif'})

# NO — mutable global
_cache = {}         # mutable global — avoid
_registry = []      # mutable global — avoid
```

If mutable global is truly necessary:
- Prefix with underscore (`_cache`)
- Add a module-level comment explaining why

---

## 2.6 Nested/Local/Inner Classes and Functions

**Fine when:** closing over a local variable (other than `self`/`cls`).

**Don't nest solely to hide from module users** — use underscore prefix instead.

```python
# YES — closing over a variable
def make_adder(n):
    def adder(x):
        return x + n
    return adder

# NO — hiding via nesting instead of underscore
def public_api():
    class _Hidden:  # just use _Hidden at module level
        ...
```

**Drawback:** nested functions/classes cannot be directly unit-tested.

---

## 2.7 Comprehensions & Generator Expressions

**Simple cases only. Prioritize readability.**

```python
# YES
result = [x * 2 for x in range(10)]
filtered = [x for x in values if x > 0]
total = sum(x * x for x in range(100))

# NO — multiple for clauses
pairs = [(x, y) for x in range(5) for y in range(5)]  # use explicit loops

# NO — too complex
result = [transform(x) for x in data if condition1(x) if condition2(x)]
```

**Rule:** at most one `for` clause + one optional `if` filter per comprehension.

---

## 2.8 Default Iterators and Operators

**Use default iterators — don't call `.keys()`, `.values()`, `.items()` unnecessarily:**
```python
# YES
for key in adict:
    ...
if key in adict:
    ...

# NO
for key in adict.keys():
    ...
if key in adict.keys():
    ...
```

**Do not mutate a container while iterating over it.**

```python
# YES — iterate over a copy
for key in list(adict):
    if should_remove(key):
        del adict[key]
```

---

## 2.9 Generators

Use for memory efficiency. Prefer over returning a list when callers only need to iterate.

```python
def read_chunks(filename):
    """Yields: chunks of the file."""
    with open(filename) as f:
        while chunk := f.read(8192):
            yield chunk
```

- Use `"Yields:"` section in docstring (not `"Returns:"`).
- Wrap with context manager for resource-holding generators.

---

## 2.10 Lambda Functions

**One-liners only.** Max 60–80 characters.

```python
# YES
fn = lambda x: x * 2
sorted_items = sorted(items, key=lambda x: x.name)

# NO — too long; use a named function
process = lambda data, config, settings: complex_transform(data, config) if settings.enabled else data
```

Prefer `operator` module functions when applicable:
```python
import operator
total = reduce(operator.add, values)  # instead of: reduce(lambda x, y: x + y, values)
```

---

## 2.11 Conditional Expressions

**Single-line ternary is acceptable for simple cases:**
```python
# YES
x = 'yes' if condition else 'no'
value = a if a is not None else default

# NO — too complex for ternary
result = (complex_function(a) if some_condition(a, b) else other_function(b, c))
```

Each portion must fit on one line, or break consistently. Use full `if` statements for complex cases.

---

## 2.12 Default Argument Values

**Never use mutable objects as default values.**

```python
# YES
def process(items=None):
    if items is None:
        items = []
    ...

def connect(timeout=None):
    if timeout is None:
        timeout = DEFAULT_TIMEOUT
    ...

# NO
def process(items=[]):       # mutable default
    ...

def connect(timeout=time.time()):  # evaluated once at definition time
    ...
```

**Why:** default values are evaluated once at function definition. Mutable defaults persist across calls.

---

## 2.13 Properties

**Use `@property` for:**
- Controlling attribute access with logic
- Computing trivially derived values (cheap computation)

```python
# YES
class Circle:
    def __init__(self, radius: float):
        self._radius = radius

    @property
    def radius(self) -> float:
        return self._radius

    @radius.setter
    def radius(self, value: float) -> None:
        if value < 0:
            raise ValueError('radius must be non-negative')
        self._radius = value

    @property
    def area(self) -> float:
        return math.pi * self._radius ** 2
```

**Do NOT use properties for:**
- Simple pass-through getters/setters with no logic — make the attribute public
- Hiding side effects
- Expensive computation
- Complex subclass hierarchies where override is expected

---

## 2.14 True/False Evaluations

**Use implicit falsiness:**
```python
# YES
if foo:
if not bar:
if seq:          # checks if sequence is non-empty
if not seq:

# NO
if foo != []:
if foo == True:
if foo is True:
if len(seq) > 0:
if len(seq) != 0:
```

**Always use `is None` / `is not None`:**
```python
# YES
if x is None:
if x is not None:

# NO
if x == None:
if x != None:
```

**Never compare booleans to `False`:**
```python
# YES
if not x:

# NO
if x == False:
if x is False:
```

**NumPy arrays:** use `.size` attribute to check emptiness.

**Note:** The string `'0'` evaluates as `True`.

---

## 2.16 Lexical Scoping

Closures can read (but not assign) enclosing scope variables without `nonlocal`:

```python
# YES — reading enclosing scope
def outer():
    count = 0
    def inner():
        return count  # OK
    return inner

# Must use nonlocal to assign
def outer():
    count = 0
    def inner():
        nonlocal count
        count += 1  # needs nonlocal
    return inner
```

**Gotcha: loop variable capture:**
```python
# BUG — all lambdas capture the same 'i'
fns = [lambda: i for i in range(5)]  # all return 4

# FIX — capture at definition time
fns = [lambda i=i: i for i in range(5)]
```

---

## 2.17 Function and Method Decorators

Use judiciously. Document decorator behavior in docstrings.

**Never use `@staticmethod`:** write a module-level function instead.
```python
# YES
def _helper(x):
    return x * 2

class MyClass:
    def method(self):
        return _helper(self.value)

# NO
class MyClass:
    @staticmethod
    def helper(x):
        return x * 2
```

**`@classmethod` only for:** named constructors or class-specific routines.

**Decorators must not** depend on external resources (files, sockets, databases) at decoration time.

---

## 2.18 Threading

**Do not rely on built-in type atomicity** (e.g., `dict` operations are not thread-safe everywhere).

```python
# YES
import queue
task_queue = queue.Queue()

import threading
condition = threading.Condition()

# NO
shared_list = []  # multiple threads appending — not safe
```

Prefer `queue.Queue` for inter-thread communication. Use `threading.Condition` over lower-level primitives.

---

## 2.19 Power Features

**Avoid:**
- Custom metaclasses (except `abc.ABCMeta`)
- Bytecode access (`compile`, `exec`, `eval` beyond simple use)
- Dynamic inheritance
- Object reparenting
- `__del__` methods (use context managers instead)
- Import hacks
- Reflection for behavior overriding

**Acceptable:** standard library items that use these internally (`dataclasses`, `enum`, `abc.ABCMeta`).

---

## 2.20 Modern Python: `from __future__` Imports

Encouraged to ease compatibility:
```python
from __future__ import annotations  # PEP 563 — postponed evaluation of annotations
```

Remove only when confident the runtime environment is sufficiently modern.

---

## 2.21 Type Annotated Code

- Annotate public APIs with type hints.
- Use pytype or mypy for checking.
- See `google-python-type-annotations` skill for full rules.
