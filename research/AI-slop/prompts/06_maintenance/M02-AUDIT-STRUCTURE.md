# M02: Audit Contracts

## Context
Phase: **AUDIT** (1 of 3)
Scenario: Pre-existing codebase, improvement goal

## Input Required
- Eiffel source files
- Specification audit from M01
- Problem domain description

## Prompt

```
TASK: Audit contract coverage and correctness.

INPUT:
{eiffel_source_code}
{specification_audit}
{problem_domain}

═══════════════════════════════════════════════════════════════
CONTRACT AUDIT
Contracts = machine-checkable intent (require, ensure, invariant)
═══════════════════════════════════════════════════════════════

FOR EACH CLASS:

1. INVARIANT AUDIT
   [ ] Has invariant clause?
   [ ] Invariant captures class-level truths?
   [ ] Invariant aligns with specification?
   [ ] Invariant aligns with domain rules?

2. PRECONDITION AUDIT (per feature)
   [ ] Has require clause?
   [ ] Precondition validates arguments?
   [ ] Precondition validates state?
   [ ] Precondition aligns with spec?
   [ ] Precondition aligns with domain?

3. POSTCONDITION AUDIT (per feature)
   [ ] Has ensure clause?
   [ ] Postcondition verifies result? (queries)
   [ ] Postcondition verifies state change? (commands)
   [ ] Postcondition uses 'old' for deltas?
   [ ] Postcondition aligns with spec?
   [ ] Postcondition aligns with domain?

═══════════════════════════════════════════════════════════════
CORRECTNESS ASSESSMENT
Not just presence, but semantic correctness
═══════════════════════════════════════════════════════════════

ABSENT CONTRACTS:
- Features with no require
- Features with no ensure
- Classes with no invariant

MALFORMED CONTRACTS:
- Precondition doesn't match feature purpose
- Postcondition doesn't guarantee spec'd behavior
- Invariant contradicts class semantics
- Contract is tautology (always true, meaningless)
- Contract is impossible (always false)
- Contract references wrong attributes
- Contract contradicts specification
- Contract contradicts domain rules

OUTPUT FORMAT:
CLASS: {class_name}

INVARIANT:
  Status: [ABSENT | PRESENT | MALFORMED]
  If MALFORMED:
    Current: {existing invariant}
    Problem: {why it's wrong}
    Should be: {correct invariant}

FEATURE: {feature_name}
  Precondition: [ABSENT | WEAK | MALFORMED | ADEQUATE]
  If issue: {description}

  Postcondition: [ABSENT | WEAK | MALFORMED | ADEQUATE]
  If issue: {description}

CONTRACT SCORE: {X}/10
  - {Y}% features have preconditions
  - {Z}% features have postconditions
  - Invariant: [YES | NO]

ALIGNMENT WITH SPEC:
  Contracts that contradict spec: {list}
  Contracts weaker than spec requires: {list}

PRIORITY FIXES:
1. {most critical contract issue}
2. {next priority}
```

## Success Criteria
- Every class checked for invariant
- Every feature checked for pre/post
- Correctness assessed, not just presence
- Alignment with spec verified

## Next Step
→ M03-AUDIT-CODE.md
