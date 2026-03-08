# Python Style Guide: Type Annotations Reference

Source: https://google.github.io/styleguide/pyguide.html (Section 3.19)

---

## 3.19.1 General Rules

**Annotate public APIs.** Also annotate:
- Code that is error-prone or complex
- Code that has matured and is unlikely to change

**Do not annotate:**
- `self` and `cls` parameters (exception: when using `Self` from `typing`)
- `__init__` return type (always `None`, implicit)

```python
from typing import Self

class Builder:
    def set_name(self, name: str) -> Self:  # Self is acceptable here
        self.name = name
        return self

    def __init__(self) -> None:  # NO — do not annotate __init__ return
        ...

    def build(self) -> 'Product':
        ...
```

Use pytype or mypy for static checking.

---

## 3.19.4 Default Values

**Spaces around `=` only when the argument has BOTH an annotation AND a default:**

```python
# YES — annotation present, spaces required
def func(a: int = 0, b: str = 'default') -> None:
    ...

# YES — no annotation, no spaces
def func(a, b=0):
    ...

# NO — inconsistent
def func(a: int=0):  # missing spaces
def func(a, b =0):   # unexpected space
```

---

## 3.19.5 NoneType

Never leave `None` as an implicit annotation. Always use `Optional` or union syntax.

```python
# YES (Python 3.10+)
def get_user(user_id: int) -> User | None:
    ...

# YES (older style)
from typing import Optional
def get_user(user_id: int) -> Optional[User]:
    ...

# YES (explicit Union)
from typing import Union
def get_user(user_id: int) -> Union[User, None]:
    ...

# NO — implicit None
def get_user(user_id: int) -> User:
    return None  # type: ignore — don't do this
```

For optional parameters:
```python
# YES
def process(data: bytes, encoding: str | None = None) -> str:
    ...

# NO — implicit None annotation
def process(data: bytes, encoding: str = None) -> str:  # wrong
    ...
```

---

## 3.19.6 Type Aliases

Use `TypeAlias` (Python 3.10+) and `CapWords` naming:

```python
from typing import TypeAlias

# Public type alias
Vector: TypeAlias = list[float]
Matrix: TypeAlias = list[list[float]]

# Private type alias (module-internal)
_JsonDict: TypeAlias = dict[str, object]
_Callback: TypeAlias = Callable[[str, int], bool]
```

For older Python, use plain assignment:
```python
# Python 3.9 and earlier
Vector = list[float]
```

---

## 3.19.9 Tuples vs Lists

**Lists:** homogeneous sequences of a single repeated type:
```python
items: list[str]
counts: list[int]
```

**Tuples:** fixed-length heterogeneous sequences, or homogeneous repeated:
```python
# Fixed structure — specify each element type
point: tuple[float, float]
record: tuple[int, str, float]

# Homogeneous variable length — use ellipsis
coords: tuple[float, ...]
```

---

## 3.19.10 Type Variables

Use `TypeVar` for generics. Private type vars (not exported) use underscore prefix:

```python
from typing import TypeVar

# Private — prefixed with underscore
_T = TypeVar('_T')
_KT = TypeVar('_KT')
_VT = TypeVar('_VT')

# Constrained TypeVar
from typing import TypeVar
AddableType = TypeVar('AddableType', int, float, str)

def add(a: AddableType, b: AddableType) -> AddableType:
    return a + b
```

Use `ParamSpec` for decorator signatures:
```python
from typing import ParamSpec, TypeVar, Callable

_P = ParamSpec('_P')
_R = TypeVar('_R')

def logged(func: Callable[_P, _R]) -> Callable[_P, _R]:
    def wrapper(*args: _P.args, **kwargs: _P.kwargs) -> _R:
        print(f'Calling {func.__name__}')
        return func(*args, **kwargs)
    return wrapper
```

---

## 3.19.11 String Types

**Use `str` for text, `bytes` for binary data.** Never use `typing.Text` (Python 2/3 compat only).

```python
from typing import AnyStr

# YES
def encode(text: str) -> bytes:
    return text.encode('utf-8')

def decode(data: bytes) -> str:
    return data.decode('utf-8')

# Use AnyStr only when a function handles either type uniformly
def to_lower(text: AnyStr) -> AnyStr:
    return text.lower()

# NO — deprecated
from typing import Text
def greet(name: Text) -> Text:  # use str instead
    ...
```

---

## 3.19.12 Imports for Typing

Import typing symbols directly:

```python
from typing import Any, Callable, Generic, Optional, Union
from collections.abc import Iterator, Mapping, Sequence
```

**Prefer `collections.abc` over `typing` for abstract base classes:**
```python
# YES
from collections.abc import Sequence, Mapping, Iterator, Generator

# Outdated (still works but prefer abc)
from typing import Sequence, Mapping
```

**Use built-in parametric types over `typing` aliases (Python 3.9+):**
```python
# YES — built-in generics
items: list[str]
mapping: dict[str, int]
pair: tuple[float, float]
result: set[bytes]

# Outdated — typing module aliases
from typing import List, Dict, Tuple, Set
items: List[str]       # use list[str]
mapping: Dict[str, int]  # use dict[str, int]
```

---

## 3.19.13 Conditional Imports

Only in exceptional cases (e.g., avoiding circular imports for type-checking only):

```python
from __future__ import annotations  # enables string-based forward references

from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from mypackage import HeavyClass  # imported only for type checking

def process(obj: 'HeavyClass') -> None:  # quoted string reference
    ...
```

With `from __future__ import annotations`, all annotations are lazily evaluated, so you can reference types without quoting:
```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from mypackage import HeavyClass

def process(obj: HeavyClass) -> None:  # no quotes needed
    ...
```

---

## 3.19.14 Circular Dependencies

Circular imports are a code smell — prefer refactoring.

If unavoidable: replace the circular-dependency module with `Any` in annotations:
```python
from typing import Any

# Instead of importing CircularModule
def use_circular(obj: Any) -> None:
    ...
```

---

## 3.19.15 Generics

**Always specify type parameters.** Unspecified parameters are treated as `Any`:

```python
# YES
def first(seq: Sequence[int]) -> int:
    return seq[0]

items: list[str] = []
mapping: dict[str, int] = {}

# NO — bare generics mean Sequence[Any], list[Any], etc.
def first(seq: Sequence) -> int:  # what type does seq contain?
    ...

items: list = []  # list of what?
```

**Writing generic classes:**
```python
from typing import Generic, TypeVar

_T = TypeVar('_T')

class Stack(Generic[_T]):
    def __init__(self) -> None:
        self._items: list[_T] = []

    def push(self, item: _T) -> None:
        self._items.append(item)

    def pop(self) -> _T:
        return self._items.pop()
```

---

## Common Patterns Quick Reference

```python
# Optional parameter
def foo(x: int | None = None) -> None: ...

# Callable
from collections.abc import Callable
callback: Callable[[int, str], bool]

# Union
from typing import Union
value: Union[int, str, None]  # or: int | str | None

# Any
from typing import Any
data: Any

# Cast (for type narrowing)
from typing import cast
result = cast(int, some_value)

# Literal
from typing import Literal
mode: Literal['r', 'w', 'a']

# TypedDict
from typing import TypedDict
class Config(TypedDict):
    host: str
    port: int
    debug: bool

# Protocol
from typing import Protocol
class Closeable(Protocol):
    def close(self) -> None: ...

# Final
from typing import Final
MAX: Final = 100
```
