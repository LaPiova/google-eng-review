> Source: google/eng-practices (https://google.github.io/eng-practices/), licensed CC BY 3.0. This file summarizes and does not reproduce the source verbatim.

# Reviewer's Guide — Key Principles for Code Review

---

## 1. The Standard of Code Review

**The core bar:** Approve a CL when it *definitely improves overall code health*, even if it is not perfect. There is no perfect code — only better code. Do not block on the pursuit of perfection.

- A CL must not be approved if it *degrades* code health (adds net complexity, introduces bad patterns, creates technical debt without countervailing benefit).
- Approve imperfect CLs that represent genuine net improvement; note remaining issues as nits or follow-up items.
- Distinguish **principle** from **personal preference**. Block on the former; never block on the latter. Style guides are the authority on style disputes — defer to them, not taste.
- Use `Nit:` prefix to signal a comment is non-blocking. Educational/mentorship comments should be labeled as optional or informational so the author is not confused about what is required.
- When disagreement cannot be resolved collaboratively, escalate to tech leads, a broader team discussion, or management — do not let a CL stall indefinitely in limbo.

---

## 2. What to Look For in a Code Review

Review **every line** you are assigned. If you cannot understand code, that is itself a signal worth raising — other readers will face the same confusion. Cover all of the following dimensions:

- **Design** — Does the change belong in the codebase? Is the approach architecturally sound and well-integrated with the rest of the system?
- **Functionality** — Does the code do what the author intends? Does it serve users (humans and other callers) correctly, including edge cases?
- **Complexity** — Is any part of the code more complex than necessary? Flag over-engineering for speculative future needs ("YAGNI"). Complex code is harder to test, maintain, and reason about.
- **Tests** — Are there appropriate unit, integration, or end-to-end tests? Are tests themselves correct, maintainable, and meaningful? A broken test that passes unconditionally is worse than no test.
- **Naming** — Do names (variables, functions, types, files) clearly communicate intent without being excessively long? Poor names hide intent.
- **Comments** — Do comments explain *why*, not just *what*? Code that needs a comment explaining what it does is often code that should be rewritten for clarity instead.
- **Style & consistency** — Does the code follow the project's style guide and remain consistent with the surrounding code? Consistency lowers cognitive load for future readers.
- **Documentation** — When behavior changes, are READMEs, API docs, and inline doc-comments updated to match?
- **Context** — Read the surrounding code to understand how this CL fits the bigger picture. A locally correct change can still be wrong in context.
- **Parallel/concurrent code** — Carefully assess race conditions, deadlocks, and visibility guarantees. These bugs are invisible to basic testing and catastrophic in production.
- **Good things** — Acknowledge and commend good practices. Positive reinforcement is a real part of mentorship and makes the review relationship collaborative.

---

## 3. Navigating a CL in Review

Approach a multi-file CL in three passes — broad to narrow:

1. **Start with the CL description.** Does this change make sense? Is it necessary? If the answer is no, say so immediately, with courtesy and a constructive alternative. Do not silently review a CL that should not exist.
2. **Identify and review the most important files first.** Find the core logic change — usually the most complex or highest-risk files. Raise critical design issues *before* spending time on minor files. If fundamental redesign is needed, flag it early so the author is not building more on a broken foundation.
3. **Complete the review in a logical order.** Review test files before implementation files when useful — tests document intended behavior and provide context. Finish all remaining files to ensure nothing was missed.

The guiding principle: surface the most consequential feedback first to prevent wasted effort on secondary details.

---

## 4. Speed of Code Reviews

**Optimize for team velocity, not individual schedule.** Slow reviews are one of the most common sources of developer frustration and a leading cause of quality degradation (pressure to approve just to unblock).

- Respond to a review request **within one business day** at minimum. Aim for "shortly after it comes in" during normal working hours.
- The key metric is **response latency per round**, not total review duration. Quick, iterative responses feel faster than one slow monolithic review.
- Do not interrupt deep focus to review; instead, respond at natural breakpoints — after finishing a task, returning from a meeting, or between context switches.
- **LGTM with comments:** You may approve while leaving non-blocking comments when you trust the author will address them and the items are truly optional or minor.
- **Large CLs:** Do not try to review a 1,000-line CL as-is. Request the author split it into smaller, sequential CLs. Reviewing unmergeable large CLs is wasted effort for everyone.
- **Cross-timezone teams:** Aim to respond before the author's workday ends, or complete the review before their next day begins, to avoid one-day-per-round delays compounding into weeks.

---

## 5. How to Write Code Review Comments

**Tone and framing matter as much as content.** A technically correct comment delivered harshly creates defensiveness and slows collaboration.

- Be **kind and respectful** in all comments. Reviews are a long-term professional relationship.
- Always **explain the why** behind a request. "Move this function" is less useful than "Move this function — it belongs in the utility module so callers don't depend on this package for a single helper."
- **Criticize the code, not the author.** Describe what the code does and its implications; avoid framing that questions the author's judgment or intelligence.
- **Mix direction with problem-identification.** Sometimes point out a problem and let the author solve it — this builds skill and can produce better solutions than a prescriptive fix.
- **Label comment severity clearly:**
  - `Nit:` — minor, non-blocking, feel free to skip.
  - `Optional:` — suggestion worth considering but not required.
  - `FYI:` — informational only, no action expected.
  - Unlabeled / strong language = blocking concern.
- When a developer asks you to explain code they wrote, prefer asking them to rewrite it for clarity rather than adding an explanatory comment. Review discussion disappears; clear code benefits every future reader permanently.

---

## 6. Handling Pushback in Code Reviews

Pushback is normal. Handle it with intellectual honesty and persistence grounded in principle.

- **First, consider whether the developer is right.** They are often closer to the code and may have valid context you lack. When they are right, acknowledge it, update your view, and move on — this is not weakness, it is good engineering judgment.
- **When you still believe your point is correct, persist — but with explanation, not authority.** Lay out the facts, the principles, and the concrete consequences. Repeat and clarify if needed; developers sometimes need to understand the reasoning before accepting a change.
- **"I'll fix it later" is usually a trap.** Unless cleanup happens immediately after the current CL, it typically never happens. Insist on addressing new complexity before it merges; a polite but firm position here protects long-term code health.
- **Complaints about strictness are a transition symptom.** When standards tighten, early pushback is common. Speeding up review turnaround helps complaints subside. Developers generally come to value high standards once they experience their benefits.
- **If consensus cannot be reached**, escalate per the standard (tech lead, broader team, manager) — do not let disputes stall indefinitely or silently degrade into approving whatever the author wants.
