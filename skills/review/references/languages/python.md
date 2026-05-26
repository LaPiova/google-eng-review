# Python — review profile

> Distilled from the Google Python Style Guide (https://google.github.io/styleguide/pyguide.html), CC BY 3.0. Review-oriented summary, not a reproduction. Tier 1 (curated).

## Naming

- **Modules & packages**: `lower_with_under` only. No dashes in file names; must end in `.py`.
- **Classes**: `CapWords` (public) or `_CapWords` (internal). Exception classes must end in `Error` (e.g., `ParseError`, not `FooParseError` — avoid redundant prefix).
- **Functions & methods**: `lower_with_under()`. Internal/protected: `_lower_with_under()`. Double underscore (`__name`) is discouraged — prefer single leading underscore.
- **Constants**: `CAPS_WITH_UNDER` (global) or `_CAPS_WITH_UNDER` (internal). Only use CAPS at module scope, not local variables.
- **Variables & parameters**: `lower_with_under`. No type information in variable names (`id_to_name_dict` anti-pattern).
- **Type variables**: Descriptive if externally visible or constrained (`AddableType = TypeVar(...)`); short `_T` / `_P` only when unconstrained and private.
- **Single-character names**: Allowed only for loop counters (`i`, `j`, `k`), exception aliases (`e`), file handles (`f`), and unconstrained private type vars.
- **No double-leading-and-trailing dunder names** (`__foo__`) unless mandated by Python itself.

## Formatting & idioms a reviewer should flag

- **Line length**: Hard max 80 characters. Exceptions: long URLs in comments, long import statements, pathnames.
- **Indentation**: 4 spaces per level — no tabs. Continuation lines align with the opening delimiter or use 4-space hanging indent.
- **No backslash continuation**: Use implicit joining inside parentheses/brackets/braces instead.
- **Blank lines**: Two blank lines between top-level definitions (functions, classes). One blank line between methods and after the class docstring. No blank line immediately after a `def`.
- **Parentheses**: Avoid redundant parentheses in `return` and conditional statements. Single-element tuples need explicit parens: `(foo,)`.
- **Trailing commas**: Recommended when the closing bracket is on its own line or for single-element tuples; this hints Black/Pyink to keep one item per line.
- **Whitespace around `=`**: No spaces for keyword arguments/defaults; spaces required when a type annotation is present on the same `=`.
- **String quoting**: Single or double quotes — pick one and be consistent within a file. Triple-double-quotes (`"""`) for all docstrings and multi-line strings; never `'''` for docstrings.
- **Comprehensions**: Allowed only for simple, single-clause cases. Flag any comprehension with multiple `for` clauses or multiple filter expressions — rewrite as a loop.
- **Lambdas**: Only acceptable for true one-liners. Flag lambdas exceeding ~60–80 characters or spanning multiple lines; replace with a named function.
- **Conditional expressions**: Each of (true-expr, condition, else-expr) must fit on one line. Flag multi-line ternaries.
- **Properties**: Allowed only when computation is trivial and access semantics are expected. Flag properties with complex logic or observable side effects.
- **No mutable global state**: Module-level constants are fine (`CAPS`). Flag any mutable global variable.
- **staticmethod**: Never use unless integrating with an external API that requires it.

## Error handling

- **No bare `except:`**: Flag always — catches `KeyboardInterrupt`, `SystemExit`, misspellings. Minimum: `except Exception:` with a re-raise or documented suppression.
- **No catch-all swallowing**: Catching `Exception` or `BaseException` is allowed only when re-raising or at an explicit isolation boundary with logging.
- **Minimize try scope**: The `try` block should contain only the lines that can actually raise. Flag large try blocks that mix setup, work, and cleanup.
- **Use `finally` for cleanup**: Resource cleanup goes in `finally`, or better, use a `with` statement. Flag explicit `file.close()` calls outside a `with`.
- **Custom exceptions**: Must subclass an existing exception class and end in `Error`. Flag custom exception classes that don't follow this convention.
- **No `assert` for runtime validation**: `assert` may be disabled (`-O`). Flag asserts used to validate arguments or enforce preconditions in production code; use `ValueError`/`RuntimeError` instead.
- **Exception chaining**: Preserve cause using `raise X from Y` when re-raising after a caught exception.

## Documentation / doc-comments

- **All public modules need a module docstring**: First statement in the file; one-line summary ending with `.`/`?`/`!`, blank line, then extended description.
- **All public classes need a class docstring**: Below the `class` line. Include an `Attributes:` section for public attributes (not properties).
- **All public functions/methods**: Docstring required unless body is trivially obvious. Required sections when applicable:
  - `Args:` — one entry per parameter; include type if no annotation exists.
  - `Returns:` / `Yields:` — omit only if function returns `None` or docstring opens with "Returns"/"Yields".
  - `Raises:` — list exceptions the caller should handle; omit misuse-triggered exceptions.
- **`@override` methods**: May omit docstring unless behavior materially diverges from the base.
- **Inline comments**: At least 2 spaces from code, `# ` prefix. Never restate what the code does — explain *why* or document non-obvious behavior.
- **TODO format**: `# TODO: <bug-link> - <explanation>`. Do not name individuals. Include date/event for time-bound TODOs.
- **Grammar & punctuation**: Comments are prose — capitalize first word, end with period.

## Testing conventions

- **Use `assert` freely in pytest tests**: The guide explicitly endorses pytest-style assertions in test code.
- **Access to protected members**: Unit tests may access `_protected` constants from the module under test.
- **Decorator tests**: Write unit tests for custom decorators.
- **Test module docstrings**: Optional; include only when providing context about test execution or setup.
- **Test method naming**: Follow `test_<method_under_test>_<state>` convention; underscores may separate logical components.

## Common anti-patterns

- **Mutable default arguments**: `def f(lst=[])` — flag unconditionally. Use `None` default + initialization inside the function.
- **Wildcard imports**: `from x import *` — flag always. Pollutes namespace and makes it impossible to audit what names are in scope.
- **Relative imports**: `from . import foo` — flag. Always use full package name.
- **`import x, y` on one line**: Flag (except `typing`/`collections.abc` multi-import lines).
- **`__del__`**: Flag use for cleanup logic. Relies on GC timing; use context managers instead.
- **Metaclasses**: Flag custom metaclasses unless wrapping `abc.ABCMeta` or `dataclasses`.
- **Dynamic features** (`getattr` by computed string, `setattr`, runtime `__class__` modification, import hacks): Flag any use without clear justification.
- **Relying on built-in atomicity**: Flag code that depends on `dict` or `list` operations being atomic in threaded code; use `queue.Queue` or explicit locks.
- **No type info in names**: Flag `list_of_users`, `user_dict`, `name_str` — use plain `users`, `user_by_id`, `name`.

## Expected tooling

- **pylint**: Primary linter. Flag suppression comments (`# pylint: disable=...`) that lack a justification comment.
- **pytype**: Static type checker; should be enabled in CI for annotated public APIs.
- **Black or Pyink**: Auto-formatter; if the project uses one, formatting deviations are a hard flag.
- **Type annotations**: Strongly expected on all public APIs. Flag unannotated public functions in any new code.
- **`from __future__ import annotations`**: Use for forward references rather than string literals.

## Highest-signal review checks (top ~8, prioritized)

1. **Mutable default argument** — `def f(items=[])` is a latent bug that corrupts shared state across calls. Always flag.
2. **Bare or over-broad `except`** — `except:` or `except Exception:` without re-raise hides bugs silently. Always flag.
3. **Missing type annotations on public APIs** — defeats static analysis for all callers; flag any unannotated public function signature.
4. **Wildcard / relative import** — `from x import *` or `from . import y` pollutes namespace or breaks refactoring. Flag both.
5. **Missing or malformed docstring** — public functions/classes/modules without a docstring, or a docstring missing required `Args:`/`Returns:`/`Raises:` sections.
6. **Assert used as runtime validation** — asserts are stripped with `-O`; misuse masks bugs in production.
7. **Comprehension complexity** — multiple `for` clauses or filter expressions in one comprehension kills readability; flag for extraction into a loop.
8. **Mutable global state** — any module-level mutable variable (not `CAPS_CONST`) is a concurrency and testability hazard.
