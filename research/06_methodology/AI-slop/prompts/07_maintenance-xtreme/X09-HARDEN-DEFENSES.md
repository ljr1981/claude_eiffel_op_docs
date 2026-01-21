# X09: Harden Defenses

## Context
Phase: **FIX** (3 of 4)
Purpose: Add permanent protective contracts and tests

## Mindset

**Never fix the same bug twice.**

After fixing bugs, add defenses so they can't recur:
- Contracts that catch the condition
- Tests that verify the behavior
- Documentation that warns users

The attack phase found weaknesses. Now we fortify.

## Input Required
- Surgical fix report from X08
- Mutation warfare report (surviving mutations)
- All source files

## Prompt

```
TASK: Add defensive contracts and tests to prevent bug recurrence.

INPUT:
{surgical_fix_report_from_X08}
{mutation_warfare_report_X06}
{source_files}

GOAL: For every bug fixed, add protection to prevent recurrence.

═══════════════════════════════════════════════════════════════
DEFENSIVE CONTRACT STRATEGY
Convert bug fixes into permanent contracts
═══════════════════════════════════════════════════════════════

For each bug that was fixed:

BUG: {finding-id}
  What went wrong: {description}
  How fixed: {the fix}

DEFENSIVE CONTRACT:
  Type: [PRECONDITION | POSTCONDITION | INVARIANT]
  Location: {class}.{feature}
  Contract:
```eiffel
require
  {defensive_precondition}: {expression}
```
  Purpose: Prevent {what condition}

═══════════════════════════════════════════════════════════════
PRECONDITION HARDENING
Add preconditions for all input assumptions
═══════════════════════════════════════════════════════════════

For features that had bugs due to invalid input:

BEFORE:
```eiffel
process_input (a_input: STRING)
  do
    -- crashed when a_input was empty
```

AFTER:
```eiffel
process_input (a_input: STRING)
  require
    input_not_empty: not a_input.is_empty
  do
    -- now protected
```

PRECONDITION ADDED: {class}.{feature}
  Contract: {expression}
  Protects against: {what bug}

═══════════════════════════════════════════════════════════════
POSTCONDITION HARDENING
Add postconditions for all output guarantees
═══════════════════════════════════════════════════════════════

For features that had wrong-result bugs:

BEFORE:
```eiffel
compute_result: INTEGER
  do
    Result := calculation
    -- sometimes returned negative when it shouldn't
```

AFTER:
```eiffel
compute_result: INTEGER
  do
    Result := calculation
  ensure
    result_non_negative: Result >= 0
```

POSTCONDITION ADDED: {class}.{feature}
  Contract: {expression}
  Guarantees: {what property}

═══════════════════════════════════════════════════════════════
INVARIANT HARDENING
Add invariants for all state consistency
═══════════════════════════════════════════════════════════════

For classes that had state corruption bugs:

BEFORE:
```eiffel
class STATE_HOLDER
  -- count and items could get out of sync
```

AFTER:
```eiffel
class STATE_HOLDER
invariant
  count_consistent: count = items.count
```

INVARIANT ADDED: {class}
  Contract: {expression}
  Maintains: {what consistency}

═══════════════════════════════════════════════════════════════
REGRESSION TEST HARDENING
Add tests for every fixed bug
═══════════════════════════════════════════════════════════════

For each bug fixed, add a test that:
1. Creates the condition that caused the bug
2. Verifies the bug no longer occurs

REGRESSION TEST: test_bug_{finding-id}
```eiffel
test_bug_{finding_id}
    -- Regression test for {finding-id}: {description}
  local
    l_obj: TARGET_CLASS
  do
    create l_obj.make
    -- Setup condition that triggered bug
    {setup_code}
    -- Verify bug doesn't occur
    {verification_code}
    assert ("no_crash", True)
    assert ("correct_result", {expected_condition})
  end
```

REGRESSION TEST ADDED: test_bug_{id}
  Tests: {what bug}
  Verifies: {what condition}

═══════════════════════════════════════════════════════════════
MUTATION SURVIVOR HARDENING
Add defenses for surviving mutations
═══════════════════════════════════════════════════════════════

From X06, mutations that survived indicate weak defenses:

SURVIVOR: {mutation-id}
  Mutation: {what was changed}
  Survived because: {why}

HARDENING:
  Add test: {test that would kill this mutation}
  Add contract: {contract that would catch it}

MUTATION DEFENSE ADDED:
  Type: [TEST | CONTRACT]
  Kills mutation: {mutation-id}

═══════════════════════════════════════════════════════════════
BOUNDARY HARDENING
Document and enforce limits
═══════════════════════════════════════════════════════════════

From X05 stress testing, document discovered limits:

LIMIT: {feature}
  Maximum: {value}
  Discovered in: Stress testing

HARDENING:
  - Add precondition: {contract}
  - Document in class note: {documentation}

BOUNDARY DEFENSE ADDED: {class}.{feature}
  Enforces: {what limit}

═══════════════════════════════════════════════════════════════
DEFENSE VERIFICATION
Verify all defenses work
═══════════════════════════════════════════════════════════════

For each defense added:

1. Compile with assertions enabled
2. Run test suite
3. Verify defense is active
4. Verify defense catches the original bug condition

DEFENSE VERIFICATION:
  Defense: {contract/test}
  Active: [YES | NO]
  Catches original bug: [YES | NO]

═══════════════════════════════════════════════════════════════
DEFENSE INVENTORY
Summary of all hardening
═══════════════════════════════════════════════════════════════

HARDENING SUMMARY:

Preconditions added: {count}
Postconditions added: {count}
Invariants added: {count}
Regression tests added: {count}
Limits documented: {count}

OUTPUT FORMAT:

# HARDENING REPORT: {library_name}

## Hardening Summary
- Defenses added: {total}
- Preconditions: {count}
- Postconditions: {count}
- Invariants: {count}
- Regression tests: {count}

## Contract Hardening

### Preconditions Added
| Class | Feature | Contract | Protects Against |
|-------|---------|----------|------------------|
| {class} | {feature} | {contract} | {bug} |

### Postconditions Added
| Class | Feature | Contract | Guarantees |
|-------|---------|----------|------------|
| {class} | {feature} | {contract} | {property} |

### Invariants Added
| Class | Contract | Maintains |
|-------|----------|-----------|
| {class} | {contract} | {consistency} |

## Test Hardening

### Regression Tests Added
| Test | Bug ID | Verifies |
|------|--------|----------|
| test_bug_001 | BUG-001 | {what} |

### Mutation Killers Added
| Test/Contract | Kills Mutation |
|--------------|----------------|
| {defense} | {mutation-id} |

## Limit Documentation
| Feature | Limit | Enforcement |
|---------|-------|-------------|
| {feature} | {limit} | {how enforced} |

## Verification Results
- All contracts compile: [YES | NO]
- All tests pass: [YES | NO]
- All defenses active: [YES | NO]

## Defense Coverage
- Bugs with regression tests: {count}/{total}
- Bugs with protective contracts: {count}/{total}
- Surviving mutations now killed: {count}/{total}
```

## Implementation Notes

1. Add defenses incrementally
2. Verify each defense works
3. Prefer contracts over tests (runtime protection)
4. Document all limits
5. Every fixed bug should have at least one defense

## Success Criteria
- Every fixed bug has a regression test
- Every fixed bug has a protective contract
- Surviving mutations have new defenses
- All limits are documented and enforced
- All defenses verified

## Next Step
→ X10-VERIFY-HARDENING.md
