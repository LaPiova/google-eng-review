# Language index — how to resolve per-language guidance

This map drives the **language resolution** step in `SKILL.md`. For each language detected in
the change, resolve a best-practice profile using the highest available tier:

- **Tier 1 — curated:** a distilled profile exists in `languages/`. Load and apply it.
- **Tier 2 — Google guide:** no curated file, but Google publishes a style guide. Apply that
  guide's conventions from your own knowledge (optionally fetch the URL if the user permits).
- **Tier 3 — idioms only:** no curated file and no Google guide. Apply `universal-principles.md`
  plus the language's widely-accepted community idioms from your own knowledge.

**Always state, in the review output, which tier was used for each language** (e.g.
"Java reviewed against curated Google Java Style profile (Tier 1)"; "Rust reviewed against
community idioms — no Google guide (Tier 3)"). This keeps fidelity honest.

Adding a language later = add a `languages/<lang>.md` file and flip its row to Tier 1.

## Map

| Language | Detect by | Tier | Source |
|---|---|---|---|
| Python | `.py`, `pyproject.toml`, `setup.py`, `requirements.txt` | 1 | `languages/python.md` ← Google Python Style Guide |
| Go | `.go`, `go.mod` | 1 | `languages/go.md` ← Google Go Style Guide |
| Java | `.java`, `pom.xml`, `build.gradle` | 1 | `languages/java.md` ← Google Java Style Guide |
| TypeScript | `.ts`, `.tsx`, `tsconfig.json` | 1 | `languages/typescript.md` ← Google TypeScript Style Guide |
| C++ | `.cc`, `.cpp`, `.h`, `.hpp`, `CMakeLists.txt` | 1 | `languages/cpp.md` ← Google C++ Style Guide |
| JavaScript | `.js`, `.jsx`, `.mjs`, `package.json` | 2 | Google JavaScript Style Guide (google.github.io/styleguide/jsguide.html) |
| C# | `.cs`, `.csproj` | 2 | Google C# Style Guide |
| Shell | `.sh`, `.bash` | 2 | Google Shell Style Guide |
| Swift | `.swift` | 2 | Google Swift Style Guide |
| Objective-C | `.m`, `.mm` | 2 | Google Objective-C Style Guide |
| R | `.R`, `.r` | 2 | Google R Style Guide |
| HTML/CSS | `.html`, `.css`, `.scss` | 2 | Google HTML/CSS Style Guide |
| **Rust** | `.rs`, `Cargo.toml` | 3 | community idioms: `rustfmt`, `clippy`, Rust API Guidelines (no Google guide) |
| Kotlin | `.kt`, `.kts` | 3 | community/JetBrains + Android Kotlin style (Google guide lives outside styleguide repo) |
| Dart | `.dart`, `pubspec.yaml` | 3 | community/Effective Dart (Google guide lives outside styleguide repo) |
| *anything else* | — | 3 | `universal-principles.md` + community idioms from your knowledge |
