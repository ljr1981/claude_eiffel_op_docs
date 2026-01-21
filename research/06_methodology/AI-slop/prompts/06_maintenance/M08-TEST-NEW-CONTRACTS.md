# M08: Validate All Layers

## Context
Phase: **VALIDATE** (3 of 3)
Scenario: Pre-existing codebase, all layers strengthened

## Input Required
- Compiled code with new/fixed specs and contracts
- Test suite (existing and new)
- Runtime output/errors
- Specifications
- Problem domain

## Prompt

```
TASK: Validate all four layers are aligned and working correctly.

INPUT:
{test_output_or_runtime_errors}
{specifications}
{contracts_added_or_fixed}
{problem_domain}

═══════════════════════════════════════════════════════════════
LAYER ALIGNMENT VALIDATION
All four layers must be consistent
═══════════════════════════════════════════════════════════════

1. SPECIFICATION ↔ DOMAIN
   [ ] Specs correctly describe domain requirements?
   [ ] No spec contradicts domain truth?

2. CONTRACT ↔ SPECIFICATION
   [ ] Contracts implement what specs describe?
   [ ] No contract contradicts spec?

3. CODE ↔ CONTRACT
   [ ] Code satisfies all contracts?
   [ ] Contract violations indicate code bugs?

4. TEST ↔ SPECIFICATION
   [ ] Tests verify spec'd behavior?
   [ ] Test expected values from spec (not code)?

═══════════════════════════════════════════════════════════════
CONTRACT RUNTIME VALIDATION
Run code with contracts enabled
═══════════════════════════════════════════════════════════════

TASK: Run code with new contracts enabled to surface latent bugs.

INPUT:
{test_output_or_runtime_errors}
{new_contracts_added}

EXPECTED OUTCOMES:

1. ALL TESTS PASS
   Interpretation: Existing code already satisfied new contracts
   Action: Contracts correctly captured existing behavior ✓

2. PRECONDITION VIOLATIONS
   Interpretation: Callers were passing invalid data
   Question: Was the code "working" by accident?

   Decision tree:
   - If caller is WRONG → Fix caller
   - If precondition is TOO STRICT → Weaken precondition
   - If behavior was intentionally permissive → Document exception

3. POSTCONDITION VIOLATIONS
   Interpretation: Implementation doesn't achieve stated goal
   Question: Is the contract wrong or the code wrong?

   Decision tree:
   - If contract matches INTENT → Fix implementation (latent bug found!)
   - If contract is WRONG → Fix contract (over-specified)

4. INVARIANT VIOLATIONS
   Interpretation: Object enters invalid state
   This usually indicates a REAL BUG that was previously hidden

   Action: Fix the feature that violated invariant

OUTPUT FORMAT:
TEST RUN SUMMARY:
- Tests run: {N}
- Passed: {N}
- Failed due to contract: {N}
- Failed other: {N}

CONTRACT VIOLATIONS FOUND:

VIOLATION 1:
  Type: [PRECONDITION | POSTCONDITION | INVARIANT]
  Contract: {tag}: {expression}
  Feature: {class.feature}

  ANALYSIS:
  - Is contract correct? [YES | NO]
  - Is code correct? [YES | NO]

  VERDICT: [LATENT BUG FOUND | CONTRACT TOO STRICT | CONTRACT WRONG]

  ACTION: {specific fix}

LATENT BUGS DISCOVERED: {count}
```

## Success Criteria
- All violations analyzed
- Each categorized correctly
- Latent bugs identified as wins
- Appropriate fixes determined

## Completion
Contract violations from new contracts = **latent bugs surfaced**

This is the payoff: code that "worked" is revealed to have been broken. Fix the bugs, not the contracts (unless contracts are genuinely wrong).
