# Java — review profile

> Distilled from the Google Java Style Guide (https://google.github.io/styleguide/javaguide.html), CC BY 3.0. Review-oriented summary, not a reproduction. Tier 1 (curated).

## Naming

- **Classes and interfaces:** `UpperCamelCase`, typically nouns or noun phrases (`ImmutableList`, `HttpRequest`). Test classes end with `Test` (`UserServiceTest`).
- **Methods:** `lowerCamelCase`, typically verb phrases (`sendMessage`, `computeTotal`). Test method names may use underscores to separate logical components (`transferMoney_insufficientFunds_throwsException`).
- **Constants:** `UPPER_SNAKE_CASE` — only for `static final` fields that are deeply immutable with no observable side effects (primitives, strings, immutable collections). A `static final List` that is mutable is NOT a constant.
- **Non-constant fields:** `lowerCamelCase` nouns (`computedValues`, `userIndex`).
- **Local variables:** `lowerCamelCase` even when `final` — immutable locals are not constants and must not use `UPPER_SNAKE_CASE`.
- **Parameters:** `lowerCamelCase`; avoid single-character names in public methods.
- **Type variables:** single capital letter optionally with a numeral (`E`, `T`, `T2`), or a class-name followed by `T` (`RequestT`, `RowT`).
- **Packages:** all lowercase, no underscores; consecutive words concatenate (`com.example.deepspace` not `com.example.deep_space`).
- **No Hungarian notation / special prefixes.** `name_`, `mName`, `s_name`, `kName` are all violations.

## Formatting & idioms a reviewer should flag

- **2-space indentation, not 4, not tabs.** Block-level indentation increases by exactly 2 spaces per level; continuation lines indent at least +4 from the originating line.
- **100-character column limit.** Exceptions: package/import lines, text block content, shell commands in comments.
- **K&R brace style for non-empty blocks.** Opening brace on the same line (no line break before it); closing brace on its own line. Empty blocks may be `{}` on one line unless they are a clause in `if/else` or `try/catch/finally`.
- **One statement per line.** Never `a = 1; b = 2;` on one line.
- **One variable per declaration.** `int a, b;` is forbidden. Exception: the header of a `for` loop.
- **Local variables declared close to first use,** not at the top of the method. Minimal scope is the goal.
- **No wildcard or module imports.** `import java.util.*;` and module imports are banned. Use specific imports in ASCII-sorted order; static imports in their own group, separated by a blank line from non-static imports.
- **Modifier ordering:** `public protected private abstract default static final sealed non-sealed transient volatile synchronized native strictfp`. Any other order is wrong.
- **No horizontal alignment.** Aligning fields or variable names with extra spaces to form columns is explicitly "never required" and must not be introduced or maintained.
- **Long integer literals use uppercase `L`:** `3_000_000_000L` not `3000000000l` (lowercase `l` is indistinguishable from `1`).
- **Static members accessed via class name,** never via instance: `Foo.staticMethod()` not `instance.staticMethod()`.
- **No `Object.finalize` override.** Finalization is deprecated and scheduled for removal.
- **Switch exhaustiveness + mandatory `default` label.** Every switch must be exhaustive; a `default` label is required even when the language doesn't mandate it, even if the body is empty.
- **Old-style switch fall-through must have a `// fall through` comment.** Every statement group in a traditional (`:`-style) switch must either terminate abruptly or carry this comment. New-style (`->`-style) switches have no fall-through.

## Error handling

- **Never silently ignore caught exceptions.** The only acceptable responses are: log the exception, rethrow (as-is or wrapped), or assert it's impossible (`throw new AssertionError(e)`). If genuinely no action is needed, document it with a comment explaining why.
- **Comment every empty catch block.** An undocumented empty `catch` is an automatic review failure — it buries failures and makes debugging impossible.
- **Catch specific exception types.** Catching `Exception` or `Throwable` broadly obscures bugs; flag unless at a top-level boundary (framework entry points, `main`).
- **Don't use exceptions for flow control.** Throwing and catching exceptions for expected branching is expensive and unreadable.

## Documentation / doc-comments

- **Javadoc required for every visible class, member, and record component.** "Visible" means `public` or `protected` within a visible class.
- **Exception: trivially obvious members.** A `getFoo()` accessor with truly nothing worth saying may omit Javadoc, but the guide warns against over-applying this exception.
- **Exception: overriding methods.** Not always required when the supertype's Javadoc suffices; `@Override` handles inheritance.
- **Summary fragment opens every Javadoc block.** It must be a noun or verb phrase — capitalized and punctuated as if a complete sentence — but should not start with "This method returns..." or "A `Foo` is a...". It must stand alone as a one-sentence description.
- **Block tag ordering:** `@param`, `@return`, `@throws`, `@deprecated`. Never use these tags with empty descriptions. Continuation lines indent 4+ spaces from the `@`.
- **Paragraphs separated by `<p>` tags** on their own line (no space between `<p>` and first word), preceded by a blank asterisk line.
- **`@Override` on every legal override,** including interface implementations and record accessor overrides. Only exception: the parent method carries `@Deprecated`.
- **Annotations on their own line** for classes, packages, modules, methods, and constructors. Exception: a single parameterless annotation may share the signature line (`@Override public int hashCode() { ... }`). Field annotations may be stacked on one line.

## Testing conventions

- **Test class names end with `Test`.** Integration or end-to-end variants may use `IT` or `IntegrationTest` suffixes by team convention.
- **Test method names describe behavior,** often following `methodUnderTest_condition_expectedResult` or `given_when_then` patterns. The guide allows underscores in test method names specifically for readability.
- **No catching exceptions to assert failure.** Use `assertThrows` (JUnit 5) or `@Test(expected=...)` (JUnit 4) rather than a `try/catch` block with a `fail()` at the end — easy to accidentally write a test that never fails.
- **One logical assertion per test** (where practical). Multiple assertions in one test obscure which condition caused a failure.
- **Use `@Before`/`@After` (`@BeforeEach`/`@AfterEach`) for setup/teardown,** not hand-rolled helpers that duplicate logic across tests.
- **Test doubles (mocks, stubs, fakes) should be named for their behavior** when multiple variants exist. Prefer fakes with real logic over mocks that just stub return values when the interaction is non-trivial.

## Common anti-patterns

- **Wildcard imports.** `import java.util.*;` obscures what is actually used and can cause subtle collisions when APIs expand. Automatic failure.
- **Mutable `static final` fields named as constants.** `static final List<String> ITEMS = new ArrayList<>()` is not a constant — it's a mutable field with a deceptive name.
- **Hungarian notation / naming prefixes.** `mField`, `sStatic`, `kConstant` are Go/C++ conventions, not Java. Google style bans all such prefixes.
- **`int a, b, c;` multi-variable declarations.** Each declaration must be on its own line; makes the types and initialisers clear.
- **Silent empty catch.** An empty `catch` block with no comment is the single most reliable way to bury failures in production.
- **Instance-qualified static access.** `myFoo.staticBar()` compiles but misleads readers into thinking the call is polymorphic.
- **`Object.finalize` override.** The JVM provides no guarantee of when (or whether) finalizers run; use `Cleaner` or `AutoCloseable`/try-with-resources instead.
- **`else` after a `return`/`throw`.** Unnecessary `else` adds nesting without adding information; flatten the branch.
- **Horizontal alignment of fields.** Creates spurious diffs when any field name changes length; prohibited by the style guide.

## Expected tooling

| Tool | Purpose |
|------|---------|
| `google-java-format` | Canonical formatter; should be applied before every commit |
| Error Prone | Compile-time bug pattern checker; catches common Java mistakes not caught by `javac` |
| Checkstyle (Google config) | Enforces structural style rules (import ordering, braces, naming) |
| SpotBugs / FindBugs | Runtime-pattern static analysis (null dereferences, resource leaks) |
| `javac -Xlint:all` | Enables all compiler warnings; clean builds should be warning-free |

A PR that fails `google-java-format` or Checkstyle should be blocked at CI, not reviewed manually for formatting.

## Highest-signal review checks (top ~8, prioritized)

1. **`google-java-format` applied?** — Reject on sight if formatting is manual; it creates noisy diffs and is 100% automatable.
2. **Every visible class/member has Javadoc, summary fragment non-trivial?** — Public API without Javadoc is documentation debt shipped as code.
3. **Empty or undocumented `catch` block?** — The single most reliable way to lose exception information in production; zero tolerance.
4. **`@Override` on every legal override?** — Missing `@Override` lets a refactored method silently stop overriding anything; bugs hide for months.
5. **Wildcard import present?** — Banned outright; auto-fail and demand specific imports.
6. **`static final` field styled as `CONSTANT` but actually mutable?** — Misleads readers about thread-safety and immutability guarantees.
7. **Static member accessed through an instance (`obj.STATIC_FIELD`)?** — Misleads readers; static analysis flags it, so its presence implies tooling is not running.
8. **Switch missing `default` label or old-style fall-through without comment?** — Silent fall-through and non-exhaustive switches are a classic source of subtle bugs that only manifest at runtime.
