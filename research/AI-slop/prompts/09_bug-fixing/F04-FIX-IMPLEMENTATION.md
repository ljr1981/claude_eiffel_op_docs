# F04: Fix Code

## Context
Phase: **FIX** (2 of 3)
Scenario: Bug's root cause is in CODE layer (code doesn't satisfy spec/contract/domain)

## Input Required
- Classification from F02 (confirms code layer)
- Specification (what it should do)
- Contract (existing or from F03)
- Source code
- Root cause
- Problem domain

## Prompt

```
TASK: Fix the code layer to satisfy spec, contracts, and domain requirements.

INPUT:
{root_cause}
{specification}
{contracts}
{eiffel_source_code}
{problem_domain}

═══════════════════════════════════════════════════════════════
CODE FIX REQUIREMENTS
Code must align with ALL higher layers
═══════════════════════════════════════════════════════════════

1. CODE MUST SATISFY CONTRACTS
   - Preconditions: Can ASSUME they're true
   - Postconditions: MUST make them true
   - Invariants: MUST maintain them

2. CODE MUST MATCH SPECIFICATION
   - Behavior matches what spec describes
   - Edge cases handled as spec requires
   - Errors handled as spec requires

3. CODE MUST RESPECT DOMAIN
   - Algorithm correct for domain
   - Domain rules enforced
   - Domain state transitions valid

4. MINIMAL CHANGE PRINCIPLE
   - Only change what's necessary
   - Don't refactor while fixing
   - Don't "improve" adjacent code
   - One bug, one fix

5. PRESERVE CORRECT BEHAVIOR
   - Correct behavior must not change
   - Only incorrect behavior changes
   - If unsure, add test first

FIX PATTERNS:

OFF-BY-ONE:
  Before: from i := 0 until i >= count
  After:  from i := 1 until i > count

MISSING CHECK:
  Before: value := items[index]
  After:  if valid_index (index) then value := items[index] end

WRONG CALCULATION:
  Before: Result := a + b
  After:  Result := a - b  -- Based on postcondition requirement

STATE UPDATE:
  Before: (missing update)
  After:  count := count + 1  -- Required by postcondition

OUTPUT FORMAT:
BUG: {summary}
ROOT CAUSE: {what was wrong}

BEFORE:
```eiffel
{buggy code - exact lines}
```

AFTER:
```eiffel
{fixed code - exact lines}
```

CHANGE EXPLANATION:
    What changed: {description}
    Why it's correct: {how it satisfies contracts}

═══════════════════════════════════════════════════════════════
LAYER ALIGNMENT VERIFICATION
═══════════════════════════════════════════════════════════════

CONTRACT VERIFICATION:
    Precondition assumed: {what we rely on}
    Postcondition achieved: {what we guarantee}
    Invariant maintained: {what stays true}

SPEC VERIFICATION:
    Spec says: {what spec requires}
    Code now does: {how code implements it}
    Alignment: [ALIGNED | explain]

DOMAIN VERIFICATION:
    Domain rule: {relevant domain rule}
    Code respects: [YES | explain]

SIDE EFFECTS:
    [NONE | {list any behavioral changes}]
```

## Success Criteria
- Implementation fixed
- Code aligns with spec
- Code aligns with domain
- Contracts satisfied
- Minimal change applied
- Side effects documented

## Next Step
→ F05-VERIFY-FIX.md
