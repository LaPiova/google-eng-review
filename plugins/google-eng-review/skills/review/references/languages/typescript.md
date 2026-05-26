# TypeScript — review profile

> Distilled from the Google TypeScript Style Guide (https://google.github.io/styleguide/tsguide.html), CC BY 3.0. Review-oriented summary, not a reproduction. Tier 1 (curated).

## Naming

- **Classes, interfaces, type aliases, enums, decorators, TSX components**: `UpperCamelCase`.
- **Variables, parameters, functions, methods, properties, module aliases**: `lowerCamelCase`.
- **Global constants and enum values**: `CONSTANT_CASE`. Only apply at module scope, not to local `const` variables.
- **`static readonly` class properties**: `CONSTANT_CASE`.
- **File names**: `snake_case` (e.g., `my_module.ts`).
- **Abbreviations as whole words**: `loadHttpUrl`, not `loadHTTPURL`. Exception: platform-spec names like `XMLHttpRequest`.
- **No Hungarian notation**, no `opt_` prefix for optional parameters, no `_` prefix/suffix on identifiers.
- **No `#` private identifiers**: Use TypeScript `private` / `protected` visibility modifiers.
- **`public` modifier**: Omit it — TypeScript symbols are public by default. The only exception is non-readonly public parameter properties in constructors.
- **Avoid single-character names** unless the variable is in scope for 10 lines or fewer.
- **Observables**: Optional `$` suffix (e.g., `clicks$`) for consistency within a team, not a hard rule.

## Formatting & idioms a reviewer should flag

- **No `var`**: Flag always. Use `const` by default; `let` only when reassignment is required.
- **One declaration per statement**: `let a = 1, b = 2;` — flag. Each variable gets its own declaration.
- **Semicolons required**: All statements must end with `;`. ASI reliance is disallowed.
- **Strict equality**: `===` and `!==` everywhere. The only exception: `== null` / `!= null` to cover both `null` and `undefined` in one check.
- **No `eval` or `Function(...string)`**: Flag always — dynamic code loading is disallowed.
- **No `with` statement**: Flag always.
- **No `debugger` statements** in production code.
- **No prototype mutation**: Adding methods to built-in constructors/prototypes is disallowed.
- **Braced blocks required**: `if`, `for`, `while` bodies always need `{}` even for single statements (except single-line `if`).
- **String literals**: Single quotes for ordinary strings. Template literals for multi-part interpolation. No backslash line continuations in strings.
- **Arrow functions over function expressions**: Use arrow functions inside methods for access to outer `this`. Named functions: prefer function declarations. Never use function expressions (except when dynamically rebinding `this` is unavoidable, which itself is discouraged).
- **Constructor calls**: Always write `new Foo()` with parentheses, even when there are no arguments.
- **`for-of` over `forEach` / `for-in`**: Prefer `for (... of arr)`. Never use unfiltered `for (... in obj)` — it iterates prototype-chain properties.
- **No wrapper types**: Never `new String(...)`, `new Boolean(...)`, `new Number(...)`. Use primitives `string`, `boolean`, `number`.
- **`readonly`**: Mark class properties that are never reassigned outside the constructor. Flag missing `readonly` on obviously immutable fields.
- **No `const enum`**: Use plain `enum`. `const enum` is opaque to JavaScript consumers and complicates builds.

## Error handling

- **Always throw `Error` instances**: `throw new Error(...)` — never throw primitives or plain objects; they produce no stack trace.
- **Reject promises with `Error`**: `reject(new Error(...))`, not `reject('message')`.
- **Catch parameter typed as `unknown`**: Use `catch (e: unknown)` and narrow via `instanceof Error` before accessing `.message`/`.stack`.
- **No empty catch blocks**: An empty `catch` must have a comment explaining why the error is being swallowed.
- **Limit try scope**: Only the risky call(s) belong inside `try`. Flag large try blocks that mix safe and unsafe operations.
- **Avoid non-nullability assertion (`!`) without justification**: Prefer runtime checks (`if (x)`) or explicit guards. When `!` is truly needed, add an explanatory comment.
- **Prefer exceptions over error-result patterns**: Do not return `{error, value}` objects; throw instead.
- **Prefer `optional` fields over `| undefined` unions** in interfaces and function parameters.
- **No nullable type aliases**: `type Result = Data | null` — flag. Add nullability at the usage site, not the alias.

## Documentation / doc-comments

- **`/** JSDoc */` for user-facing documentation** on exported symbols. Use `//` for implementation-only comments.
- **Multi-line implementation comments**: Use multiple `//` lines, not `/* ... */` blocks.
- **No decorative box comments** (rows of `*` or `-`).
- **JSDoc content is Markdown**: Use Markdown lists (`-`) not plain-text formatting inside JSDoc. Prose renders as Markdown in editors and doc generators.
- **`@param` and `@return` tags**: Include when the parameter/return meaning is not self-evident from the name and type.
- **No `@ts-ignore` / `@ts-expect-error` / `@ts-nocheck`**: Forbidden in production code. In tests, permitted only with an explanatory comment.
- **Deprecation**: Mark deprecated symbols with `@deprecated` JSDoc tag and a migration note.
- **File overview**: Files may start with a `@fileoverview` JSDoc block. Imports follow exactly one blank line after it.

## Testing conventions

- **Test method names may use `_`**: In xUnit-style frameworks, `testX_whenY_doesZ()` underscore separators are acceptable.
- **No `@ts-ignore` in tests without comment**: Even test suppressions need explanation.
- **Use `unknown` in test mocks**: When mocking with `any`, document why — prefer typed mocks.
- **TypeScript compiler must pass**: Tests included; strict mode applies to test files unless explicitly scoped out.

## Common anti-patterns

- **Default exports**: `export default Foo` — flag always. Named exports only; default exports lack a canonical name and enable inconsistent import aliases.
- **`export let`**: Flag — mutable exports have unpredictable behavior across module systems. Use an explicit getter function instead.
- **Container classes with static members**: A class whose only purpose is to namespace static methods/properties should be a module — flag and replace with individual exported functions/constants.
- **`namespace Foo { ... }`**: Flag — TypeScript namespaces are disallowed. Use ES module imports/exports.
- **`require(...)` imports**: Flag — use ES6 `import` exclusively.
- **`{}` type**: Flag — use `unknown` for opaque values, `Record<string, T>` for dictionaries, `object` to exclude primitives.
- **`any` without justification**: Flag — prefer `unknown` + type narrowing, or a specific interface.
- **Type assertions (`as Foo`) on object literals**: Flag — use a type annotation (`: Foo`) instead; it catches property mismatches at definition rather than usage.
- **Unfiltered `for-in`**: Flag — always filter with `hasOwnProperty` or use `Object.keys()` / `Object.entries()`.
- **Enum coercion**: `if (level)` where `level` is an enum value — flag. Require explicit comparison: `level !== SupportLevel.NONE`.
- **Spreading `null` / `undefined`**: `[...maybeNull]` — flag. Spreading a nullable into an array or object is a runtime error.

## Expected tooling

- **TypeScript compiler (strict mode)**: All files must pass type checking. Strict settings (e.g., `noImplicitAny`, `strictNullChecks`) are assumed.
- **tsetse / tsec**: Google's conformance frameworks enforce critical safety rules (no `eval`, no unsafe DOM APIs). Flag suppressions without justification.
- **ESLint**: Primary linting tool for style and anti-pattern enforcement.
- **clang-format or Prettier**: Formatting is expected to be consistent; formatting deviations in auto-formatted projects are hard flags.
- **`isolatedModules` compatible**: Use `export type` when re-exporting types; use `import type` for type-only imports.

## Highest-signal review checks (top ~8, prioritized)

1. **`any` type without explanation** — silently disables the type system; every `any` should have a comment justifying it or be replaced with `unknown` + type guard.
2. **Default export** — `export default` makes import names arbitrary, breaking refactoring tools; flag unconditionally and replace with named export.
3. **`var` declaration** — function-scoped, no block scoping, causes subtle bugs; `const`/`let` always.
4. **Throwing non-`Error` values** — primitives and plain objects produce no stack trace; always `throw new Error(...)`.
5. **`@ts-ignore` / `@ts-suppress` in production code** — hides type errors that will surface at runtime; flag all uses outside tests.
6. **Missing `readonly` on immutable fields** — structural correctness and intent clarity; flag class properties assigned only in the constructor.
7. **`===` violated** — any `==` or `!=` except for the `== null` idiom is a flag; loose equality masks type mismatches.
8. **Untyped catch parameter** — `catch (e)` without `: unknown` allows unsafe `e.message` access on non-Error values; always type as `unknown` and narrow.
