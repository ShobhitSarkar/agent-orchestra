# Simplification Patterns Reference

A checklist of known anti-patterns for the simplifier agent. Organized by category from most critical to least.

## Security

- **Hardcoded credentials**: Replace password/key/connection-string literals with `os.environ.get(...)` or config references. Never leave them as "defaults".
- **eval/exec on untrusted data**: Replace `eval()` with AST-safe alternatives. Remove `eval(repr(x))` round-trips entirely. Replace `exec()` with explicit logic.
- **Shell injection via os.system**: Replace `os.system(f"...")` with `subprocess.run([...])` using argument lists. Use Python-native equivalents (`shutil`) where possible.
- **SQL injection**: Replace `f"SELECT ... WHERE x = '{input}'"` with parameterized queries `(template, params)`.
- **Self-modifying code**: Delete any function that reads/writes its own source file (`__file__`).
- **Dangerous deployment commands**: Remove functions running `rm -rf`, `git push --force`, or `ssh` against production unless properly safeguarded.
- **Intentional resource exhaustion**: Delete functions whose only purpose is to leak memory or spin forever.

## Correctness

- **Broken dunder methods**: `__eq__` must be deterministic (no `random`). `__hash__` must be consistent with `__eq__`. `__bool__` must not call `not self` (infinite recursion). `__len__` must return a non-negative int. `__repr__` must return a string. `__getattr__` returning `self` creates infinite loops. `__del__` must not create new instances. When in doubt, delete the dunder.
- **Recursion without base cases**: Every recursive function needs a terminating condition that makes progress toward it.
- **Circular self-references (`self.x = self`)**: Remove — prevents garbage collection, serves no purpose.
- **Thread-unsafe shared state**: Class-level mutable variables shared across threads need `threading.Lock`. Pattern: `_lock` as instance attr, `with self._lock:` around mutations.
- **Bare `except:`**: Always catch specific exceptions. `except Exception:` at minimum — bare `except:` swallows `KeyboardInterrupt` and `SystemExit`.
- **Infinite retry on error**: An event handler that fails should not retry recursively. Let exceptions propagate, log them, or use bounded retry.
- **Event systems overwriting callbacks**: Use `dict[str, list[callable]]` with `.append()` to support multiple subscribers per event.
- **Recursive `input()` for type resolution**: Never prompt the user at runtime. Raise `TypeError` or return a sensible default.

## Redundancy

- **Constants that lie** (`TRUE = False`, `PI = 3`): Delete or replace with stdlib values (`math.pi`). A wrong constant is worse than none.
- **Lying type hints** (`def add(a: str, b: str) -> list` for numeric addition): Remove incorrect hints entirely.
- **Lying comments** (`i = 0  # increment i by 2`): Delete — a wrong comment is worse than none.
- **Duplicate accessor methods**: Consolidate `getValue()`, `SetValue()`, `set_the_value_of_...()` into one `get_value()`/`set_value()`.
- **Duplicate function implementations**: If a method and standalone function do the same thing, keep one. Prefer standalone unless instance state is needed.
- **Temp variable chains** (`temp = x; temp2 = temp; result = temp2`): Replace with direct assignment. Keep only meaningful transformation steps.
- **Module-level throwaway statements** (`i = 0; i += 1` at module scope with no consumer): Delete.
- **Global mutable module state**: Delete unused `GLOBAL_STATE = {}` / `database = []`. If needed, prefix with `_`.
- **Module-level `global` declarations**: `global x` at module scope is a no-op — remove.
- **Wildcard imports**: Replace `from x import *` with explicit imports of what's actually used.
- **Unused imports**: Audit and delete after fixing wildcards.

## Logic

- **Hardcoded lookup tables for formulas**: `is_even()` as 11 `if` statements → `n % 2 == 0`. Same for `factorial`, `is_prime`, `is_leap_year` with case-by-case logic.
- **Custom sort when stdlib exists**: Replace hand-rolled sorting with `sorted()` / `list.sort()`.
- **Over-engineered truthiness**: 50+ string comparisons for truthy/falsy → `bool(value)` or a small set `{"yes", "true", "1"}`.
- **Multiple negation**: Even `not`s = identity. Odd = single `not x`.
- **Pyramid of doom**: Collapse constant-true `if True:` nesting. For real conditions, use early returns/guard clauses.

## Cleanup

- **Identity functions** (`def f(x): return x`): Delete if no callers rely on it as a hook point.
- **Stub functions that only raise**: `def check_admin(): raise NotImplementedError` — implement or delete.
- **Trivial wrappers** (`def is_truthy(x): return bool(x)`): Delete. Callers should use the builtin.
- **Magic default strings**: `return "default"` → use `None` or accept default as parameter.
- **Generic names**: Rename `data`, `do_thing`, `process` to describe their actual purpose.
- **Inconsistent parameter naming**: `add(a, b)` and `multiply(x, y)` doing analogous work should use the same names.
- **`callable` vs `Callable`**: Use `collections.abc.Callable` for type annotations, not the builtin function.
