# Universal review principles (language-agnostic)

These dimensions apply to **every** change in **every** language — including languages with
no curated profile and no published Google style guide. Apply them alongside the
language-specific profile (see `language-index.md`) and the review philosophy in
`reviewers-guide.md`.

For each finding, name the dimension and explain *why* it matters in this change. Prioritize
by impact: a correctness or security defect outranks any number of style nits.

## 1. Correctness & logic
- Is the logic correct on all paths, including edge and boundary conditions (empty, zero,
  one, max, negative, overflow)?
- Off-by-one, integer overflow/truncation, wrong operator precedence, incorrect rounding?
- Are absent/optional/default values handled (null/None/nil/Option, empty collections)?
- Are invariants, pre-, and post-conditions maintained? Does the code do what the CL says?

## 2. Error handling & failure modes
- Is every error path handled or deliberately propagated — never silently swallowed?
- Behavior under failure: network errors, timeouts, partial writes, disk full, invalid input?
- Are errors actionable (enough context to diagnose), and not leaking sensitive detail?
- Resources released on the error path too (no leak when an exception/early-return fires)?

## 3. Concurrency & state
- Data races, torn reads/writes, check-then-act races on shared state?
- Deadlock/lock-ordering risks; locks held across I/O or await points?
- Is shared mutable state actually shared, and is the synchronization sound?

## 4. Security & input validation
- Is untrusted input validated and sanitized at the boundary?
- Injection risks: SQL, shell/command, path traversal, template, deserialization, SSRF?
- Secrets/credentials/PII: not logged, not hardcoded, not committed?
- Authz/authn checks present and correct; least privilege respected?

## 5. Performance & resource management
- Unnecessary allocations/copies; accidental O(n²) where O(n) is available; unbounded growth?
- Resources (handles, connections, locks, memory) acquired and released deterministically?
- Avoid premature optimization — flag real, reasoned hotspots, not speculative micro-tuning.

## 6. Design & architecture
- Does the change fit the existing architecture, or introduce an inconsistency?
- Are abstraction boundaries respected; are implementation details leaking across modules?
- Right level of abstraction — not over-engineered, not under-abstracted? (YAGNI.)
- Could this be simpler? Complexity that isn't justified by a concrete need is a defect.

## 7. API design & contracts
- Public surface minimal and hard to misuse; "make illegal states unrepresentable"?
- Signatures clear about ownership, mutability, nullability, effects, thread-safety?
- Breaking changes identified and justified? Backward/forward compatibility considered?

## 8. Naming & readability
- Do names precisely describe intent, without ambiguity or misleading terms?
- Consistent with conventions already in this codebase?
- Could a new teammate understand this without a verbal explanation?

## 9. Documentation & comments
- Public APIs / exported items documented per the language's doc convention.
- Comments explain **why**, not **what** the code obviously does.
- Are existing comments/docs still accurate after this change, or now stale/misleading?
- `TODO`/`FIXME`/`HACK` justified and tracked (issue link), not abandoned.

## 10. Tests
- Are there tests for this change, including failure paths and edge cases — not just the
  happy path? (This is a first-class review dimension, not optional.)
- Are the tests correct, deterministic, and actually asserting behavior (no vacuous asserts)?
- Do they test behavior/contract rather than implementation detail?
- Did existing tests need updating, and are they still meaningful after the change?
- Is the code structured to be testable (injectable deps, clear seams)?

## 11. Dependencies & integration
- Are new dependencies justified, maintained, appropriately licensed, and version-pinned?
- Could this be done without the dependency? Any transitive/security concern?
- No circular dependencies introduced; compatible with the deployment/config model?

## 12. The change itself (CL hygiene — see `authors-guide.md`)
- Is the CL appropriately small and single-purpose, or should it be split?
- Does the description explain *what* and *why* well enough to stand in the VCS history?
