---
name: python-review
description: Review Python code against established coding guidelines and best practices. Use when reviewing code quality, checking for common pitfalls, or ensuring consistency with Python standards.
allowed-tools: Read, Grep, Glob
---

# Python Code Review

Review the provided Python code against these coding guidelines and identify violations or improvements.

## Review Process

1. Analyze the code for guideline violations
2. Prioritize issues: CRITICAL → IMPORTANT → MINOR
3. Provide specific code examples for each suggestion
4. Explain the rationale and potential impact

## Python Coding Guidelines to Check

### 1. Variable Names
- ✓ Use snake_case consistently
- ✗ Avoid single letters (except in comprehensions)
- ✗ Eliminate abbreviations (e.g., `usr` → `user`, `calc` → `calculate`)

### 2. Type Annotations
- ✓ Annotate ALL function signatures with types and return values
- ✓ Use built-in types (`list`, `dict`, `set`) instead of `typing.List`, `typing.Dict`
- ✓ Configure mypy with `disallow_untyped_defs=true`

**Bad:**
```python
def process_data(items):
    pass
```

**Good:**
```python
def process_data(items: list[str]) -> dict[str, int]:
    pass
```

### 3. Comments
- ✓ Document WHY, not WHAT
- ✓ Use sparingly to reduce noise
- ✗ Avoid obvious comments that restate code

**Bad:**
```python
# Loop through items and append to result
for item in items:
    result.append(item * 2)
```

**Good:**
```python
# We double values here due to API v2 expecting doubled integers
for item in items:
    result.append(item * 2)
```

### 4. Docstrings
- ✓ Only for non-obvious behavior
- ✗ Don't document types (use type annotations)
- ✗ Avoid redundant docstrings for simple functions

### 5. Regular Expressions
- ✓ ALWAYS use raw strings: `r"\W+"` not `"\W+"`

**Bad:**
```python
pattern = re.compile("\W+")
```

**Good:**
```python
pattern = re.compile(r"\W+")
```

### 6. Decimals
- ✓ Pass strings to Decimal, NEVER floats
- ✗ `Decimal(2.2)` causes precision issues

**Bad:**
```python
value = Decimal(2.2)  # Decimal('2.20000000000000017763...')
```

**Good:**
```python
value = Decimal("2.2")  # Decimal('2.2')
```

### 7. Global Variables
- ✗ Avoid direct access to mutable globals
- ✓ Use factory functions or return copies

**Bad:**
```python
CWE_MAP = {"CWE-80": ["foo", "bar"]}

def get_values(cwe: str) -> list:
    return CWE_MAP[cwe]  # Returns reference, not copy!
```

**Good:**
```python
def get_cwe_map() -> dict:
    return {"CWE-80": ["foo", "bar"]}

def get_values(cwe: str) -> list:
    return get_cwe_map()[cwe]
```

### 8. Default Arguments
- ✗ NEVER use mutable defaults (`[]`, `{}`)
- ✓ Use `None` and initialize inside function

**Bad:**
```python
def extend_list(values: list = []) -> list:
    values.append(10)
    return values
```

**Good:**
```python
def extend_list(values: list | None = None) -> list:
    if values is None:
        values = []
    values.append(10)
    return values
```

### 9. Dataclass Defaults
- ✓ Use `field(default_factory=)` for mutable types

**Bad:**
```python
@dataclass
class Config:
    settings: dict = {}  # Shared across instances!
```

**Good:**
```python
@dataclass
class Config:
    settings: dict = field(default_factory=dict)
    items: list = field(default_factory=lambda: [1, 2, 3])
```

### 10. Properties vs Methods
- ✓ Prefer explicit methods over properties
- ✗ Properties hide computational cost

**Bad:**
```python
@property
def user_count(self) -> int:
    return db.count_users()  # Expensive operation hidden
```

**Good:**
```python
def get_user_count(self) -> int:
    return db.count_users()  # Clear this may be expensive
```

### 11. Early Returns / Fail Fast
- ✓ Validate failures upfront
- ✓ Reduce nesting with early returns

**Bad:**
```python
def process(data: dict) -> str:
    if data:
        if validate(data):
            if transform(data):
                return result
```

**Good:**
```python
def process(data: dict) -> str | None:
    if not data:
        return None
    if not validate(data):
        return None
    if not transform(data):
        return None
    return result
```

### 12. Loggers
- ✓ Use module-level logger: `logger = logging.getLogger(__name__)`
- ✗ Never use root logger: `logging.info()`

**Bad:**
```python
def do_work():
    logging.info("Working...")
```

**Good:**
```python
logger = logging.getLogger(__name__)

def do_work():
    logger.info("Working...")
```

### 13. Logging Parameters
- ✓ Use parameter passing, not f-strings
- Enables downstream tools to group/filter logs

**Bad:**
```python
logger.error(f"Error: {msg}")
```

**Good:**
```python
logger.error("Error: %s", msg)
# or with structlog:
logger.error("Error occurred", message=msg)
```

### 14. Exception Handling
- ✗ NEVER use bare `except:`
- ✓ Use `except Exception:` to allow KeyboardInterrupt propagation

**Bad:**
```python
try:
    risky_operation()
except:  # Catches too much!
    pass
```

**Good:**
```python
try:
    risky_operation()
except Exception:
    logger.exception("Operation failed")
```

### 15. String Prefix/Suffix Removal
- ✓ Use `removeprefix()` and `removesuffix()`
- ✗ NEVER use `lstrip()`/`rstrip()` for this (they treat input as character sets!)

**Bad:**
```python
"bison.json".rstrip(".json")  # Returns "bi", not "bison"!
```

**Good:**
```python
"bison.json".removesuffix(".json")  # Returns "bison"
```

### 16. Dataclass Serialization
- ✓ Define explicit `to_dict()` method
- ✗ Avoid `asdict()` for non-trivial types

**Bad:**
```python
data = asdict(obj)  # May corrupt complex types
```

**Good:**
```python
@dataclass
class Data:
    counts: Counter

    def to_dict(self) -> dict:
        return {"counts": dict(self.counts)}
```

### 17. Membership Testing
- ✓ Use `not in` operator
- ✗ Avoid `not ... in`

**Bad:**
```python
if not item in collection:
    pass
```

**Good:**
```python
if item not in collection:
    pass
```

### 18. Imports
- ✓ Use absolute imports
- ✗ Avoid relative imports

**Bad:**
```python
from .utils import helper
from ..config import settings
```

**Good:**
```python
from myapp.utils import helper
from myapp.config import settings
```

## Output Format

For each issue found, provide:

**[PRIORITY] Guideline: [Name]**
- **Location:** `file.py:line_number`
- **Issue:** [Description]
- **Current Code:**
```python
# problematic code
```
- **Suggested Fix:**
```python
# improved code
```
- **Rationale:** [Why this matters]

End with a summary of findings and overall code quality assessment.
