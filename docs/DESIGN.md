# Design: `google-eng-review` plugin

Date: 2026-05-25
Status: approved (brainstorming complete) — ready for implementation

## Purpose

A Claude Code plugin that performs **language-agnostic code review grounded in Google's
published engineering practices**. It draws on two bodies of Google guidance:

- **`google/eng-practices`** — the *universal* review process and judgment (how to review,
  what to look for, how fast, how to write comments, how to handle pushback) plus the
  CL author's guide (good descriptions, small CLs, handling reviewer comments).
- **`google/styleguide`** — *per-language* best practices, applied to the language(s)
  actually present in the change.

It also covers the comments / documentation / test-case dimensions carried over from the
existing personal `senior-code-review` skill, where `eng-practices` is silent or thin.

The plugin is **general-purpose and language-agnostic**: it always applies the universal
principles, and resolves the right language guidance for whatever language(s) appear in a
change. It is **not** scoped to any fixed language list — being authored inside a Rust repo
does not privilege Rust (Google publishes no Rust guide, so Rust is just an ordinary
community-idioms entry).

## Provenance & sourcing

Reference content is **distilled offline** (committed into the plugin) from:

- `google/eng-practices` (`review/reviewer/*`, `review/developer/*`)
- `google/styleguide` (per-language guides)

Source material is fetched via **WebFetch** at authoring time (user-approved). Reference
files are deterministic and require **no network access at review time**.

## Naming

| Component | Name | Collision check |
|---|---|---|
| Plugin | `google-eng-review` | distinct from `code-review`, `pr-review-toolkit`, `security-guidance` |
| Skill (the entry) | `review` → invoked `/google-eng-review:review` | plugin skills are *always* namespaced, so this cannot collide with the bare `/review`/`/code-review` skills |
| Subagent (optional, deferred) | `google-eng-reviewer` | — |

**Note on invocation:** Claude Code namespaces all plugin skills and commands as
`/plugin:name`, so a bare `/google-eng-review` is not reliable. The canonical, robust
invocation is **`/google-eng-review:review`**. A single forked skill (no separate command
file) is used because it gives `context: fork` + bundled-reference access directly and
avoids command→skill delegation pitfalls — mirroring the existing `senior-code-review`
skill's `disable-model-invocation: true` + `context: fork` pattern.

## Plugin structure

```
google-eng-review/
├── .claude-plugin/
│   └── plugin.json                  # manifest: name, version, description, author
└── skills/
    └── review/                      # → invoked /google-eng-review:review
        ├── SKILL.md                 # forked entry: workflow + language resolution + output format
        └── references/
            #   files referenced from SKILL.md via ${CLAUDE_SKILL_DIR}/references/...
            ├── reviewers-guide.md       # eng-practices reviewer guide, distilled
            ├── authors-guide.md         # eng-practices CL-author guide, distilled
            ├── universal-principles.md  # cross-language dimensions (see below)
            ├── language-index.md        # language → {curated file | Google guide | idioms} map
            └── languages/               # extensible CACHE of pre-distilled profiles (a seed, not the scope)
                ├── python.md
                ├── go.md
                ├── java.md
                ├── typescript.md
                └── cpp.md
```

### `reviewers-guide.md` (from `google/eng-practices`)
Distilled from the 6 reviewer pages:
1. The Standard of Code Review — approve when it definitely improves overall code health,
   even if imperfect; mentorship; no perfectionism; principle over personal preference.
2. What to Look For — design, functionality, complexity, tests, naming, comments, style,
   consistency, documentation, every line, context, and *calling out good things*.
3. Navigating a CL in Review — broad view first, then main parts, then the rest.
4. Speed of Code Reviews — respond within ~one business day; why speed matters.
5. How to Write Code Review Comments — courteous, explain reasoning, balance directives
   vs. guidance, prefix true nits with `Nit:`.
6. Handling Pushback — facts and principles over opinions; resolve conflicts.

### `authors-guide.md` (from `google/eng-practices`)
1. Writing Good CL Descriptions. 2. Small CLs. 3. How to Handle Reviewer Comments.
Used so the review can also assess the *change's* description and scope, not just code.

### `universal-principles.md` (cross-language, carries `senior-code-review` coverage)
Correctness & edge cases · error handling · concurrency/races · security & input validation ·
performance & resource management · API design & contracts · dependencies · testability &
test coverage · documentation & comments (explain *why*, doc public APIs, stale-comment check).
**These apply to every language, including uncurated ones.**

### `language-index.md` (the language-agnostic dispatch map)
A table: `language → resolution`. Each row says whether a curated profile exists, and if
not, which Google style guide URL covers it (for synthesis) or that only community idioms
apply. Drives the resolution algorithm below and makes adding a language a one-row change.

### `languages/<lang>.md` (curated cache — seed set)
Seed criterion: widely-used languages that have a **published Google style guide**.
Initial seed: **Python, Go, Java, TypeScript, C++**. Each contains, distilled from the
relevant `google/styleguide`: naming conventions · formatting/idioms a reviewer should flag ·
error-handling norms · doc-comment conventions · testing conventions · common anti-patterns ·
expected tooling (`ruff`/`black`, `gofmt`/`go vet`, `google-java-format`, `tsc`/`eslint`,
`clang-format`/`clang-tidy`). The set is an **extensible cache**, not a scope boundary.

## Language resolution (language-agnostic core)

1. Resolve the change set: slash-command argument, else default `git diff HEAD`.
2. Detect language(s) by file extension + project manifests (`Cargo.toml`, `pyproject.toml`,
   `go.mod`, `package.json`, `pom.xml`, `CMakeLists.txt`, …).
3. For **each** detected language, resolve a best-practice profile in order:
   a. **curated file** in `references/languages/` if one exists; else
   b. **synthesize** from the model's knowledge of that language + the **Google style guide
      for it** (per `language-index.md`), fetching only if the user permits; else
   c. **universal principles + community idioms only.**
4. Apply `universal-principles.md` + the eng-practices guides to *everything*; apply each
   resolved language profile to its own files.
5. **Always state which resolution path was used** per language, so output is honest about
   fidelity (curated vs. synthesized vs. idioms-only).

This means **any language works on day one** via the fallback; the curated cache only raises
fidelity for the seeded languages and is expanded by dropping in new files + index rows.

## Output format

Reuses the proven `senior-code-review` structure:
**Summary → Critical 🔴 → Important 🟡 → Minor 🟢 → Positive 👍.**
Every finding cites `file:line`, states *why* it matters, gives a concrete fix, and cites
the originating principle/guide. Findings are *written* per Google's comment etiquette:
courteous, reasoning explained, true nits prefixed `Nit:`.

## Invocation

- Skill frontmatter: `disable-model-invocation: true` + `context: fork` (same as
  `senior-code-review`) — invoked explicitly via `/google-eng-review:review`, so it does not
  fight the existing `code-review` / `review` skills for model triggering.
- Bundled reference files are read from `${CLAUDE_SKILL_DIR}/references/...` so they resolve
  inside the forked context regardless of the user's working directory.
- A `google-eng-reviewer` subagent is **deferred**; `context: fork` already gives isolation,
  so it is not needed for v0.1.

## Relationship to existing tools

`senior-code-review` (personal skill) is **left untouched**. `code-review` / `review` /
`security-review` are unchanged.

## Build manifest (files)

1. ✅ `.claude-plugin/plugin.json`
2. ✅ `skills/review/SKILL.md` (forked entry; workflow + language resolution + output format)
3. ✅ `skills/review/references/reviewers-guide.md`
4. ✅ `skills/review/references/authors-guide.md`
5. ✅ `skills/review/references/universal-principles.md`
6. ✅ `skills/review/references/language-index.md`
7–11. ✅ `skills/review/references/languages/{python,go,java,typescript,cpp}.md`
12. `agents/google-eng-reviewer.md` — deferred (fork already isolates).

Independent reference/language files (3–11) were authored in parallel.

## Open items

- Whether to package as its own installable marketplace (add a `marketplace.json`) — defer
  until the plugin works locally.
- Expanding the curated cache beyond the seed (C#, Shell, Swift, JS, R, …) — extend later
  by adding a file + a `language-index.md` row.
