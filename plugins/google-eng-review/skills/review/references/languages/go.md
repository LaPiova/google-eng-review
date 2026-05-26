# Go — review profile

> Distilled from the Google Go Style Guide (https://google.github.io/styleguide/go/), CC BY 3.0. Review-oriented summary, not a reproduction. Tier 1 (curated).

## Naming

- **MixedCaps only.** All identifiers use `MixedCaps` or `mixedCaps`; never underscores (except generated code, `_test.go` files, and low-level cgo). No `MAX_PACKET_SIZE`, no `kMaxBuffer`.
- **Constants follow MixedCaps too.** `MaxRetries`, not `MAX_RETRIES` — Go has no special constant casing.
- **Package names:** lowercase, single word, no underscores (e.g., `tabwriter` not `tab_writer`). Avoid `util`, `helper`, `common`, `misc`.
- **Receiver names:** one or two letters abbreviating the type (`func (t Tray)`); never `self` or `this`; never an underscore; consistent across all methods on a type.
- **Getters omit "Get":** `c.Counts()` not `c.GetCounts()`. Exception: HTTP verbs (the word "GET" in a name is fine).
- **Initialisms hold their casing:** `URL`, `HTTP`, `ID` — never `Url`, `Http`, `Id`. Adjust only the first letter for exportedness (`urlParser` vs `URLParser`).
- **Avoid repetition in context:** if the package is `http`, don't name a type `http.HTTPServer` — just `http.Server`.
- **Test-double packages:** name them `<name>test` (e.g., `creditcardtest`); name doubles after behavior (`AlwaysCharges`, `AlwaysDeclines`).

## Formatting & idioms a reviewer should flag

- **gofmt is mandatory.** Every Go source file must be formatted by `gofmt`/`goimports`. Any diff that isn't gofmt-clean should be rejected outright.
- **No artificial line-length limits.** Go has no hard column limit; don't split lines just to stay under 80/100 chars. Refactor long expressions into helpers instead.
- **Early-return / happy-path unindented.** Handle error conditions first with early returns; keep the success path at the leftmost indent level. Avoid `else` after an error-returning `if`.
- **Accept interfaces, return concrete types.** Functions should take interface arguments and return structs or concrete types — not interfaces — unless the interface itself is the abstraction (e.g., `error`). Consumers define the interfaces they need.
- **`context.Context` is always the first parameter** (never in a struct field, never stored). Base contexts (`context.Background()`) belong only in `main`, `init`, or top-level test functions.
- **Nil slice preferred over empty slice.** Declare `var s []string`, not `s := []string{}`. Callers should not need to distinguish nil vs. zero-length.
- **Use `defer` for cleanup.** Unlock mutexes, close files, cancel contexts immediately after acquiring/opening, via `defer`. Exceptions only when defer overhead is measured.
- **Directional channels.** Parameters should use `<-chan T` or `chan<- T` to convey ownership; bidirectional `chan T` only at creation.
- **Don't shadow standard packages.** Declaring a local `url`, `sync`, or `context` variable renders the package inaccessible below that point.

## Error handling

- **Return `error` as the last return value.** Never use sentinel in-band values (`-1`, `nil` to mean failure). Return `(result, error)`.
- **Error strings: lowercase, no trailing punctuation.** `"failed to open file"` not `"Failed to open file."` — they get concatenated in larger messages.
- **Wrap with `%w` to preserve inspectability.** Use `fmt.Errorf("context: %w", err)` when callers may need `errors.Is`/`errors.As`. Use `%v` at system boundaries (RPC, storage) where you're translating errors and don't want callers to depend on internals.
- **Place `%w` at the end:** `"reading config: %w"` mirrors how error chains print. Exception: sentinel errors at the beginning for category visibility.
- **Never silently drop errors.** Using `_` for an error return requires a comment explaining why it is safe. Never pass an `errgroup` error off into the void.
- **Use sentinel errors or structured types for programmatic distinction.** Never `strings.Contains(err.Error(), "...")` — define `var ErrNotFound = errors.New(...)` or a custom type with `errors.Is`/`errors.As`.
- **Panics must not cross package boundaries.** Use `defer recover()` at public API surfaces to translate panics into returned errors. Libraries should return errors, not panic.
- **Return concrete error types cautiously.** Returning a concrete `*MyError` from a function that might return `nil` risks interface nil-pointer bugs; return the `error` interface.

## Documentation / doc-comments

- **All exported top-level names require a doc comment.** Functions, types, variables, and constants that are exported must have a Godoc comment or the comment is missing.
- **Doc comments start with the symbol name.** `// Request represents an outgoing HTTP request.` not `// This is a request.`
- **Package comment appears immediately above `package` — no blank line.** One comment per package; use `doc.go` if it's unusually long.
- **Complete sentences: capitalised and punctuated.** Fragments are fine in non-doc contexts; doc comments are always complete sentences.
- **Don't document what the function signature already says** (parameter types, return types) — add domain context not visible from the signature.

## Testing conventions

- **Table-driven tests.** Group cases in a slice of structs; use `t.Run` for subtests; name test cases descriptively; omit zero-value fields.
- **Failure messages follow `YourFunc(%v) = %v, want %v`.** Use `got`/`want` terminology. Include function name, inputs, actual, expected.
- **Use `cmp.Equal` / `cmp.Diff` for complex comparisons.** Avoid external assertion libraries; Go's built-in reporting is sufficient and clearer.
- **Prefer `t.Error` over `t.Fatal`** so all failures in a run are visible. Use `t.Fatal` only when proceeding would be meaningless (e.g., nil pointer would panic).
- **Never call `t.Fatal` from a goroutine.** Use `t.Error` + `return` inside spawned goroutines; `t.Fatal` is only safe in the test's own goroutine.
- **Mark test helpers with `t.Helper()`.** This attributes failure line numbers to the call site, not the helper internals.
- **`InMemoryStore` / test doubles instead of mocks.** Go idiom is concrete fakes; avoid mock frameworks that obscure control flow. For services, prefer real transports to test doubles of the client.
- **`t.Cleanup` for resource teardown** (Go 1.14+) rather than deferred calls in helper functions that lose context.

## Common anti-patterns

- **`util`, `common`, `helper` packages.** Symptom of insufficient abstraction; name packages after what they provide.
- **Overlong function signatures.** More than ~4-5 parameters suggests an option struct or function decomposition.
- **Contexts in structs** (`s.ctx`). Pass context explicitly down the call chain; never store it.
- **Interface types exported by the producer** rather than defined by the consumer. The caller should define the minimal interface they need; large exported interfaces ossify the API.
- **goroutines without a clear exit path.** Every spawned goroutine must have a visible termination condition (`context.Context`, `sync.WaitGroup`, close channel). Goroutines that can leak are bugs.
- **`recover()` to suppress panics broadly.** Masking unexpected panics propagates corrupted state; only recover from panics your own code raises, and re-panic on anything unexpected.
- **Duplicate logging.** Libraries should not log errors they return — callers decide what to log. Logging and returning is double-reporting.
- **Variable shadowing.** `:=` in an inner scope hides the outer variable; easy to miss, hard to debug. Use `=` when modifying an existing binding.

## Expected tooling

| Tool | Purpose |
|------|---------|
| `gofmt` / `goimports` | Canonical formatting; enforced as a hard gate |
| `go vet` | Catches common correctness mistakes (printf verbs, unreachable code, etc.) |
| `staticcheck` | Extended static analysis beyond `go vet` |
| `govulncheck` | Dependency vulnerability scanning |
| `go test -race` | Data-race detector; run in CI |
| `golangci-lint` | Aggregates multiple linters in one pass |

A PR failing `gofmt`, `go vet`, or `staticcheck` should be blocked, not merely noted.

## Highest-signal review checks (top ~8, prioritized)

1. **gofmt clean?** — Reject immediately if not. No discussion needed.
2. **Error strings lowercase, no punctuation, wrapped with `%w` appropriately?** — Most common nit that silently breaks `errors.Is` chains.
3. **Exported symbols have doc comments starting with the symbol name?** — Godoc breaks without this; easy to miss under time pressure.
4. **No context stored in struct fields?** — Subtle lifetime and cancellation bugs; should be passed explicitly.
5. **Goroutine has a clear shutdown path?** — Goroutine leaks accumulate silently and cause OOM in production.
6. **Error silently discarded (`_` or no check)?** — Each unchecked error is a potential silent data-loss or crash.
7. **Accept interfaces / return concrete types respected?** — Returning an interface where a struct suffices couples callers to implementation details.
8. **Table-driven test with meaningful failure messages?** — Tests that don't show inputs/expected/actual waste debugging time on every future failure.
