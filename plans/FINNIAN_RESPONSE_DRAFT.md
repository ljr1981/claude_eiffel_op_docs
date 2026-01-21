# Response to Finnian Reilly

**Date:** 2026-01-20

---

## Final Response

Dear Finnian,

Thank you for your candid response. I want to clarify what we actually did, as there appears to be a misunderstanding.

**We did not fork Eiffel-Loop.** We created new, independent libraries.

What we did:
1. Studied the current master branch (commit `db7b4cffc`, Nov 18, 2025) to understand your architectural patterns
2. Built entirely **new libraries** with original code, inspired by concepts we learned
3. Published these as independent repositories under the `simple-eiffel` organization

The simple_* libraries contain no EL_* classes. For example, SIMPLE_ZSTRING is inspired by the dual-storage concept of EL_ZSTRING, but the implementation, contracts, and tests are original work.

**On your specific points:**
- **Gobo:** You're right that Gobo dependencies were removed. The `base.ecf` we analyzed has only ISE dependencies. Noted in our documentation.
- **"Old version":** We used current master — the 479 commits since January 2024 demonstrate your active development.

**Our motivation:** We wanted to learn from Eiffel-Loop's patterns while targeting EiffelStudio 25.02 with void safety from day one. Rather than attempt to modify your codebase, we studied the concepts and built new implementations targeting `void_safety = "conformance"` or higher.

**Our process:** We apply a systematic 7-step research methodology to all code we study:

1. **Scope** — Define the problem space and boundaries
2. **Landscape** — Survey existing solutions and approaches
3. **Requirements** — Extract functional and non-functional needs
4. **Decisions** — Make explicit design choices with rationale
5. **Innovations** — Identify novel approaches and improvements
6. **Risks** — Analyze what could go wrong and mitigations
7. **Recommendation** — Synthesize findings into actionable specs

We apply this process uniformly — to ISE EiffelBase, to Gobo, to Eiffel-Loop, and to our own simple_* code. We eat our own dogfood. This isn't casual borrowing; it's disciplined study that produces original, well-considered designs with full traceability to their inspirations.

**To be clear:** We do not claim to represent Eiffel-Loop, claim compatibility, claim feature parity, or seek your endorsement. These are independent libraries that happen to be inspired by your work.

**Going forward:** We will continue studying EL libraries beyond Base — reflection, text processing, persistence — and building specifications inspired by those patterns. This is not about replacing Eiffel-Loop. It's about creating another option for Eiffel developers: something to learn from, borrow from, build upon. More options strengthen the Eiffel ecosystem.

Your 20+ years of work on Eiffel-Loop is a significant contribution to the community. The architectural patterns — ZSTRING, factories, reflection — are well-designed, which is precisely why they're worth studying.

If any wording suggested we were forking or modifying Eiffel-Loop, I apologize. That was not our intent.

Respectfully,
Larry

---

## Key Points Summary

| Concern | Clarification |
|---------|---------------|
| "Forking old version" | Not a fork. New libraries. Current master was studied. |
| "Gobo removed" | Confirmed. base.ecf has no Gobo dependencies. |
| "Missing features/fixes" | We don't claim parity. These are new implementations. |
| "Do not endorse" | Understood. No endorsement claimed or implied. |

---

## Tone Notes

- Respectful, not defensive
- Factual, with specific evidence
- Acknowledges Finnian's expertise and work
- Clear about boundaries (no endorsement claimed)
- Brief and focused

