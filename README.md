# Code Guidelines for Python

These are coding guidelines I've successfully used with teams in the past
and can be applied generically to python projects.

- [Recommended Setup](#recommended-setup)
- [Variable Names](#variable-names)
- [Type Annotations](#type-annotations)
- [Comments](#comments)
- [Docstrings](#docstrings)
- [Regular Expressions](#regular-expressions)
- [Decimals](#decimals)
- [Global Variables](#global-variables)
- [Default Arguments](#default-arguments)
- [Default Factory in Dataclasses](#default-factory-in-dataclasses)
- [Test Structure and Organisation](#test-structure-and-organisation)
- [Properties](#properties)
- [Return early / Fail fast](#return-early-fail-fast)
- [Loggers](#loggers)
- [Logging Parameters](#logging-parameters)
- [Catching Exceptions](#catching-exceptions)
- [Removing Prefixes and Suffixes](#removing-prefixes-and-suffixes)
- [Serializing Dataclasses](#serializing-dataclasses)
- ["in" statements](#in-statements)
- [Absolute Imports](#absolute-imports)

## Recommended Setup

A good python project typically has a combination of the following tools:

- Use `pre-commit`
- use `mypy` with `disallow_untyped_defs` set to true for checking type annotations.
- Use either `black` and `isort` or `ruff format` for formatting your code and imports
- use `flake8` or `ruff check` for general linting and enforcing code standards.
- use `pytest` for running unit tests.
- configure `pytest` with `filterwarnings = error` to prevent warnings going unnoticed.
- use `poetry` or `uv` for dependency management.
- consider using `structlog` for simpler and better logging

## Variable Names

Use [snake case](https://en.wikipedia.org/wiki/Snake_case) as a convention
for variable names.

<details>
<summary>Why?</summary>
This is standard python practice :)
</details>

Do not use single letter variable names.

<details>
<summary>Why?</summary>
Variable names are an important part of code readability so we should try our best
to communicate a variable's intention as part of its name.

This is extremely hard to do if the variable name is a single letter.

There are some rare exceptions to this rule. For example `x` and `y`
are perfectly good variable names when plotting data on an graph.
It is up to the reviewer and yourself to determine these exceptions together.

Another exception to this rule is using single letter variables within comprehensions.
For example:

```python
data = [r["url"] for r in references]
```

</details>

Avoid abbreviations.

<details>
<summary>Why?</summary>
Abbreviations assume prior knowledge that a reader might not have.

Abbreviations are also easy to misunderstand as they can be interpreted
as a short hand for different things depending on the reader's background and
knowledge.

</details>

## Type Annotations

You should type annotate all function and method signatures. This should be enforced at the CI level. There is no need to type annotate individual variables unless [mypy](https://mypy.readthedocs.io/en/stable/) explicitly needs the help.

<details>
<summary>Why?</summary>

Type annotations are useful because they provide the reader with a clear
understanding of what the inputs and outputs of the function they are analysing
are.

Type annotations are a powerful tool because we can use static analysis tools
like `mypy` to check the correctness of the type annotations over time. If a
function signature no longer matches with what mypy detects, the CI should warn
you that the type annotations are no longer correct. This either means you have
mistakenly introduced a bug or you've changed the function signature and forgot
to update it. In both cases, we've prevented a regression in our code which is
great news for us.

</details>

## Comments

Avoid documenting _what_ code is doing. Prefer to document _why_ if there context.

<details>
<summary>Why?</summary>
Documenting _what_ something is doing is not always useful because the code is
already doing that by being written. That _what_ is very likely to change over
time too. Nothing in our CI checks will stop you from updating the code but
not updating the code comment.

However, explaining _why_ something was written in a certain way is useful.
The _why_ will give the reader the context they need to understand certain
decisions. It's also less likely for the _why_ to change in code over time
compared to the _what_.

There are exceptions to this rule. If code is complicated to understand,
it might be worth explaining what it is doing. But keep in mind there
is a risk that code might then change over time and the _what_ will become
out of date if not noticed. This will in turn cause the comment to make
the code harder to understand by conflicting with what the code is doing.

</details>

Use comments sparingly and only when needed.

<details>
<summary>Why?</summary>
As a writer of code, you want to make sure that important comments are read.

If too many comments exist in a codebase, it's very likely that they will be
ignored due to the noise to signal ratio (known as "noise fatigue").

For this reason only use code comments when you feel it's _absolutely necessary_
to highlight something important and that is not obvious.

</details>

## Docstrings

Only write docstrings to document unobvious behaviour at a high level.

Treat functions as a blackbox when writing docstrings. Only use docstrings to
explain inputs, outputs and possible side effects.

It's perfectly acceptable for a function not to have a docstring if writing one
brings no value to the readability of the function.

<details>
<summary>Why?</summary>
Code should ideally be self descriptive. If docstrings are written for everything,
there is a high chance what is written in the docstring will become out of date
as the code is updated over time.

If something however is very unobvious, then it might be worth documenting it
as a docstring or a code comment at a high level. These docstrings should never
get into technical details about what the function is doing internally. It should
just document the inputs, outputs and possible side effects.

</details>

Do not document type arguments in docstrings

<details>
<summary>Why?</summary>
We already type annotate all our functions and these are all verified as part of
our CI.

In comparison, type arguments in docstrings are redundant and very likely to
become out of date over time as the code changes alongside it.

</details>

## Regular Expressions

When using regular expressions, **ALWAYS** use raw strings. Raw strings are
strings preceded with an `r` at the beginning.

Example:

```python
import re

re.compile(r"\W+")
```

This prevents incorrect regular expressions from mistakenly being added

<details>
<summary>Why?</summary>

Regular expressions rely on backslashes to indicate special forms or special
characters. However the backslash character is reserved for a similar but
different purpose in normal string literals.

The only way to guarantee regular expressions work correctly is to use raw
strings as a result. This is already pretty well documented and detailed in
the [the Backslash Plague](https://docs.python.org/3/howto/regex.html#the-backslash-plague)
section of the python documentation.

</details>

## Decimals

When using `Decimal` values in python, always prefer passing a string as input
rather than a float:

```python
# Bad
from decimal import Decimal

Decimal(2.2)
```

```python
# Good
from decimal import Decimal

Decimal("2.2")
```

<details>
<summary>Why?</summary>

Decimals are used as a way to specify precise, exact decimal values.

Floating point values are more flexible than decimals, but as a result of their
internal representation, they can not represent some specific values correctly.

An easy way to see this is by printing the outputs of two different `Decimal`
values.

```python
from decimal import Decimal

print(Decimal("2.2"))
# Decimal('2.2')

print(Decimal(2.2))
# Decimal('2.20000000000000017763568394002504646778106689453125')
```

As you can see above, using a float as a constant to intialise our Decimal
resulted in an approximately equal value to 2.2 being stored - but not the
exact value that we were looking for.

This is often suprising to many coders, especially once comparison checks
start failing. For example, the equality check below would result in a `False`

```python
Decimal(2.2) == Decimal("2.2")
# False
```

</details>

## Global Variables

Avoid interacting with global variables like lists, dicts, sets or any other
mutable objects directly.
Write a function that returns the same value instead.

```python
# Bad

CWE_MAP = {"CWE-80": ["foo", "bar", "baz"]}
```

```python
# Good


def get_cwe_map() -> Dict[List[str]]:
    return {"CWE-80": ["foo", "bar", "baz"]}
```

<details>
<summary>Why?</summary>

Using the values of mutable global variables directly will
mean the global variable will very likely be unexpectedly updated. This will
cause other pieces of code which also read the global variable to get incorrect results.

For example, the code below is likely to cause problems:

```python
CWE_MAP = {"CWE-80": ["foo", "bar", "baz"]}


def get_cwe_values(cwe: str) -> List[str]:
    # !!! Notice that we do _not_ copy the data here
    return CWE_MAP[cwe]
```

At first glance this function might seem perfectly fine. However there is a real
danger of mistakenly updating the global variable due to not using a copy.

```python
values = get_cwe_values("CWE-80")

# Append needed data
values.append("bang")
```

Unexpectedly, the global variable has also been updated:

```python
print(CWE_MAP)
# {'CWE-80': ['foo', 'bar', 'baz', 'bang']}
```

Using a function to return the data instead solves this issue:

```python
def get_cwe_map() -> Dict[List[str]]:
    return {"CWE-80": ["foo", "bar", "baz"]}


def get_cwe_values(cwe: str) -> List[str]:
    return get_cwe_map()[cwe]
```

```python
values = get_cwe_values("CWE-80")

# Append needed data
values.append("bang")

print(get_cwe_map())  # Nothing has changed
# {'CWE-80': ['foo', 'bar', 'baz']}
```

</details>

## Default Arguments

When using default arguments in functions, never directly assign lists, sets,
dictionaries or any other mutable objects.

Instead, specify the default value as `None` and initialise at the beginning of
the function:

```python
# Bad
def do_something(values: list = []) -> None:
    pass
```

```python
# Good
def do_something(values: Optional[list] = None) -> None:
    if values is None:
        values = []

    # continue the function as usual
```

<details>
<summary>Why?</summary>

Values assigned to default arguments are essentially global values. This means
that in the likely scenario you'll append, update or remove from the object,
that new state will then be used in the next function call.

Take this example and see how calling it multiple times might cause issues:

```python
def extend_list(values: list = []) -> list:
    values.append(10)
    return values
```

Calling `extend_list()` the first time will correctly return `[10]`. However
calling `extend_list()` a second time will return `[10, 10]` instead. Calling
it a third time will return `[10, 10, 10]` and so on and so forth.

Using this modified version will correctly return `[10]` every single time
instead:

```python
def extend_list(values: Optional[list] = None) -> list:
    if values is None:
        values = []

    values.append(10)
    return values
```

While the previous one is easier to read, it's unfortunately a python side
effect to be aware of and should be avoided at all costs to prevent undesired
and unexpected behaviour.

</details>

## Default Factory in Dataclasses

When assigning default values to Dataclasses, always use `field(default_factory=)`
when assigning lists, dicts, sets or any mutable objects.

Use lambdas, if you want to assign a non-empty initial value.

```python
# Good
from dataclasses import dataclass, field


@dataclass
class Foo:
    description: str = "strings and numbers are ok to assign directly"
    configuration: dict = field(default_factory=dict)
    values: list[int] = field(default_dactory=lambda: [1, 2, 3])
    producer: Producer = field(default_factory=lambda: Producer("defaultarg"))
```

```python
# Bad
from dataclasses import dataclass, field


@dataclass
class Foo:
    description: str = "strings and numbers are ok to assign directly"
    configuration: dict = {}
    values: list[int] = [1, 2, 3]
    producer: Producer = Producer("defaultarg")
```

<details>
<summary>Why?</summary>

Values directly assigned as defaults are global variables. This means that multiple
instances of the same dataclass will share the same instance of those defaults and
unexpectedly update each other when changed.

For example, take the dataclass below:

```python
from dataclasses import dataclass


@dataclass
class Foo:
    values: dict = {}
```

See what happens when we initialise two instances of `Foo()` and change its values:

```python
foo1 = Foo()
foo2 = Foo()
foo1.values["hello"] = "world"

print(foo1.values)  # outputs {"hello": "world"}
print(foo2.values)  # also outputs {"hello": "world"}
```

NOTE: Newer versions of python (3.9+) will automatically raise errors for dicts,
lists and sets. However, they will not raise an error if you assign any other Object.

</details>

## Test structure and organisation

Use the following test structure:

- Tests written for functions, classes or methods in `foo/bar/filename.py`
  belong in `tests/foo/bar/test_filename.py`.

- Tests for functions with the name `do_something` should be grouped under
  `class TestDoSomething` within the test file. Specific test cases should then
  be methods within that class.

- Tests for classes with the name `SomeClass` should be grouped under
  `class TestSomeClass` within the test file.

`foo/bar/filename.py`

```python
def do_something(a: int, b: int) -> int:
    pass


class SomeClass:
    def __init__(self) -> None:
        pass

    def get_values() -> list:
        pass

    def run() -> None:
        pass
```

`tests/foo/bar/test_filename.py`

```python
class TestDoSomething:
    def test_correct_values(self):
        pass

    def test_zero(self):
        pass


class TestSomeClass:
    def test_get_values(self):
        pass

    def test_run_succeeds(self):
        pass

    def test_run_fails(self):
        pass
```

<details>
<summary>Why?</summary>

The benefit of this organisational approach is that it makes it:

- A lot more obvious where to find and locate tests.
- Easy to identify missing tests
- Easy to detect what is running and what failed when running `pytest`

Quite importantly, this structure also allows you to be quite flexible about
what tests you want to run:

- Run all tests related to filename.py: `pytest tests/test_filename.py`
- Run all tests related to `SomeClass`: `pytest tests/test_filename.py::TestSomeClass`
- Run a specific test case: `pytest tests/test_filename.py::TestSomeClass::test_some_specific_case`

</details>

## Properties

Avoid using properties unless strictly necessary. Stick to methods instead as
it is more clear they perform non-trivial operations.

```python
# Bad
class MyClass:
    @property
    def value(self):
        # do stuff here
        return do_something()
```

```python
# Good
class MyClass:
    def get_value(self):
        # do stuff here
        return do_something()
```

<details>
<summary>Why?</summary>

Properties give the impression that a method which performs a non-trivial
memory access is a "free" operation in terms of accessing this value as a
variable. When in actual fact the method might often be performing calculations
(e.g. arithmetic) or expensive operations (e.g. database queries) under the hood.

For example if we had the following property defined on a django model:
model:

```python
class FooBar(models.Model):
    release = models.ForeignKey("Release")
    title = models.TextField()

    @property
    def is_released(self):
        if self.release:
            return self.release.active
        else:
            return False
```

And we had some code somewhere in our codebase that shows the following lines:

```python
foobar = FooBar.objects.get(pk=1)
print(foobar.title)
print(foobar.is_released)
```

As someone who has never seen/used `is_released` would you know that the
second line calling `title` is a free operation (no IO/DB operation, just accessing a variable)
while the third line is an expensive operation? (Performs an IO/DB access).

**Properties give the impression we are performing a quick RAM access when its likely
not the case.**

Properties convolute this understanding for minor syntactic sugaring.

On the other hand using a function notation is more clear that it might be more
expensive because it doesn't give the impression we are just accessing a variable.

```python
foobar = FooBar.objects.get(pk=1)
print(foobar.title)
print(foobar.is_released())
```

Another way to think of it is, If you were debugging some code line by line,
you wouldn't think to "go to definition" for `is_released` because you would
think its just a variable access. On the other hand if you saw `is_proprietary()`
you would think it might be smart to see what is happening inside this function.

</details>


## Return early / Fail fast

Avoid nesting the happy path inside an if branch.
Validate the failure conditions and return or fail early.

```python
# Bad
def foo(bar: Any) -> Any:
    if bar:
        if do_something_with(bar):
            return do_something_else_with(bar)
        else:
            return None
    else:
        return None
```

Recommended:

```python
# Good
def foo(bar: Any) -> Any:
    if not bar:
        return None
    if not do_something_with(bar):
        return None
    return do_something_else_with(bar)
```

<details>
<summary>Why?</summary>
Code is less nested, easier to follow and to focus on the "happy" (or "main") path.
</details>

## Loggers

Define a logger for each module with the module's name. Do not use the root logger.

```python
# Bad
import logging


def do_something():
    logging.info("I just did something")
```

```python
# Good
import logging


logger = logging.getLogger(__name__)


def do_something():
    logger.info("I just did something")
```

If you are using `structlog`:

```python
# Good
import structlog

logger = structlog.get_logger(__name__)

def do_something():
    logger.info("I just did something")
```

<details>
<summary>Why?</summary>
Using a separate logger per module and using the module name allows us to filter,
configure and analyze our logs more easily.

Some examples of what can be done using this pattern:

- filter logs to a specific module path or subpath in datadog.
  For example filter all loggers starting with `foo.bar`
- configure certain loggers and their children.

For example send anything under `core.models` to a specific handler

</details>

## Logging Parameters

Use logging parameters rather than format strings when writing log messages:

```python
# Bad
import logging

logger = logging.getLogger(__name__)
logger.error(f"Something happened: {message}")
```

```python
# Good
import logging

logger = logging.getLogger(__name__)
logger.error("Something happened: %s", message)
```

If you are using `structlog`:

```python
# Good
import structlog

logger = logging.get_logger(__name__)
logger.error("Something happened", message=message)
```

<details>
<summary>Why?</summary>
Downstream consumers of logs can take advantage of logging parameters.
For example:

- Sentry can group the same error logs together despite different parameters
- Datadog can index the parameters for searching, grouping and filtering

If format strings were used instead, downstream integrations might treat
the same logger call with different parameter values differently.
In the case of Sentry for example this could result in spammy event
notifications as it cannot understand they come from the same log message.

</details>

## Catching Exceptions

In general, never use `except:` on its own. If you want to catch all
exceptions, use `except Exception:`:

```python
# Bad
try:
    my_failing_function()
except:
    logger.exception("Oh no!")
```

```python
# Good
try:
    my_failing_function()
except Exception:
    logger.exception("Oh no!")
```

<details>
<summary>Why?</summary>
Some exceptions should be allowed to blow up without being caught. Some typical
examples of exceptions we wouldn't want to catch and allow to bubble up the stack are:

- KeyboardInterrupt: in order to allow the user to interrupt code with Ctrl+C
- OutOfMemory: in order for the program to shut down gracefully if it gets killed by OOM

These exceptions inherit from `Error` rather than `Exception`. Standard exceptions which
should be caught typically are a subclass of `Exception`. This means that using
`except Exception:` allows us to catch all of the exceptions we would probably care about
while allowing other exceptions like the above to bubble up the stack and let the program
exit in most cases.

</details>

## Removing Prefixes and Suffixes

Do **not** use the `lstrip` and `rstrip` python functions to remove prefixes
and suffixes. You will get incorrect results when you least expect
them if you use them for this purpose.

Use `removeprefix`/`removesuffix` instead.

<details>
<summary>Why?</summary>

`rstrip` and `lstrip` functions actually use the input string
as a collection of characters to remove as long they exist at the
right/left side of the string value.

Some examples can help make this more clear:

```python
assert "bison.json".rstrip(".json") == "bi"
```

Notice how the "son" in "bison" was removed because those
characters still existed in the input ".json"

Similar situations can be seen for `lstrip` below:

```python
assert "hellooliver".lstrip("hello") == "iver"
assert "http://hp.com".lstrip("http://") == ".com"
```

</details>

## Serializing Dataclasses

Feel free to use dataclasses, but do **not** use the in-built `asdict` helper
function when trying to serialize the data.

Define your own `to_dict` method instead

<details>
<summary>Why?</summary>

`asdict` does not work reliably with sub types which are not "basic" types like `int`,
`str`, `dict`, `list` etc...

The author of the `asdict` function himself actually mentions that the function
should not be used and in hindsight should never have been created:

> The asdict API was a mistake, because it's not possible for it to know how to create
> all possible sub-objects. I can't decide what to do about it. I might just
> "abandon it in place", by documenting its problems and suggesting it not be used.

https://bugs.python.org/issue39929

Take an example of where `asdict` fails horribly:

```python
from collections import Counter
from dataclasses import dataclass, asdict


@dataclass
class Foo:
    values: Counter


foo = Foo(values=Counter("hello"))
print(asdict(foo))
```

This prints the output:

```
{'values': Counter({('h', 1): 1, ('e', 1): 1, ('l', 2): 1, ('o', 1): 1})}
```

In particular, notice that `Counter` has not been serialized to a dictionary
as expected and that the data has also been corrupted (compare it with the
original Counter below).

```python
Counter("hello")
# Counter({'h': 1, 'e': 1, 'l': 2, 'o': 1})
```

</details>

## "in" statements

Prefer writing

```python
# Good
if "something" not in data:
    ...
```

Rather than:

```python
# Bad
if not "something" in data:
    ...
```

<details>
<summary>Why?</summary>
This helps readability in a few ways:

- this is a fairly standard to follow in python projects so the consistency
  alone will help with readability.
- The code feels closer to plain text English.
- Keeping both `not` and `in` in the same location makes it easier to understand and visually grep what the actual operation being performed is.

</details>

## Absolute Imports

Prefer to use absolute imports in general, rather than relative imports.

```python
# Good
from foobar.baz import bang
```

```python
# Bad
from .baz import bang
```

<details>
<summary>Why?</summary>
While there are some cases where relative imports might help readability, in general
making use of relative imports makes it harder for the reader to understand where exactly
an import is coming from.

It either requires the reader to mentally calculate the location of the module themselves
or use a tool like `go-to-definition` in their IDE to figure it out.
In either case, that is wasted time and un-needed overhead.

Absolute imports make it clear exactly where and what the module being imported is.
</details>
