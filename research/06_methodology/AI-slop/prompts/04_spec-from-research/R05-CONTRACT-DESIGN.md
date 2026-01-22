# R05: Contract Design

## Context
Phase: **DESIGN** (2 of 3)
Purpose: Define all contracts (preconditions, postconditions, invariants)

## Mindset

**Contracts are the specification. Code merely implements them.**

In Design by Contract:
- Preconditions define valid input
- Postconditions guarantee valid output
- Invariants maintain valid state

Design contracts FIRST, before thinking about implementation.

## Input Required
- Class design from R04
- Domain rules from R02
- Requirements from R01/R03

## Prompt

```
TASK: Design complete contracts for all classes and features.

INPUT:
{class_design_from_R04}
{domain_rules_from_R02}
{requirements}

═══════════════════════════════════════════════════════════════
INVARIANT DESIGN
What must always be true for each class?
═══════════════════════════════════════════════════════════════

For each class:

CLASS: {CLASS_NAME}

INVARIANTS:
  From domain rules:
    {rule_name}: {expression}
      -- From DR-{id}: {rule description}

  From data integrity:
    {integrity_name}: {expression}
      -- {what consistency this maintains}

  From valid state:
    {state_name}: {expression}
      -- {what invalid state this prevents}

INVARIANT SPECIFICATION:
```eiffel
invariant
  {rule_name}: {expression}
  {integrity_name}: {expression}
  {state_name}: {expression}
```

═══════════════════════════════════════════════════════════════
CREATION CONTRACT DESIGN
What must be true when objects are created?
═══════════════════════════════════════════════════════════════

For each creation procedure:

CREATION: {CLASS}.make_{variant}

  PRECONDITIONS:
    From parameter validity:
      {param}_valid: {expression}
        -- {what this checks}

    From creation conditions:
      {condition}: {expression}
        -- {when creation is allowed}

  POSTCONDITIONS:
    From attribute initialization:
      {attr}_set: {attribute} = {expected}
        -- Verify {attribute} initialized

    From state establishment:
      {state}: {condition}
        -- Object is in {state}

CREATION CONTRACT:
```eiffel
make (a_param: TYPE)
  require
    {param}_valid: {expression}
  do
    {implementation}
  ensure
    {attr}_set: {attribute} = a_param
    {state}: {is_valid}
  end
```

═══════════════════════════════════════════════════════════════
QUERY CONTRACT DESIGN
What do queries guarantee?
═══════════════════════════════════════════════════════════════

For each query (function):

QUERY: {CLASS}.{query_name}: {RETURN_TYPE}

  PRECONDITIONS:
    When can this be called?
      {condition}: {expression}
        -- {why this is required}

  POSTCONDITIONS:
    What is guaranteed about Result?
      {result_property}: Result.{property}
        -- {what this guarantees}

    Relationship to state?
      {relationship}: Result = {derived_from_state}
        -- {how result relates to attributes}

QUERY CONTRACT:
```eiffel
{query_name}: {RETURN_TYPE}
  require
    {precondition}: {expression}
  do
    Result := {computation}
  ensure
    {postcondition}: {expression}
  end
```

═══════════════════════════════════════════════════════════════
COMMAND CONTRACT DESIGN
What do commands guarantee?
═══════════════════════════════════════════════════════════════

For each command (procedure with side effects):

COMMAND: {CLASS}.{command_name}

  PRECONDITIONS:
    When can this be called?
      {condition}: {expression}

  POSTCONDITIONS:
    What state changes?
      {modified_attr}: {attribute} = {new_value}

    What stays the same?
      {unchanged}: {attribute} = old {attribute}

    What relationship to input?
      {input_used}: {state} = f(old state, input)

COMMAND CONTRACT:
```eiffel
{command_name} (a_param: TYPE)
  require
    {precondition}: {expression}
  do
    {implementation}
  ensure
    {state_changed}: {attribute} = {expected}
    {unchanged}: other_attribute = old other_attribute
  end
```

═══════════════════════════════════════════════════════════════
FRAME SPECIFICATION
What CAN'T change?
═══════════════════════════════════════════════════════════════

For commands, specify what must NOT change:

FRAME: {CLASS}.{command}
  Modifies:
    - {attribute1}
    - {attribute2}

  Does NOT modify:
    - {attribute3}: {unchanged_name}: {attr3} = old {attr3}
    - {attribute4}: {unchanged_name}: {attr4} = old {attr4}

Use `old` to verify unchanged state.

═══════════════════════════════════════════════════════════════
CONTRACT STRENGTH ANALYSIS
Are contracts strong enough?
═══════════════════════════════════════════════════════════════

For each contract:

CONTRACT: {class}.{feature}

  Precondition strength:
    Too weak: Allows invalid inputs
    Too strong: Rejects valid inputs
    VERDICT: [TOO_WEAK | JUST_RIGHT | TOO_STRONG]

  Postcondition strength:
    Too weak: Doesn't guarantee enough
    Too strong: Over-constrains implementation
    VERDICT: [TOO_WEAK | JUST_RIGHT | TOO_STRONG]

ADJUSTMENTS:
  {contract}: strengthen by adding {clause}
  {contract}: weaken by removing {clause}

═══════════════════════════════════════════════════════════════
SEMANTIC POSTCONDITIONS
Postconditions that verify meaning
═══════════════════════════════════════════════════════════════

Beyond syntax, add semantic contracts:

SEMANTIC CONTRACT: {class}.{feature}

  Semantic property:
    {meaningful_property}: {expression}
      -- This verifies {what the feature means}

Examples:
```eiffel
diff_strings (a_source, a_target: STRING): DIFF_RESULT
  ensure
    -- Semantic: if inputs identical, result shows no changes
    identical_means_no_changes:
      a_source.same_string (a_target) implies Result.is_identical

    -- Semantic: applying result to source yields target
    result_correct:
      apply (Result, a_source).same_string (a_target)
```

═══════════════════════════════════════════════════════════════
DEFENSIVE CONTRACTS
Contracts that catch bugs
═══════════════════════════════════════════════════════════════

Add contracts specifically to catch likely bugs:

DEFENSIVE CONTRACT: {class}.{feature}

  Against null:
    {param}_attached: {param} /= Void (for detachable)

  Against empty:
    {param}_not_empty: not {param}.is_empty

  Against out of range:
    {index}_valid: {index} >= 1 and {index} <= count

  Against invalid state:
    {ready}: is_ready

═══════════════════════════════════════════════════════════════
CONTRACT COMPLETENESS CHECK
Are all features contracted?
═══════════════════════════════════════════════════════════════

CONTRACT COVERAGE:

| Class | Feature | Preconditions | Postconditions | Complete |
|-------|---------|---------------|----------------|----------|
| {class} | {feature} | YES | YES | ✓ |
| {class} | {feature} | YES | NO | ✗ |

MISSING CONTRACTS:
  {class}.{feature}: needs postcondition for {what}

COVERAGE: {percent}%

═══════════════════════════════════════════════════════════════
REQUIREMENT → CONTRACT TRACING
Every requirement has a contract
═══════════════════════════════════════════════════════════════

TRACEABILITY:

| Requirement | Verified By | Contract |
|-------------|-------------|----------|
| FR-001 | {class}.{feature} | ensure {clause} |
| NFR-001 | {class} | invariant {clause} |

UNVERIFIED REQUIREMENTS:
  FR-{id}: No contract verifies this
    Action: Add contract to {class}.{feature}

OUTPUT FORMAT:

# CONTRACT DESIGN: {project_name}

## Contract Summary
- Classes: {count}
- Features: {count}
- Preconditions: {count}
- Postconditions: {count}
- Invariants: {count}
- Coverage: {percent}%

## Class Contracts

### {CLASS_NAME}

#### Invariants
```eiffel
invariant
  {invariant_clauses}
```

#### Creation Contracts
```eiffel
make (a_param: TYPE)
  require
    {preconditions}
  ensure
    {postconditions}
  end
```

#### Feature Contracts
```eiffel
{feature_name}: RESULT_TYPE
  require
    {preconditions}
  ensure
    {postconditions}
  end
```

### {NEXT_CLASS}
...

## Requirement Traceability
| Requirement | Contract |
|-------------|----------|
| FR-001 | {class}.{feature} ensure {x} |

## Semantic Postconditions
{List of meaningful postconditions}

## Missing Contracts
{List of gaps to fill}
```

## Success Criteria
- All classes have invariants
- All features have preconditions (where applicable)
- All features have postconditions
- Semantic postconditions verify meaning
- Requirements traced to contracts

## Next Step
→ R06-INTERFACE-DESIGN.md
