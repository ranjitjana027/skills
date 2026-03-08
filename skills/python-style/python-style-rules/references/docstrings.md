# Python Style Guide: Docstrings & Comments Reference

Source: https://google.github.io/styleguide/pyguide.html (Section 3.8)

---

## 3.8 Comments and Docstrings

### 3.8.1 Docstrings — Core Rules

- Always use triple double-quotes `"""`.
- First line: one-line summary (≤80 chars), ends with `.`, `?`, or `!`.
- If more content: blank line after summary, then body at same indentation.
- Closing `"""` on its own line for multi-line docstrings.

```python
def fetch_data(url: str) -> bytes:
    """Fetch raw bytes from the given URL."""
    ...


def process(items: list[str], max_count: int = 100) -> list[str]:
    """Process and filter items according to business rules.

    Applies normalization, deduplication, and truncation.

    Args:
        items: Raw strings to process.
        max_count: Maximum number of items to return.

    Returns:
        A filtered and normalized list of strings, at most max_count items.

    Raises:
        ValueError: If items is empty.
    """
    ...
```

---

### 3.8.2 Module Docstrings

Start the file with license boilerplate (if applicable), then the module docstring.

```python
# Copyright 2024 Google LLC
#
# Licensed under the Apache License, Version 2.0 ...

"""Utilities for parsing and validating configuration files.

This module provides functions for loading YAML/JSON configs, validating
required fields, and merging layered configuration sources.

Example usage:
    config = load_config('settings.yaml')
    validate_config(config, schema=REQUIRED_SCHEMA)
"""
```

---

### 3.8.3 Function and Method Docstrings

**Required when:** function is part of the public API, is non-trivial, or contains non-obvious logic.

**Standard sections:**

#### `Args:`
Document each parameter:
- Name, then colon, then description
- Include type if not already in the annotation
- For `*args` and `**kwargs`: describe as `*args` and `**kwargs`
- Use hanging indent for long descriptions

```python
def create_user(
    name: str,
    email: str,
    role: str = 'viewer',
    *,
    send_welcome: bool = True,
) -> User:
    """Create a new user account.

    Args:
        name: Full display name of the user.
        email: Email address, used as the login identifier.
        role: Access role. One of 'admin', 'editor', 'viewer'.
        send_welcome: If True, sends a welcome email upon creation.

    Returns:
        The newly created User object.

    Raises:
        ValueError: If email is already registered.
        PermissionError: If the caller lacks admin privileges.
    """
```

#### `Returns:` / `Yields:`
- Describe the return value and its type (if not obvious from annotation)
- Omit if the function only returns `None`
- Use `Yields:` for generators

```python
def get_active_users() -> list[User]:
    """Fetch all currently active users.

    Returns:
        A list of User objects sorted by last login date, descending.
    """


def stream_events(source: str):
    """Stream events from the given source.

    Args:
        source: Event source identifier.

    Yields:
        Event objects as they arrive from the source.
    """
```

#### `Raises:`
Document exceptions that are part of the interface (not implementation details):

```python
    """...

    Raises:
        FileNotFoundError: If the config file does not exist.
        PermissionError: If the process cannot read the config file.
        yaml.YAMLError: If the file is not valid YAML.
    """
```

---

### 3.8.4 Class Docstrings

- Placed directly below the `class` definition line.
- One-line summary describing what an **instance** represents.
- Public attributes (excluding properties) documented in `Attributes:` section.

```python
class DataPipeline:
    """A configurable data processing pipeline.

    Manages a sequence of transformation steps applied to input data
    in order, with optional parallel execution.

    Attributes:
        name: Human-readable pipeline identifier.
        steps: Ordered list of transformation step names.
        parallel: Whether steps run in parallel when possible.
    """

    def __init__(self, name: str, steps: list[str], parallel: bool = False):
        self.name = name
        self.steps = steps
        self.parallel = parallel
```

---

### 3.8.5 Block and Inline Comments

**Use for tricky, non-obvious, or important code.** Never just describe what the code does.

```python
# YES — explains the *why*
# We use a set here to allow O(1) membership tests during the inner loop.
seen = set()

# Account for the fact that the API returns timestamps in UTC but the
# database stores them in the local timezone.
adjusted_time = utc_time.astimezone(LOCAL_TZ)

# NO — states the obvious
# Create an empty list
items = []

# Increment i by 1
i += 1
```

**Inline comment format:**
- 2+ spaces from code
- `#` then one space, then comment text

```python
x = x + 1  # compensate for border
```

---

### 3.8.6 Punctuation, Spelling, and Grammar

Write comments like prose:
- Proper capitalization: start with a capital letter
- End complete sentences with a period
- Correct spelling and grammar

```python
# YES
# This handles the edge case where the user has no subscription.
# See go/subscription-design for details.

# NO
# handles edge case where user has no subscription
# see go/subscription-design for details
```

---

## Summary: Docstring Template

```python
def my_function(param1: type1, param2: type2 = default) -> return_type:
    """One-line summary ending with period.

    Extended description of behavior, edge cases, or algorithm details.
    Can be multiple paragraphs.

    Args:
        param1: Description of param1.
        param2: Description of param2. Defaults to `default`.

    Returns:
        Description of what is returned.

    Raises:
        ErrorType: When and why this error is raised.
    """
```
