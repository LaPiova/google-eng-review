# google-eng-review

A Claude Code plugin for **language-adaptive code review grounded in Google's published
engineering practices**.

- **Universal review judgment** comes from [`google/eng-practices`](https://github.com/google/eng-practices)
  — the standard of code review, what to look for, navigating a CL, speed, how to write
  comments, and handling pushback, plus the CL author's guide.
- **Per-language best practices** come from [`google/styleguide`](https://github.com/google/styleguide),
  applied to the language(s) actually present in the change.

It detects the language(s) in a change and applies the matching guidance, with a **dynamic
fallback** so *any* language is covered — not just a fixed list.

## Install

Clone the repo, then add it as a local plugin marketplace:

```bash
git clone https://github.com/LaPiova/google-eng-review.git
```

In Claude Code (run from the directory where you cloned it):

```
/plugin marketplace add ./google-eng-review
/plugin install google-eng-review@eng-review
```

Then reload Claude Code so the plugin loads.

## Usage

Plugin skills are namespaced, so invoke it as:

```
/google-eng-review:review                 # reviews the current working tree
/google-eng-review:review src/             # reviews changes under a path
/google-eng-review:review 42               # reviews PR #42 (uses `gh` if available)
```

The skill runs in an isolated (`context: fork`) review pass and returns a structured report:
**Summary → Critical 🔴 → Important 🟡 → Minor/Nits 🟢 → Positive 👍**, with every finding
citing `file:line`, the reason, a concrete fix, and the principle/guide it comes from.

## How language resolution works

For each language in the change, a best-practice profile is resolved by tier
(see `skills/review/references/language-index.md`):

1. **Tier 1 — curated:** a distilled profile exists in `references/languages/`
   (seeded: Python, Go, Java, TypeScript, C++ — the widely-used languages with a published
   Google style guide).
2. **Tier 2 — Google guide:** no curated file, but Google publishes a guide (JS, C#, Shell,
   Swift, Obj-C, R, …) → apply it from the model's knowledge.
3. **Tier 3 — idioms only:** no curated file and no Google guide (e.g. **Rust**, Kotlin,
   Dart) → universal principles + community idioms.

The review states which tier was used per language, so fidelity is explicit.

### Adding a language

Drop a `references/languages/<lang>.md` profile in (same section structure as the others)
and flip its row to Tier 1 in `references/language-index.md`. No code changes.

## Structure

```
.claude-plugin/{plugin.json, marketplace.json}
skills/review/
  SKILL.md                       # forked review workflow + language resolution + output format
  references/
    reviewers-guide.md           # eng-practices reviewer guide (distilled)
    authors-guide.md             # eng-practices CL-author guide (distilled)
    universal-principles.md      # cross-language review dimensions
    language-index.md            # language → resolution tier map
    languages/{python,go,java,typescript,cpp}.md
docs/DESIGN.md                   # design rationale
```

## Attribution & licensing

The reference files are **review-oriented distillations** (summaries, not reproductions) of:

- Google Engineering Practices — <https://github.com/google/eng-practices> — © Google,
  licensed **CC BY 3.0**.
- Google Style Guides — <https://github.com/google/styleguide> — © Google, licensed
  **CC BY 3.0**.

Each reference file links to its source. This plugin is not affiliated with or endorsed by
Google.
