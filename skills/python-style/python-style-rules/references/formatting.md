# Python Style Guide: Formatting Reference

Source: https://google.github.io/styleguide/pyguide.html (Sections 3.1–3.7, 3.10–3.15, 3.17–3.18)

---

## 3.1 Semicolons

**Never** terminate lines with semicolons.
**Never** use semicolons to put two statements on one line.

```python
# YES
x = 1
y = 2

# NO
x = 1; y = 2
x = 1;
```

---

## 3.2 Line Length

**Maximum: 80 characters.**

**Exceptions (no limit):**
- Long import statements
- URLs in comments
- Pathnames
- Long string constants without natural whitespace

**No backslash for line continuation.** Use implicit joining inside parentheses, brackets, or braces:

```python
# YES
result = (
    some_long_variable_name
    + another_long_variable_name
    + yet_another_variable
)

foo_bar(
    self,
    width=None,
    height=None,
    color='black',
)

# NO
result = some_long_variable_name + \
    another_long_variable_name
```

Break at the highest syntactic level possible.

---

## 3.3 Parentheses

Use sparingly.

```python
# YES
return foo
return (foo, bar)    # returning a tuple
if x and y:
for x in range(10):

# NO
return (foo)         # unnecessary parens
if (x and y):        # unnecessary parens
for (x) in range(10):
```

Single-element tuple requires a trailing comma:
```python
onesie = (foo,)
```

---

## 3.4 Indentation

**4 spaces per level. Never tabs.**

Align wrapped elements vertically, OR use a 4-space hanging indent:

```python
# YES — 4-space hanging indent
foo(
    long_argument_one,
    long_argument_two,
    long_argument_three,
)

# YES — vertical alignment
foo(long_argument_one,
    long_argument_two,
    long_argument_three)

# NO — mixed/unclear
foo(long_argument_one,
  long_argument_two)
```

### 3.4.1 Trailing Commas

Recommended when the closing `]`/`)`/`}` is on a different line from the last element. Also required for single-element tuples.

```python
# YES — trailing comma signals "keep one item per line"
golomb4 = [
    0,
    1,
    3,
    6,
]
```

---

## 3.5 Blank Lines

- **2 blank lines** between top-level definitions (functions, classes)
- **1 blank line** between method definitions within a class
- **1 blank line** between a class docstring and the first method
- **No blank line** directly after a `def` line

```python
class Foo:
    """A foo."""

    def method_one(self):
        pass

    def method_two(self):
        pass


class Bar:
    pass
```

---

## 3.6 Whitespace

**No space inside:** parentheses, brackets, braces; before commas/semicolons/colons; before open paren for calls or indexing; trailing whitespace.

```python
# YES
spam(ham[1], {eggs: 2})
foo(bar)
dct['key'] = lst[index]

# NO
spam( ham[ 1], { eggs: 2} )
foo (bar)
dct ['key'] = lst [index]
```

**Space required around** binary/comparison/boolean operators:
```python
# YES
x = y + z
if x == y:
result = a and b or c
```

**`=` in keyword arguments and defaults:**
```python
# NO type annotation — no spaces around =
def func(a, b=0):
func(x, y=10)

# WITH type annotation — spaces required
def func(a: int, b: int = 0):
```

---

## 3.7 Shebang Line

Most `.py` files do not need one.

For directly executable main scripts:
```python
#!/usr/bin/env python3
```

---

## 3.10 Strings

**Prefer f-strings for interpolation.** Also acceptable: `%` operator, `.format()`.
**Never** use `+` concatenation inside loops.

```python
# YES
name = 'World'
greeting = f'Hello, {name}!'
log_msg = 'User %s logged in' % username

# NO — loop concatenation
result = ''
for item in items:
    result += str(item)  # creates a new string each time

# YES — join from list
result = ''.join(str(item) for item in items)
```

Pick one quote style (`'` or `"`) and be consistent within a file. Docstrings must always use `"""`.

### 3.10.1 Logging

Use `%`-style pattern strings as the first argument — **not f-strings:**

```python
# YES
logger.info('Processing item %d of %d', current, total)
logger.error('Failed to connect: %s', error)

# NO — f-strings prevent log aggregation by pattern
logger.info(f'Processing item {current} of {total}')
```

### 3.10.2 Error Messages

- Must precisely match the actual error condition.
- Interpolated parts must be clearly identifiable.
- Use `f'{var=}'` style to show variable name + value:

```python
raise ValueError(f'Not a probability: {p=}')
# Output: ValueError: Not a probability: p=1.5
```

---

## 3.11 Files, Sockets, and Stateful Resources

**Explicitly close** all files and sockets. Use `with` statements:

```python
# YES
with open('/path/to/file') as f:
    data = f.read()

# YES — contextlib for objects without native context manager
import contextlib
with contextlib.closing(urllib.request.urlopen(url)) as response:
    data = response.read()

# NO
f = open('/path/to/file')
data = f.read()
# f never closed
```

**Never rely on `__del__`** for resource cleanup — GC timing is non-deterministic.

---

## 3.12 TODO Comments

```
# TODO: <resource-link> - <explanation>
```

- `TODO:` in caps
- Followed by a resource link (bug tracker URL, issue number, etc.)
- Avoid personal references (`@username`)
- Include specific date or triggering event for time-based TODOs

```python
# YES
# TODO: crbug.com/192795 - Investigate cpufreq optimizations.
# TODO: github.com/org/repo/issues/42 - Remove after Python 3.12 migration.

# NO
# TODO(john): fix this later
# TODO: fix this
```

---

## 3.13 Imports Formatting

Imports go at the top of the file, after module comments/docstrings, before globals.

**Group order (separated by blank lines):**
1. `from __future__` imports
2. Python standard library
3. Third-party modules
4. Repository sub-package imports

Within each group: **alphabetical by full package path**, case-insensitive.

```python
"""Module docstring."""

from __future__ import annotations

import os
import sys
from typing import Any, Optional

import numpy as np
import requests

from mypackage import models
from mypackage.utils import helpers
```

---

## 3.14 Statements

Generally **one statement per line.**

Exception: test + result can share a line if it fits and there is no `else`:
```python
if foo: bar(foo)  # acceptable

# Cannot combine try/except on one line
```

---

## 3.15 Getters and Setters

Use explicit getter/setter methods when providing meaningful logic or behavior.

```python
# YES — logic present
class Foo:
    def get_voltage(self):
        return self._voltage

    def set_voltage(self, value):
        if value < 0:
            raise ValueError('Voltage cannot be negative')
        self._voltage = value

# YES — no logic, make attribute public
class Bar:
    def __init__(self):
        self.count = 0  # public attribute, no getter needed
```

Do not retroactively bind getter/setters as `@property` if the attribute was previously accessed directly.

---

## 3.17 Main

Put all main logic in a `main()` function and guard with `if __name__ == '__main__':`:

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

With `absl`:
```python
from absl import app

def main(argv):
    del argv  # unused
    ...

if __name__ == '__main__':
    app.run(main)
```

**Do not execute functions or create objects at module level** unless truly necessary.

---

## 3.18 Function Length

- Prefer small, focused functions.
- **~40 lines** is the informal threshold to reconsider — not a hard limit.
- Long functions are harder to read, modify, debug, and test.
- If a function has a long but simple sequence of operations, length is more acceptable than if it has complex logic.
- Extract logically complete sub-operations into helper functions.
