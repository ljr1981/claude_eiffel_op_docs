# D06: Extract Abstractions

## Context
Phase: **REFACTORING** (2 of 4)
Purpose: Create new classes and refactor inheritance

## Mindset

**Create the missing abstractions. Fix the broken hierarchies.**

This is where the actual code changes happen:
- Create new classes identified in D05
- Extract features from God classes
- Replace inheritance with composition where needed
- Introduce proper deferred classes

## Input Required
- Refactoring plan from D05
- Source files to modify
- Test suite

## Prompt

```
TASK: Implement class extractions and hierarchy changes from the refactoring plan.

INPUT:
{refactoring_plan_from_D05}
{source_files}
{test_suite}

RULE: Make one refactoring at a time. Compile and test after each.

═══════════════════════════════════════════════════════════════
EXTRACT CLASS REFACTORING
Pull cohesive features into new class
═══════════════════════════════════════════════════════════════

For each Extract Class in the plan:

EXTRACTION: {id}
  Source class: {GOD_CLASS}
  Features to extract: {list}
  New class name: {EXTRACTED_CLASS}

STEP 1: Create new class file

```eiffel
note
  description: "Extracted from {GOD_CLASS} - {responsibility}"

class
  {EXTRACTED_CLASS}

create
  make

feature {NONE} -- Initialization

  make
    do
      -- Initialize
    end

feature -- Access

  {extracted_queries}

feature -- Operations

  {extracted_commands}

feature {NONE} -- Implementation

  {extracted_private_features}

end
```

STEP 2: Add attribute in source class

```eiffel
feature {NONE} -- Implementation

  {extracted}: {EXTRACTED_CLASS}
    -- Handles {responsibility}
```

STEP 3: Update source class features to delegate

BEFORE:
```eiffel
do_something
  do
    -- direct implementation
  end
```

AFTER:
```eiffel
do_something
  do
    {extracted}.do_something
  end
```

STEP 4: Compile and test
  - Compiles: [YES | NO]
  - Tests pass: [YES | NO]

═══════════════════════════════════════════════════════════════
INTRODUCE DEFERRED CLASS
Create abstract base class
═══════════════════════════════════════════════════════════════

For each deferred class to introduce:

DEFERRED CLASS: {id}
  Name: {DEFERRED_NAME}
  Children will be: {list of concrete classes}
  Deferred features: {list}

```eiffel
deferred class
  {DEFERRED_NAME}

feature -- Status

  {deferred_query}: {TYPE}
    deferred
    end

feature -- Operations

  {deferred_command}
    deferred
    end

  template_method
    -- Template using deferred features
    do
      {uses deferred features}
    end

end
```

UPDATE children to inherit:

```eiffel
class CONCRETE_A
inherit
  {DEFERRED_NAME}
    redefine
      {deferred_query}, {deferred_command}
    end
```

═══════════════════════════════════════════════════════════════
REPLACE INHERITANCE WITH COMPOSITION
Convert is-a to has-a
═══════════════════════════════════════════════════════════════

For each inheritance to replace:

CONVERSION: {id}
  Class: {CHILD}
  Remove parent: {PARENT}
  Add attribute: {parent}: {PARENT}

BEFORE:
```eiffel
class CHILD
inherit
  PARENT
    rename
      feature_a as parent_feature_a
    end
```

AFTER:
```eiffel
class CHILD
feature
  parent_component: PARENT
    -- Component for {functionality}

  parent_feature_a
    -- Delegate to component
    do
      Result := parent_component.feature_a
    end
```

DELEGATIONS NEEDED:
  - {feature}: delegate to {component}.{feature}
  - {feature}: delegate to {component}.{feature}

═══════════════════════════════════════════════════════════════
PULL UP FEATURES
Move features to parent class
═══════════════════════════════════════════════════════════════

For features duplicated in children:

PULL UP: {id}
  Features: {list}
  From: {child1}, {child2}
  To: {parent}

STEP 1: Add feature to parent (if not deferred)
```eiffel
class PARENT
feature
  {pulled_feature}
    do
      {common_implementation}
    end
```

STEP 2: Remove from children (or keep override if different)

STEP 3: Verify behavior unchanged

═══════════════════════════════════════════════════════════════
PUSH DOWN FEATURES
Move features to child classes
═══════════════════════════════════════════════════════════════

For features only used in specific children:

PUSH DOWN: {id}
  Feature: {feature_name}
  From: {parent}
  To: {specific_children}

STEP 1: Copy feature to children
STEP 2: Remove from parent
STEP 3: Update any callers

═══════════════════════════════════════════════════════════════
SPLIT CLASS
Divide class by responsibility
═══════════════════════════════════════════════════════════════

For classes with multiple responsibilities:

SPLIT: {id}
  Source: {ORIGINAL_CLASS}
  Into:
    - {NEW_CLASS_A}: {responsibility_A}
    - {NEW_CLASS_B}: {responsibility_B}

APPROACH:
  1. Create NEW_CLASS_A with responsibility_A features
  2. Create NEW_CLASS_B with responsibility_B features
  3. ORIGINAL_CLASS becomes facade/coordinator
  4. ORIGINAL_CLASS delegates to A and B

═══════════════════════════════════════════════════════════════
INTRODUCE PARAMETER OBJECT
Replace long parameter list with object
═══════════════════════════════════════════════════════════════

For features with too many parameters:

PARAMETER OBJECT: {id}
  Feature: {class}.{feature}
  Old params: {param1}, {param2}, {param3}, {param4}
  New class: {PARAM_OBJECT}

```eiffel
class {PARAM_OBJECT}
create
  make

feature {NONE} -- Initialization
  make (a_{param1}: T1; a_{param2}: T2; ...)
    do
      {param1} := a_{param1}
      ...
    end

feature -- Access
  {param1}: T1
  {param2}: T2
  ...
end
```

UPDATE feature:
```eiffel
{feature} (a_params: {PARAM_OBJECT})
  do
    -- Use a_params.{param1}, etc.
  end
```

═══════════════════════════════════════════════════════════════
REFACTORING LOG
Track what was done
═══════════════════════════════════════════════════════════════

For each refactoring completed:

COMPLETED: {id}
  Type: {EXTRACT | INTRODUCE_DEFERRED | COMPOSITION | etc}
  Files created: {list}
  Files modified: {list}
  Lines added: {count}
  Lines removed: {count}
  Compiles: [YES]
  Tests pass: [YES]

OUTPUT FORMAT:

# EXTRACTION REPORT: {library_name}

## Summary
- Refactorings applied: {count}
- New classes created: {count}
- Classes modified: {count}
- Lines added: {count}
- Lines removed: {count}

## New Classes Created

### {NEW_CLASS_1}
- Purpose: {why}
- Extracted from: {source}
- File: {path}
- Features: {count}

### {NEW_CLASS_2}
...

## Class Modifications

### {MODIFIED_CLASS_1}
- Changes:
  - {change 1}
  - {change 2}
- Inheritance: {old} → {new}

### {MODIFIED_CLASS_2}
...

## Refactoring Log
| ID | Type | Source | Target | Status |
|----|------|--------|--------|--------|
| R01 | Extract | GOD | A, B | DONE |

## Test Results
- Compile: PASS
- Tests: {pass}/{total}
- New tests added: {count}

## Files Changed
{List of all files created/modified}
```

## Implementation Notes

1. One refactoring at a time
2. Compile after each
3. Test after each
4. Revert if tests fail
5. Commit working checkpoints

## Success Criteria
- All planned extractions completed
- All new classes compile
- All tests pass
- No regressions

## Next Step
→ D07-APPLY-GENERICITY.md
