---
name: python-write
description: Explains how to write Python code following established coding guidelines and best practices. Use when creating new Python code, implementing features, or writing functions that should follow Python conventions.
---

# Python Code Writing

Write Python code following these coding guidelines and best practices.

## Core Principles

- Write clean, maintainable, type-safe code
- Follow established Python conventions
- Prioritize readability and explicitness
- Avoid common pitfalls and anti-patterns

## Python Coding Guidelines

### 1. Variable Names
✓ Use snake_case consistently
✓ Use descriptive names without abbreviations
✗ Avoid single letters (except in comprehensions/plotting)

```python
# Good
page_count = 10
user_name = "Alice"

# Bad
pg_cnt = 10
usr_nm = "Alice"
```

### 2. Type Annotations
✓ Annotate ALL function signatures with types and return values
✓ Use built-in generic types: `list`, `dict`, `set`
✗ Avoid: `from typing import List, Dict, Set`

```python
# Good
def process_data(items: list[str]) -> dict[str, int]:
    result: dict[str, int] = {}
    for item in items:
        result[item] = len(item)
    return result

# Bad
def process_data(items):
    pass
```

### 3. Comments & Docstrings
✓ Comment WHY, not WHAT
✓ Use comments sparingly
✓ Add docstrings only for non-obvious behavior
✗ Don't document types in docstrings (use annotations)

```python
# Good - explains why
# We double values here due to API v2 expecting doubled integers
for item in items:
    result.append(item * 2)

# Bad - explains what (obvious from code)
# Loop through items and append to result
for item in items:
    result.append(item * 2)
```

### 4. Regular Expressions
✓ ALWAYS use raw strings

```python
import re
pattern = re.compile(r"\W+")  # Always use r"..."
```

### 5. Decimals
✓ Pass strings to Decimal constructor
✗ NEVER pass floats

```python
from decimal import Decimal

# Good
value = Decimal("2.2")  # Decimal('2.2')

# Bad
value = Decimal(2.2)    # Decimal('2.200000000000000177...')
```

### 6. Global Variables
✓ Avoid mutable global access
✓ Use factory functions or return copies

```python
# Good - factory function
def get_config() -> dict:
    return {"key": ["value1", "value2"]}

# Bad - direct mutable global access
GLOBAL_CONFIG = {"key": ["value1", "value2"]}

def get_config() -> dict:
    return GLOBAL_CONFIG  # Returns reference!
```

### 7. Default Arguments
✓ Use `None` for mutable defaults
✗ NEVER use `[]`, `{}`, or other mutable objects

```python
# Good
def extend_list(values: list | None = None) -> list:
    if values is None:
        values = []
    values.append(10)
    return values

# Bad - mutable default persists across calls!
def extend_list(values: list = []) -> list:
    values.append(10)
    return values
```

### 8. Dataclass Defaults
✓ Use `field(default_factory=)` for mutable types

```python
from dataclasses import dataclass, field

# Good
@dataclass
class Config:
    settings: dict = field(default_factory=dict)
    items: list = field(default_factory=lambda: [1, 2, 3])
    name: str = "default"  # Immutable types are safe

# Bad - shared across instances!
@dataclass
class Config:
    settings: dict = {}
    items: list = [1, 2, 3]
```

### 9. Properties vs Methods
✓ Prefer explicit methods over properties
✗ Properties hide computational cost

```python
# Good - clear this may be expensive
def get_user_count(self) -> int:
    return db.count_users()

# Bad - hides expensive operation
@property
def user_count(self) -> int:
    return db.count_users()
```

### 10. Early Returns / Fail Fast
✓ Validate failures upfront
✓ Reduce nesting with early returns

```python
# Good
def process(data: dict) -> str | None:
    if not data:
        return None
    if not validate(data):
        return None
    if not transform(data):
        return None
    return result

# Bad - deeply nested
def process(data: dict) -> str:
    if data:
        if validate(data):
            if transform(data):
                return result
```

### 11. Loggers
✓ Use module-level logger with `__name__`
✗ Never use root logger

```python
import logging

logger = logging.getLogger(__name__)

def do_work():
    logger.info("Working...")

# With structlog:
import structlog

logger = structlog.get_logger(__name__)

def do_work():
    logger.info("Working...")
```

### 12. Logging Parameters
✓ Use parameter passing for structured logging
✗ Avoid f-strings in log messages

```python
# Standard logging - good
logger.error("Error occurred: %s", error_msg)

# Structlog - good
logger.error("Error occurred", message=error_msg, user_id=user_id)

# Bad - can't be grouped by downstream tools
logger.error(f"Error occurred: {error_msg}")
```

### 13. Exception Handling
✓ Use `except Exception:` to catch exceptions
✗ NEVER use bare `except:`

```python
# Good - allows KeyboardInterrupt to propagate
try:
    risky_operation()
except Exception:
    logger.exception("Operation failed")

# Bad - catches everything including KeyboardInterrupt
try:
    risky_operation()
except:
    logger.exception("Operation failed")
```

### 14. String Prefix/Suffix Removal
✓ Use `removeprefix()` and `removesuffix()`
✗ NEVER use `lstrip()` or `rstrip()` for this purpose

```python
# Good
filename = "data.json".removesuffix(".json")  # Returns "data"
url = "http://example.com".removeprefix("http://")  # Returns "example.com"

# Bad - lstrip/rstrip treat input as character sets, not strings!
filename = "bison.json".rstrip(".json")  # Returns "bi" (removes all ., j, s, o, n chars!)
```

### 15. Dataclass Serialization
✓ Define explicit `to_dict()` method
✗ Avoid `asdict()` for non-trivial types

```python
from dataclasses import dataclass
from collections import Counter

# Good
@dataclass
class Data:
    counts: Counter

    def to_dict(self) -> dict:
        return {"counts": dict(self.counts)}

# Bad - asdict() may corrupt complex types
@dataclass
class Data:
    counts: Counter

data = Data(counts=Counter("hello"))
result = asdict(data)  # May not serialize correctly
```

### 16. Membership Testing
✓ Use `not in` operator
✗ Avoid `not ... in`

```python
# Good - readable
if item not in collection:
    pass

# Bad - less clear
if not item in collection:
    pass
```

### 17. Imports
✓ Use absolute imports
✗ Avoid relative imports

```python
# Good - clear and explicit
from myapp.utils import helper
from myapp.config import settings

# Bad - requires mental calculation
from .utils import helper
from ..config import settings
```

## Recommended Development Setup

When setting up a Python project, include:

- `pre-commit` for automated checks
- `mypy` with `disallow_untyped_defs=true`
- `ruff` for formatting and linting
- `pytest` with `filterwarnings=error`
- `poetry` or `uv` for dependency management
- `structlog` for structured logging

## Writing Process

1. Start with type-annotated function signatures
2. Implement with clear, descriptive variable names
3. Add early returns for validation
4. Use module-level logger for logging
5. Handle exceptions specifically
6. Add comments only for non-obvious WHY
7. Review against all guidelines above
