# X03: Contract Assault

## Context
Phase: **ATTACK** (1 of 4)
Purpose: Add contracts designed to expose bugs

## Mindset

**Contracts are weapons. Deploy them to break things.**

Standard contracts document expected behavior. Assault contracts are TIGHT—they express what SHOULD be true, not just what IS true. When they fail, you've found a bug.

## Input Required
- Vulnerability scan from X02
- Source files to modify
- Understanding of intended behavior

## Prompt

```
TASK: Add aggressive contracts designed to catch bugs.

INPUT:
{vulnerability_scan_from_X02}
{source_files}

GOAL: Make the code fail with valid inputs. Each contract failure = bug found.

═══════════════════════════════════════════════════════════════
PRECONDITION ASSAULT
Add strict preconditions that document true requirements
═══════════════════════════════════════════════════════════════

For each feature accepting parameters, add TIGHT preconditions:

BEFORE:
  feature_name (a_param: STRING)
    do
      -- uses a_param.count

AFTER:
  feature_name (a_param: STRING)
    require
      param_not_empty: not a_param.is_empty  -- ADD THIS
    do
      -- uses a_param.count

ASSAULT PRECONDITION ADDED: {class}.{feature}
  Contract: {the precondition}
  Purpose: Document true requirement
  Expected outcome: [PASS | FAIL]
  If fails: {what bug this reveals}

═══════════════════════════════════════════════════════════════
POSTCONDITION ASSAULT
Add strict postconditions that verify results
═══════════════════════════════════════════════════════════════

For each feature returning results or modifying state, add TIGHT postconditions:

BEFORE:
  compute_something: INTEGER
    do
      Result := calculation

AFTER:
  compute_something: INTEGER
    do
      Result := calculation
    ensure
      result_positive: Result >= 0  -- ADD THIS if it should be
      result_bounded: Result <= Maximum  -- ADD THIS if bounded

ASSAULT POSTCONDITION ADDED: {class}.{feature}
  Contract: {the postcondition}
  Purpose: Verify expected outcome
  Expected outcome: [PASS | FAIL]
  If fails: {what bug this reveals}

═══════════════════════════════════════════════════════════════
INVARIANT ASSAULT
Add strict invariants that catch state corruption
═══════════════════════════════════════════════════════════════

For classes with mutable state, add TIGHT invariants:

ASSAULT INVARIANT ADDED: {class}
  Contract: {the invariant}
  Purpose: Prevent state corruption
  Expected outcome: [PASS | FAIL]
  If fails: {what inconsistent state this catches}

═══════════════════════════════════════════════════════════════
OLD-VALUE ASSAULT
Use `old` to catch state corruption
═══════════════════════════════════════════════════════════════

For features that should NOT change certain state:

BEFORE:
  modify_x (a_value: INTEGER)
    do
      x := a_value

AFTER:
  modify_x (a_value: INTEGER)
    do
      x := a_value
    ensure
      y_unchanged: y = old y  -- ADD: verify unrelated state unchanged
      z_unchanged: z = old z

ASSAULT OLD-CHECK ADDED: {class}.{feature}
  Contract: {the old-value check}
  Purpose: Detect unintended side effects
  Expected outcome: [PASS | FAIL]
  If fails: {what side effect this catches}

═══════════════════════════════════════════════════════════════
RELATIONSHIP ASSAULT
Add contracts that verify object relationships
═══════════════════════════════════════════════════════════════

For objects that reference each other:

ASSAULT RELATIONSHIP: {class}
  Contract: parent.child = Current implies child.parent = Current
  Purpose: Verify bidirectional consistency
  Expected outcome: [PASS | FAIL]

═══════════════════════════════════════════════════════════════
RANGE ASSAULT
Add contracts that verify value ranges
═══════════════════════════════════════════════════════════════

For numeric values:

ASSAULT RANGE: {class}.{attribute}
  Contract: attribute >= Minimum and attribute <= Maximum
  Purpose: Catch out-of-range values
  Expected outcome: [PASS | FAIL]

═══════════════════════════════════════════════════════════════
MODEL-BASED ASSAULT (HIGHEST IMPACT)
Use MML models for complete postconditions
═══════════════════════════════════════════════════════════════

If class has no model query, ADD ONE FIRST:

ASSAULT: Add model query to {CLASS}
  ```eiffel
  feature -- Model (specification layer)
      model: MML_SEQUENCE [G]
          -- Abstract mathematical model of container state
          do
              create Result.make_from_list (internal_storage.twin)
          end
  ```

Then assault with model-based postconditions:

ASSAULT: {CLASS}.{feature} model postcondition
  BEFORE: ensure count = old count + 1
  AFTER:  ensure model |=| old model & x

  Why stronger: Model postcondition specifies COMPLETE behavior,
                not just one property. If implementation is wrong
                in ANY way, model equality fails.

MODEL ASSAULT CATEGORIES:

1. EXACT TRANSFORMATION (strongest)
   ```eiffel
   extend (x: G)
       ensure
           model_extended: model |=| old model & x
           -- Specifies exactly how model changes
   ```

2. CONTAINMENT
   ```eiffel
   merge (other: like Current)
       ensure
           model_superset: model >= old model
           -- New model contains old model
   ```

3. CARDINALITY + CONTENTS
   ```eiffel
   add_unique (x: G)
       ensure
           model_has: model.has (x)
           model_bounded: model.count <= old model.count + 1
   ```

4. FRAME CONDITIONS (model-based)
   ```eiffel
   update_item (i: INTEGER; x: G)
       ensure
           other_unchanged: model.but_item (i) |=| old model.but_item (i)
           -- Nothing else changed
   ```

ASSAULT MODEL RESULT: {class}.{feature}
  Model query added: [YES | NO | ALREADY_EXISTS]
  Primitive → Model upgrade: {old_contract} → {new_contract}
  Expected outcome: [PASS | FAIL]
  If fails: Model reveals implementation is WRONG

═══════════════════════════════════════════════════════════════
SEMANTIC ASSAULT
Add contracts that verify semantic meaning
═══════════════════════════════════════════════════════════════

Go beyond syntax to semantics:

BEFORE:
  diff_strings (a, b: STRING): DIFF_RESULT
    do
      -- compute diff

AFTER:
  diff_strings (a, b: STRING): DIFF_RESULT
    do
      -- compute diff
    ensure
      identical_if_same: a.same_string (b) implies Result.is_identical
      different_if_different: not a.same_string (b) implies Result.has_changes

ASSAULT SEMANTIC: {class}.{feature}
  Contract: {semantic postcondition}
  Purpose: Verify meaningful correctness
  Expected outcome: [PASS | FAIL]

═══════════════════════════════════════════════════════════════
COMPILE AND RUN
See what breaks
═══════════════════════════════════════════════════════════════

After adding assault contracts:

1. COMPILE the code
2. RUN existing tests
3. RECORD failures

ASSAULT RESULTS:

Contracts added: {count}
  - Preconditions: {count}
  - Postconditions: {count}
  - Invariants: {count}

Compilation result: [SUCCESS | FAILURE]
  If failure: {what contract is syntactically wrong}

Test run result:
  Tests passed: {count}
  Tests failed: {count}

CONTRACT FAILURES FOUND:
  1. {class}.{feature}: {contract} FAILED
     Input: {what triggered it}
     BUG REVEALED: {what bug this exposed}

  2. ...

OUTPUT FORMAT:

# CONTRACT ASSAULT REPORT: {library_name}

## Assault Summary
- Contracts deployed: {count}
- Contract failures: {count}
- Bugs revealed: {count}

## Contracts Added

### Preconditions
| Class | Feature | Contract | Result |
|-------|---------|----------|--------|
| {class} | {feature} | {contract} | PASS/FAIL |

### Postconditions
| Class | Feature | Contract | Result |
|-------|---------|----------|--------|
| {class} | {feature} | {contract} | PASS/FAIL |

### Invariants
| Class | Contract | Result |
|-------|----------|--------|
| {class} | {contract} | PASS/FAIL |

## Bugs Exposed

### BUG-001: {title}
- Location: {class}.{feature}
- Contract that failed: {contract}
- Trigger: {what input caused failure}
- Root cause: {why this is a bug}

### BUG-002: ...

## Contracts That Passed
{List of contracts that didn't find bugs—leave them as hardening}

## Next Attacks
{What to try in adversarial tests phase}
```

## Implementation Notes

When adding contracts:
1. Add them incrementally (a few at a time)
2. Compile after each batch
3. Run tests after each batch
4. Record every failure immediately
5. Don't fix bugs yet—just document them

## Success Criteria
- Multiple assault contracts added
- At least some contracts failed (found bugs)
- All failures documented with root cause
- Bugs cataloged for later fixing

## Next Step
→ X04-ADVERSARIAL-TESTS.md
