# C++ — review profile

> Distilled from the Google C++ Style Guide (https://google.github.io/styleguide/cppguide.html), CC BY 3.0. Review-oriented summary, not a reproduction. Tier 1 (curated).

## Naming

- **File names:** `snake_case.cc` / `snake_case.h`; test files end in `_test.cc`. No `.cpp`, no `.C`, no `.cxx` at Google.
- **Types (classes, structs, enums, type aliases):** `PascalCase` — `MyClass`, `MyEnum`, `using MyAlias = int;`. No underscores.
- **Functions and methods:** `PascalCase` — `MyFunction()`, `GetValue()`, `SetValue()`. Accessors may drop the verb for trivial getters (`int count() const`).
- **Variables (local and parameters):** `snake_case` — `int job_count`.
- **Class data members:** `snake_case_` (trailing underscore) — `int job_count_`. Struct data members: `snake_case` (no underscore, structs are passive data).
- **Constants and `constexpr` values:** `kPascalCase` — `const int kMaxRetries = 3;`. Not `MAX_RETRIES`, not `kmax_retries`.
- **Macros:** `UPPER_CASE_WITH_UNDERSCORES`. Macros are strongly discouraged; prefer `constexpr`, `inline`, or templates.
- **Namespaces:** all lowercase, no underscores — `namespace myproject`. Internal-only namespaces: `namespace internal`.
- **Enumerators:** `kPascalCase`, same as constants — `enum class Color { kRed, kGreen, kBlue };`. (Old-style `ALL_CAPS` enumerators are a flag.)
- **Template parameters:** `PascalCase` — `template <typename ElementType>`.

## Formatting & idioms a reviewer should flag

- **clang-format is mandatory.** All Google C++ code is formatted by `clang-format` (Google style). Any diff that isn't clang-format-clean should be rejected. Formatting debates are non-issues — the tool decides.
- **Line length: 80 characters.** (Google's canonical limit.) Flag lines that significantly exceed this; use clang-format rather than manual wrapping.
- **Indentation: 2 spaces, no tabs.** Clang-format enforces this, but watch for patches that introduce tab characters.
- **Braces: same-line for all constructs.** Opening brace on the same line as the statement (`if (cond) {`, `class Foo {`). Clang-format enforces this.
- **Header include order** (must be separated by blank lines and alphabetized within each group):
  1. Related header (`foo.h` in `foo.cc`)
  2. C system headers (`<unistd.h>`)
  3. C++ standard library headers (`<vector>`)
  4. Third-party library headers
  5. Project headers (`"myproject/foo.h"`)
- **Project headers use `"quoted"` paths; system headers use `<angle>` brackets.** No `.` or `..` in paths.
- **No `using namespace foo;`** in headers or at namespace scope in `.cc` files. Pollutes namespace for every TU that includes the file. `using`-declarations (`using std::string;`) are acceptable only in `.cc` files.
- **No inline namespaces.** Google prohibits them.
- **Unnamed namespace / `static` for TU-local definitions only in `.cc` files**, never in headers.
- **Declare variables in the narrowest scope possible; initialize at declaration.** Flag `int x; x = f();` — should be `int x = f();` or `auto x = f();`.
- **Objects with expensive constructors/destructors in tight loops** may be declared outside the loop — flag if the opposite pattern causes measurable overhead.
- **No `std::endl`** — prefer `'\n'`; `std::endl` flushes the buffer unnecessarily.
- **Trailing return type (`auto Foo() -> T`)** only when regular syntax is impractical (complex templates, lambdas). Flag casual use where `T Foo()` reads fine.

## Error handling

- **Google C++ code does NOT use exceptions.** No `throw`, no `try`/`catch`, no `std::exception_ptr`, no `std::nested_exception`. This is Google's most distinctive stance. Flag any exception code in a new PR.
- **Constructor failures:** constructors cannot signal errors without exceptions. Use factory functions or a two-phase `Init()` method that returns a status, or use `absl::Status`/`absl::StatusOr<T>`.
- **Error returns:** prefer `absl::Status` / `absl::StatusOr<T>` over raw boolean or `int` error codes. Flag ad-hoc sentinel values.
- **`noexcept`:** in environments where exceptions are disabled (most Google code), prefer unconditional `noexcept` on move constructors and other performance-sensitive functions. Flag non-`noexcept` move constructors — they prevent `std::vector` from using the move path.
- **No RTTI in production code.** `dynamic_cast` and `typeid` are forbidden except in unit tests and rare safe-downcast scenarios. Use `absl::down_cast` when a downcast is provably safe; use virtual dispatch or the Visitor pattern otherwise.
- **Assertions for invariants.** Use `DCHECK`/`CHECK` (or `assert`) to enforce preconditions rather than silently returning bad values.

## Documentation / doc-comments

- **Every non-obvious `.h` declaration needs a comment.** Classes get a class-level comment explaining purpose and usage; functions get a comment on the declaration (not the definition) describing what they do, not how.
- **File-level comment** at the top of each file: describe overall purpose; no copyright boilerplate is required in internal code, but follow project convention.
- **`//` style for all comments.** `/* */` is acceptable for disabling code blocks but not for normal documentation.
- **Implementation comments** go above the line they describe, not at end-of-line for anything non-trivial.
- **TODO format:** `// TODO(<username>): <description>` or `// TODO(b/1234): <description>`. Flag free-form TODOs with no owner.
- **Deprecated APIs:** annotate with `[[deprecated("Use Foo instead.")]]` and document the migration in the comment.
- **Don't describe the obvious.** `// Increment i` over `i++` is noise; flag over-commented trivial code.
- **Namespace and brace closing comments:** `}  // namespace foo` and `}  // class Foo` are required for long blocks.

## Testing conventions

- **Use GoogleTest (`gtest`/`gmock`).** Flag use of other test frameworks unless the project has an explicit exception.
- **`InMemoryStore` / fakes over mocks** when a real lightweight implementation exists — prefer concrete test doubles to `MOCK_METHOD` stubs for pure data types.
- **RTTI is allowed freely in unit tests** (`dynamic_cast`, `typeid`) — this is the one accepted production-vs-test divergence.
- **`protected` data members in test fixture classes** (defined in `.cc` files) are permitted — this is the exception to the "data members private" rule.
- **Test file placement:** `foo_test.cc` lives alongside `foo.cc`; no separate `tests/` directory required by style.
- **`dummy`/fake provider pattern** for integration tests: avoid real I/O in unit tests; inject a fake via a trait/interface.
- **One `TEST` per logical behavior**, not one per function. Flag tests that do too many unrelated assertions.

## Common anti-patterns

- **`std::shared_ptr` overuse.** Shared ownership is the option of last resort. Prefer `std::unique_ptr` (sole ownership) or raw pointers/references (non-owning). Flag `shared_ptr` that could be `unique_ptr`.
- **`std::auto_ptr` usage.** Banned entirely — use `std::unique_ptr`.
- **Implicit single-argument constructors.** Every single-argument constructor must be `explicit` (except copy/move constructors and `std::initializer_list` constructors) to prevent silent type coercions. Flag missing `explicit`.
- **Implicit conversion operators.** Same rule — mark all conversion operators `explicit`.
- **`using namespace std;` (or any namespace) in a header.** Contaminates every TU that includes the file.
- **Forward declarations of external types.** Avoid forward-declaring types from other libraries; include their headers. Especially forbidden: forward-declaring anything in `std::`.
- **C-style casts.** `(int)x` is banned. Use `static_cast<int>(x)`, `absl::implicit_cast`, brace initialization for arithmetic narrowing, or `reinterpret_cast` (sparingly). Flag any `(T)expr` that isn't `(void)expr`.
- **Dynamic initialization of namespace-scope or class-scope static variables with non-trivial destructors.** `static const std::string kFoo = "foo";` is forbidden — use `constexpr` or a raw string literal, or `static const char kFoo[] = "foo";`.
- **`thread_local` at namespace/class scope without `constinit`.** Flag missing `constinit` annotation.
- **Virtual method calls in constructors.** Will not dispatch to derived implementations during construction — flag as a latent bug.
- **`protected` data members in non-test classes.** Use `private` with accessor methods instead.
- **Multiple implementation inheritance.** Flag classes that inherit implementation from more than one base; composition is usually the right fix.
- **Overloading `&&`, `||`, `,`, unary `&`, or user-defined literals.** All are explicitly prohibited.
- **`struct` with invariants or methods beyond simple accessors.** If the type has a non-trivial invariant, it must be a `class`.
- **Defaulted-but-not-explicit copy/move.** Every class must clearly express its copy/move semantics with `= default` or `= delete` in the public section.
- **Macro use for constants or function-like macros.** Prefer `constexpr` / `inline` functions. Macros have no scope, no type safety, and are flagged in review.
- **Template metaprogramming in application code.** TMP is discouraged outside infrastructure libraries; flag complex `enable_if` chains or SFINAE in application business logic.

## Expected tooling

- **`clang-format` (Google style):** enforces all whitespace, brace, and line-wrap rules. Should be run as a pre-commit/CI check. Flag any PR that skips it.
- **`clang-tidy`:** catches common bugs, modernization opportunities, and readability issues. Prefer enabling `modernize-*` and `readability-*` checks.
- **`cpplint.py`:** Google's original style linter; detects include ordering, header guard format, and other style violations not covered by clang-format.
- **Dependency vulnerability scanning:** run OSV-Scanner (or `vcpkg audit` for vcpkg projects) over third-party dependencies.
- **Address/Thread/Undefined-Behaviour Sanitisers (`-fsanitize=address,undefined,thread`):** expected in CI at minimum for test runs.
- **`absl` (Abseil):** the standard Google C++ support library. Prefer `absl::Status`, `absl::string_view`, `absl::flat_hash_map`, etc. over rolling equivalents.

## Highest-signal review checks (top ~8, prioritized)

1. **No exceptions anywhere.** `throw`, `try`, `catch`, `std::exception_ptr` — hard reject in Google C++ code. This is the single most important Google-specific rule.

2. **`explicit` on all single-argument constructors and conversion operators.** Missing `explicit` enables silent, surprising implicit conversions. Easy to miss, high impact on correctness.

3. **Header include guards use `#define`, not `#pragma once`.** Format must be `<PROJECT>_<PATH>_<FILE>_H_`. `#pragma once` is not portable per Google style.

4. **Include order is correct and each group is separated by a blank line.** Wrong order hides missing dependencies and causes brittle builds. Related header must be first in a `.cc` file.

5. **No `using namespace` in headers.** Contaminates all translation units that include the file; causes hard-to-diagnose name collisions.

6. **Ownership model is explicit.** `unique_ptr` for sole ownership; `shared_ptr` only with clear justification; non-owning raw pointers / references for everything else. `shared_ptr` where `unique_ptr` suffices is a design smell.

7. **No dynamic initialization of namespace-scope statics with non-trivial destructors.** `static const std::string kFoo = "foo";` is undefined-behavior territory across TUs. Use `constexpr` or a pointer-that-is-never-deleted pattern.

8. **No C-style casts; no RTTI in production code.** `(T)expr` → `static_cast<T>(expr)`; `dynamic_cast` → `absl::down_cast` or a virtual dispatch. Both indicate either a type-system bypass or a design flaw.
