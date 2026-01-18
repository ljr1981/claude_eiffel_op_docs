# R06: Interface Design

## Context
Phase: **DESIGN** (3 of 3)
Purpose: Design clean, usable public APIs

## Mindset

**The interface IS the specification. Implementation is detail.**

A well-designed interface:
- Is easy to use correctly
- Is hard to use incorrectly
- Follows Eiffel conventions
- Supports the builder pattern
- Has clear, consistent naming

## Input Required
- Class design from R04
- Contract design from R05

## Prompt

```
TASK: Design clean, usable public interfaces for all classes.

INPUT:
{class_design_from_R04}
{contract_design_from_R05}

═══════════════════════════════════════════════════════════════
FACADE INTERFACE DESIGN
Design the main entry point API
═══════════════════════════════════════════════════════════════

FACADE: SIMPLE_{X}

PURPOSE: {what the library does}

INTERFACE:

```eiffel
class
  SIMPLE_{X}

create
  make,
  make_with_{option}

feature -- Access

  -- Configuration state
  {option1}: {TYPE}
    -- Current {option1} setting.

  {option2}: {TYPE}
    -- Current {option2} setting.

feature -- Status

  is_ready: BOOLEAN
    -- Is the object ready for use?

  has_error: BOOLEAN
    -- Did the last operation fail?

  last_error: detachable STRING
    -- Error message if `has_error`.

feature -- Configuration (Builder Pattern)

  set_{option1} (a_value: {TYPE}): like Current
    -- Set {option1} and return self for chaining.
    require
      valid_value: {validation}
    ensure
      {option1}_set: {option1} = a_value
      result_is_current: Result = Current
    end

  set_{option2} (a_value: {TYPE}): like Current
    -- Set {option2} and return self for chaining.

feature -- Operations

  {primary_operation} (a_input: {INPUT_TYPE}): {RESULT_TYPE}
    -- {What this does}.
    require
      input_valid: {precondition}
    ensure
      result_valid: {postcondition}
    end

  {secondary_operation} (a_param: {TYPE}): {RESULT}
    -- {What this does}.
```

USAGE EXAMPLE:
```eiffel
local
  l_lib: SIMPLE_{X}
  l_result: {RESULT_TYPE}
do
  create l_lib.make
  l_lib.set_{option1} (value1)
       .set_{option2} (value2)
  l_result := l_lib.{primary_operation} (input)
end
```

═══════════════════════════════════════════════════════════════
RESULT INTERFACE DESIGN
Design result/data class APIs
═══════════════════════════════════════════════════════════════

RESULT CLASS: {RESULT_NAME}

```eiffel
class
  {RESULT_NAME}

feature -- Access

  -- Primary data
  {data1}: {TYPE}
    -- The {data1}.

  {data2}: {TYPE}
    -- The {data2}.

feature -- Status

  is_valid: BOOLEAN
    -- Is this result valid?

  is_empty: BOOLEAN
    -- Is this result empty?

  has_{property}: BOOLEAN
    -- Does this have {property}?

feature -- Measurement

  count: INTEGER
    -- Number of {items}.
    ensure
      non_negative: Result >= 0
    end

  {measurement}: {NUMERIC_TYPE}
    -- The {measurement}.

feature -- Comparison

  is_equal (other: like Current): BOOLEAN
    -- Is `other` equal to Current?

feature -- Output

  to_string: STRING
    -- String representation.

  to_json: STRING
    -- JSON representation.

  to_{format}: STRING
    -- {Format} representation.
```

═══════════════════════════════════════════════════════════════
COMMAND-QUERY SEPARATION
Verify all features follow CQS
═══════════════════════════════════════════════════════════════

For each feature:

FEATURE: {class}.{feature}
  Type: [QUERY | COMMAND | CREATION]

  If QUERY:
    Returns value: [YES]
    Modifies state: [NO - required]

  If COMMAND:
    Returns value: [NO - except like Current for builders]
    Modifies state: [YES]

CQS VIOLATIONS:
  {feature}: returns value AND modifies state
    Fix: Split into {query} and {command}

═══════════════════════════════════════════════════════════════
UNIFORM ACCESS DESIGN
Design consistent access
═══════════════════════════════════════════════════════════════

For each public attribute:

ACCESS: {class}.{attribute}
  Current: attribute
  Could be: function (computed)
  Client sees: same interface

PRINCIPLE: Clients shouldn't know if value is stored or computed.

VERIFY:
  All queries that COULD be attributes ARE attributes: [YES | NO]
  All attributes that COULD be functions CAN be changed later: [YES]

═══════════════════════════════════════════════════════════════
NAMING CONVENTION
Consistent, Eiffel-idiomatic names
═══════════════════════════════════════════════════════════════

NAMING RULES:

Classes:
  - Facades: SIMPLE_{X}
  - Engines: {X}_ENGINE
  - Results: {X}_RESULT
  - Items: {X}_ITEM
  - Builders: {X}_BUILDER

Features:
  - Queries (boolean): is_{state}, has_{property}, can_{action}
  - Queries (data): {noun}, {noun}_count, last_{noun}
  - Commands: {verb}, {verb}_{noun}, set_{attribute}
  - Creation: make, make_from_{source}, make_with_{options}

Parameters:
  - Always prefixed: a_{name}, an_{name}
  - Descriptive: a_source_path, a_target_count

NAMING REVIEW:
  {feature}: {current_name} → {better_name}
    Reason: {why rename}

═══════════════════════════════════════════════════════════════
BUILDER PATTERN DESIGN
Design fluent configuration
═══════════════════════════════════════════════════════════════

BUILDER: {CLASS}

  Builder methods (return like Current):
```eiffel
set_{option1} (a_value: TYPE): like Current
  do
    {option1} := a_value
    Result := Current
  ensure
    option_set: {option1} = a_value
    result_is_current: Result = Current
  end
```

  Chain example:
```eiffel
create obj.make
obj.set_a (1)
   .set_b (2)
   .set_c (3)
```

═══════════════════════════════════════════════════════════════
ERROR HANDLING DESIGN
Design error reporting interface
═══════════════════════════════════════════════════════════════

ERROR PATTERN:

```eiffel
feature -- Status

  has_error: BOOLEAN
    -- Did the last operation fail?

  last_error: detachable STRING
    -- Error message if `has_error`.

  last_error_code: INTEGER
    -- Error code if `has_error`.

feature -- Error Codes

  Error_none: INTEGER = 0
  Error_invalid_input: INTEGER = 1
  Error_file_not_found: INTEGER = 2
  ...
```

USAGE:
```eiffel
result := obj.operation (input)
if obj.has_error then
  print (obj.last_error)
end
```

═══════════════════════════════════════════════════════════════
VISIBILITY DESIGN
Decide feature visibility
═══════════════════════════════════════════════════════════════

For each feature:

FEATURE: {feature}
  Public (feature): Used by clients
  Restricted (feature {CLASS}): Used by specific classes
  Private (feature {NONE}): Implementation only

VISIBILITY ASSIGNMENTS:
```eiffel
feature -- Access (PUBLIC)
  {public_queries}

feature -- Operations (PUBLIC)
  {public_commands}

feature {SIMPLE_{X}} -- Internal (RESTRICTED)
  {features_for_facade}

feature {NONE} -- Implementation (PRIVATE)
  {private_features}
```

═══════════════════════════════════════════════════════════════
API DOCUMENTATION
Design clear documentation
═══════════════════════════════════════════════════════════════

DOCUMENTATION TEMPLATE:

```eiffel
{feature_name} (a_param: TYPE): RESULT
    -- {Brief description of what this does}.
    -- {Additional context if needed}.
    --
    -- `a_param`: {What the parameter represents}
    --
    -- Returns: {What the result represents}
    --
    -- Example:
    --   result := obj.{feature_name} (value)
  require
    param_valid: {condition}
  ensure
    result_valid: {condition}
  end
```

═══════════════════════════════════════════════════════════════
INTERFACE SUMMARY
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

# INTERFACE DESIGN: {project_name}

## Public Classes
| Class | Role | Key Operations |
|-------|------|----------------|
| SIMPLE_{X} | Facade | {operations} |
| {RESULT} | Data | {queries} |

## Facade Interface
```eiffel
{Complete facade class interface}
```

## Result Interfaces
```eiffel
{Result class interfaces}
```

## Builder Pattern
{Configuration API design}

## Error Handling
{Error pattern design}

## Naming Conventions
{Summary of naming rules}

## API Examples

### Basic Usage
```eiffel
{code example}
```

### Configuration
```eiffel
{builder example}
```

### Error Handling
```eiffel
{error handling example}
```

## CQS Compliance
{Verification of command-query separation}

## Visibility Summary
| Class | Public | Private |
|-------|--------|---------|
| {class} | {count} | {count} |
```

## Success Criteria
- All public interfaces defined
- Builder pattern implemented
- Naming consistent
- CQS followed
- Error handling designed
- Documentation template applied

## Next Step
→ R07-SYNTHESIZE-SPEC.md
