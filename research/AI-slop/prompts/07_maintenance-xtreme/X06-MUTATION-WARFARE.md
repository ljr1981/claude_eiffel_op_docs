# X06: Mutation Warfare

## Context
Phase: **ATTACK** (4 of 4)
Purpose: Mutate the code to test test quality and contract strength

## Mindset

**If I change the code, do the tests notice?**

Mutation testing deliberately introduces bugs. If tests still pass, either:
1. The tests are weak (don't catch the bug)
2. The contracts are weak (don't catch the bug)
3. The mutation is semantically equivalent (rare)

Every surviving mutation is a weakness in your defenses.

## Input Required
- Source code to mutate
- Test suite
- Contract coverage information from X03

## Prompt

```
TASK: Mutate the code and verify tests/contracts catch the mutations.

INPUT:
{source_files}
{test_suite}
{contract_coverage}

GOAL: Find weaknesses in tests and contracts by introducing controlled bugs.

═══════════════════════════════════════════════════════════════
MUTATION OPERATORS
Types of code changes to make
═══════════════════════════════════════════════════════════════

ARITHMETIC MUTATIONS:
  + → -       (addition to subtraction)
  - → +       (subtraction to addition)
  * → /       (multiplication to division)
  / → *       (division to multiplication)
  + 1 → - 1   (increment to decrement)

COMPARISON MUTATIONS:
  < → <=      (less than to less-or-equal)
  > → >=      (greater than to greater-or-equal)
  = → /=      (equal to not-equal)
  <= → <      (less-or-equal to less-than)
  >= → >      (greater-or-equal to greater-than)

BOOLEAN MUTATIONS:
  and → or
  or → and
  not X → X
  X → not X
  True → False
  False → True

BOUNDARY MUTATIONS:
  i < n → i <= n
  i <= n → i < n
  i > 0 → i >= 0
  i >= 1 → i > 1

RETURN VALUE MUTATIONS:
  Result := X → Result := Void (if detachable)
  Result := X → Result := Default
  Return early (add premature return)

DELETION MUTATIONS:
  Delete a statement
  Delete a branch
  Delete an assignment

═══════════════════════════════════════════════════════════════
MUTATION PROCEDURE
How to apply and test mutations
═══════════════════════════════════════════════════════════════

For each mutation:

1. IDENTIFY target: {class}.{feature} line {N}
2. ORIGINAL code: {original expression}
3. MUTATED code: {mutated expression}
4. COMPILE: Does it compile? [YES | NO]
5. RUN TESTS: Do tests catch it? [KILLED | SURVIVED]
6. CHECK CONTRACTS: Do contracts catch it? [KILLED | SURVIVED]
7. REVERT mutation

═══════════════════════════════════════════════════════════════
ARITHMETIC MUTATION CAMPAIGN
═══════════════════════════════════════════════════════════════

Find all arithmetic operations and mutate:

MUTATION: {id}
  Location: {class}.{feature}:{line}
  Original: a + b
  Mutated: a - b
  Compiles: YES
  Tests: SURVIVED ← WEAKNESS!
  Contracts: KILLED
  Weakness: Tests don't verify addition result

MUTATION RESULTS - ARITHMETIC:
  Mutations applied: {count}
  Killed by tests: {count}
  Killed by contracts: {count}
  Survived: {count} ← These are WEAKNESSES

═══════════════════════════════════════════════════════════════
COMPARISON MUTATION CAMPAIGN
═══════════════════════════════════════════════════════════════

Find all comparisons and mutate:

MUTATION: {id}
  Location: {class}.{feature}:{line}
  Original: i < count
  Mutated: i <= count
  Result: {KILLED | SURVIVED}

MUTATION RESULTS - COMPARISON:
  Mutations applied: {count}
  Killed: {count}
  Survived: {count}

═══════════════════════════════════════════════════════════════
BOOLEAN MUTATION CAMPAIGN
═══════════════════════════════════════════════════════════════

Find all boolean operations and mutate:

MUTATION RESULTS - BOOLEAN:
  Mutations applied: {count}
  Killed: {count}
  Survived: {count}

═══════════════════════════════════════════════════════════════
BOUNDARY MUTATION CAMPAIGN
═══════════════════════════════════════════════════════════════

Focus on loop boundaries and array indices:

MUTATION RESULTS - BOUNDARY:
  Mutations applied: {count}
  Killed: {count}
  Survived: {count}

═══════════════════════════════════════════════════════════════
RETURN VALUE MUTATION CAMPAIGN
═══════════════════════════════════════════════════════════════

Mutate return values:

MUTATION RESULTS - RETURNS:
  Mutations applied: {count}
  Killed: {count}
  Survived: {count}

═══════════════════════════════════════════════════════════════
DELETION MUTATION CAMPAIGN
═══════════════════════════════════════════════════════════════

Delete statements and branches:

MUTATION RESULTS - DELETION:
  Mutations applied: {count}
  Killed: {count}
  Survived: {count}

═══════════════════════════════════════════════════════════════
SURVIVING MUTATION ANALYSIS
Why did these survive?
═══════════════════════════════════════════════════════════════

For each survived mutation:

SURVIVOR: {mutation_id}
  Location: {class}.{feature}:{line}
  Mutation: {what was changed}
  Why survived:
    [ ] Equivalent mutation (semantically same)
    [ ] Missing test for this code path
    [ ] Missing postcondition
    [ ] Test doesn't verify this aspect
    [ ] Weak assertion in test

  Recommended fix:
    [ ] Add test: test_{what}
    [ ] Add contract: ensure {what}
    [ ] Strengthen existing test

═══════════════════════════════════════════════════════════════
MUTATION SCORE
Overall test/contract quality
═══════════════════════════════════════════════════════════════

MUTATION SCORE = Killed / (Total - Equivalent)

Category breakdown:
  Arithmetic: {X}%
  Comparison: {X}%
  Boolean: {X}%
  Boundary: {X}%
  Returns: {X}%
  Deletion: {X}%

OVERALL: {X}%

Interpretation:
  90-100%: Excellent test/contract coverage
  70-89%: Good, but has gaps
  50-69%: Moderate, needs improvement
  <50%: Weak, many gaps

OUTPUT FORMAT:

# MUTATION WARFARE REPORT: {library_name}

## Mutation Summary
- Total mutations: {count}
- Killed by tests: {count}
- Killed by contracts: {count}
- Survived: {count}
- Mutation score: {percent}%

## Results by Category

| Category | Mutations | Killed | Survived | Score |
|----------|-----------|--------|----------|-------|
| Arithmetic | {n} | {n} | {n} | {%} |
| Comparison | {n} | {n} | {n} | {%} |
| Boolean | {n} | {n} | {n} | {%} |
| Boundary | {n} | {n} | {n} | {%} |
| Returns | {n} | {n} | {n} | {%} |
| Deletion | {n} | {n} | {n} | {%} |

## Surviving Mutations (WEAKNESSES)

### SURVIVOR-001
- Location: {class}.{feature}:{line}
- Mutation: {+ → -}
- Reason survived: No test verifies the sum
- Fix: Add test_sum_correctness

### SURVIVOR-002
- ...

## Recommended Improvements

### Tests to Add
1. test_{what}: Catch {mutation type} in {feature}
2. ...

### Contracts to Add
1. {class}.{feature}: ensure {postcondition}
2. ...

## Kill Map
{Visual showing which mutations each test kills}

## Conclusions
- Weakest area: {category with lowest score}
- Strongest area: {category with highest score}
- Critical gaps: {most concerning survivors}
```

## Implementation Notes

1. Make ONE mutation at a time
2. Compile and test after each
3. REVERT before next mutation
4. Track which tests/contracts kill each mutation
5. Don't fix code yet—just identify weaknesses

## Success Criteria
- At least 50 mutations attempted
- Mutation score calculated
- Surviving mutations analyzed
- Recommended improvements documented

## Next Step
→ X07-TRIAGE-FINDINGS.md (Fix Phase begins)
