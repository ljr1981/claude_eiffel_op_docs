# M06: Fix Contracts (Strengthen Contract Layer)

## Context
Phase: **STRENGTHEN** (2 of 3)
Scenario: Pre-existing codebase, contract issues identified in M02

## Input Required
- Contract audit from M02 (gaps identified)
- Specification audit/fixes from M01/M05
- Problem domain description
- Source code

## Prompt

```
TASK: Fix absent or malformed contracts to align with spec and domain.

INPUT:
{contract_audit}
{specifications}
{problem_domain}
{eiffel_source_code}

═══════════════════════════════════════════════════════════════
CONTRACT LAYER FIXES
Contracts = machine-checkable form of specifications
═══════════════════════════════════════════════════════════════

FOR EACH CONTRACT ISSUE:

1. ABSENT CONTRACT → ADD
   - Derive from specification
   - Align with domain rules
   - Make machine-checkable

2. MALFORMED CONTRACT → CORRECT
   - Identify mismatch with spec/domain
   - Fix to match spec/domain truth

═══════════════════════════════════════════════════════════════
PRECONDITION FIXES
═══════════════════════════════════════════════════════════════

For absent/weak preconditions:
- What does SPEC say about valid inputs?
- What does DOMAIN require?
- Convert to machine-checkable require clause

═══════════════════════════════════════════════════════════════
POSTCONDITION FIXES
═══════════════════════════════════════════════════════════════

For absent/weak postconditions:
- What does SPEC say feature guarantees?
- What does DOMAIN require as outcome?
- Convert to machine-checkable ensure clause

═══════════════════════════════════════════════════════════════
INVARIANT FIXES
═══════════════════════════════════════════════════════════════

TASK: Add or strengthen class invariants.

INPUT:
{contract_audit_gaps}
{specifications}
{eiffel_source_code}

INVARIANT DISCOVERY PROCESS:

1. ANALYZE ATTRIBUTES
   For each attribute, ask:
   - Valid range? (count >= 0, percentage <= 100)
   - Never void? (for attached types, compiler handles; for detachable, add check)
   - Relationship to other attributes? (start <= end, count <= capacity)

2. ANALYZE POSTCONDITIONS
   Look for patterns across features:
   - If multiple features ensure "count >= 0", promote to invariant
   - If relationship maintained by all mutators, it's an invariant

3. ANALYZE COLLECTIONS
   - No void elements? across items as ic all ic /= Void end
   - Sorted? across 1 |..| count - 1 as i all items[i] <= items[i+1] end
   - Unique? (complex - may need helper query)

4. ANALYZE STATE MACHINES
   - Valid states enumerated?
   - State transition rules?

INVARIANT CATEGORIES:
    invariant
        -- Attribute validity
        count_non_negative: count >= 0

        -- Attribute relationships
        consistent_bounds: lower <= upper

        -- Collection integrity
        no_void_items: across items as ic all ic /= Void end

        -- State machine
        valid_state: state = Open or state = Closed or state = Pending

OUTPUT FORMAT:
CLASS: {class_name}
CURRENT INVARIANT: [none | existing clauses]

DISCOVERED INVARIANTS:

Category: Attribute Validity
    {tag}: {expression}  -- {rationale}

Category: Attribute Relationships
    {tag}: {expression}  -- {rationale}

Category: Collection Integrity
    {tag}: {expression}  -- {rationale}

RISK ASSESSMENT:
- {invariant_tag}: [LOW | MEDIUM | HIGH] - {why}

HIGH RISK = likely to fail on existing data/usage
```

## Success Criteria
- Attribute validity captured
- Relationships between attributes captured
- Collection integrity addressed
- Risk of breaking existing code assessed

## Next Step
→ M07-COMPILE-VALIDATE.md (PHASE CHANGE: Validate)
