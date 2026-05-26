> Source: google/eng-practices (https://google.github.io/eng-practices/), licensed CC BY 3.0. This file summarizes and does not reproduce the source verbatim.

# Author's Guide — What Reviewers Should Assess About CL Descriptions, Scope, and Author Behavior

This guide distills Google's guidance written for CL authors. A reviewer should use it to evaluate not just the code but also the *quality of the change itself* as a unit of work: its description, its size, and how the author engages with feedback.

---

## 1. Writing Good CL Descriptions

A CL description is a permanent public record in version control history. It will be read by future engineers searching for context on why the system is the way it is. A reviewer should flag a weak description just as they would flag bad code.

**What a good description must contain:**

- **First line:** A short, imperative-sentence summary of *what* is being done — specific enough to stand alone in a commit log. Example: `Delete the FizzBuzz RPC and replace it with the new system.` Not: `Deleting the FizzBuzz RPC.` or `Fix bug.`
- **Body:** Explains *why* the change is being made, what problem it solves, what tradeoffs were made, and any context (bug numbers, benchmark results, design doc links) that is not obvious from the code. Even small CLs deserve a body that puts them in context.
- The description should still be accurate when the CL is submitted — if the code changed significantly during review, the description must be updated to match.

**Red flags a reviewer should call out:**

- Vague first lines: `Fix bug`, `Fix build`, `Add patch`, `Phase 1`, `Moving code from A to B`, `Add convenience functions`.
- No body, or a body that only restates the first line without explaining *why*.
- A description that no longer matches what the CL actually does (author updated code but not the description).
- External links as the only source of context — those links may rot; enough context must exist inline.

**Tags** (e.g., `[refactor]`, `#bugfix`) are optional. If used, keep them short and do not let them crowd out the actual summary in the first line.

---

## 2. Small CLs

Small, focused CLs are one of the highest-leverage practices in a review culture. A reviewer should both *benefit from* small CLs and *request splitting* when a CL is too large.

**Why size matters (reviewer perspective):**

- A reviewer can find five minutes several times for small CLs; a 30-minute block for a large one is hard to schedule.
- Smaller CLs receive more thorough, higher-quality review. Large CLs invite rubber-stamping.
- Small CLs are easier to roll back if something goes wrong.
- Fewer changes per CL means fewer interactions and faster, cleaner merge histories.

**What "small" means:**

- A CL should represent *one self-contained change* with a single clear objective.
- ~100 lines changed is a healthy rough target. ~1,000 lines is almost always too much.
- File count matters too: 200 lines in one file is different from 200 lines spread across 50 files.
- Tests should be included in the same CL as the code they test — do not review untested production code expecting tests to appear separately later.
- The CL should give the reviewer enough context to understand it without requiring external reading.

**When to request splitting:**

- The CL touches multiple unrelated concerns (e.g., a refactor bundled with a feature addition).
- The reviewer cannot hold the full change in mind at once.
- The first line of the description requires multiple clauses to capture what the CL does.

**Valid splitting strategies** an author should be directed toward:

- **Stacking**: Submit CL 1, begin CL 2 based on it while awaiting review.
- **By reviewer**: Split by which team owns which files.
- **Horizontal layers**: Separate interface/abstraction from implementation.
- **Vertical features**: Each CL is a complete, independently shippable vertical slice of functionality.
- Each intermediate CL must leave the system in a buildable, working state — do not split in a way that breaks the build or tests mid-stack.

---

## 3. How Authors Should Handle Reviewer Comments

A reviewer assessing a CL should also notice *how the author is engaging* — a hostile, dismissive, or emotionally reactive author is a process problem even if the code is correct.

**What healthy author behavior looks like:**

- Treats critical feedback as assistance for improving code quality, not as a personal attack.
- Does not reply with anger or sarcasm. (Angry replies are permanent records and indicate a collaboration problem.)
- When they don't understand a reviewer comment, asks a clarifying question rather than ignoring or dismissing it.
- When a reviewer cannot understand their code, improves the *code* (or adds a clarifying comment in source) rather than only explaining it in the review thread. Review threads disappear; clearer code stays.
- Engages with the substance of disagreements: explains tradeoffs, provides evidence, discusses alternatives. Does not simply assert "this is fine."

**Escalation path for author:**

If a reviewer is being persistently unconstructive (rude, vague, blocking on taste rather than principle), the appropriate path is: (1) discuss in person or via call, (2) send a private, respectful message, (3) escalate to a manager if unresolved. The answer is never to abandon quality standards or bypass review.

**Reviewer implication:** If an author's responses in the review thread are dismissive, evasive, or show they do not understand their own code, that is itself a reason to withhold approval until the code and discussion reflect genuine understanding.
