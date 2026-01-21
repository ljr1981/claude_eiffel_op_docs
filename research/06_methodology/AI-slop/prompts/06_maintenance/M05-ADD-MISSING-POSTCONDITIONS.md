# M05: Fix Specifications (Strengthen Spec Layer)

## Context
Phase: **STRENGTHEN** (2 of 3)
Scenario: Pre-existing codebase, spec issues identified in M01

## Input Required
- Specification audit from M01 (gaps identified)
- Contract audit from M02
- Problem domain description
- Source code

## Prompt

```
TASK: Fix absent or malformed specifications identified in M01.

INPUT:
{specification_audit}
{contract_audit}
{problem_domain}
{source_code}

═══════════════════════════════════════════════════════════════
SPECIFICATION LAYER FIXES
Specs define WHAT the system should do (human-readable intent)
═══════════════════════════════════════════════════════════════

FOR EACH ABSENT SPECIFICATION:
1. What does the feature do? (domain meaning)
2. What inputs are valid? (will become preconditions)
3. What does it guarantee? (will become postconditions)
4. What domain rules apply?

FOR EACH MALFORMED SPECIFICATION:
1. What does it currently say?
2. What does domain require?
3. How should it be corrected?

OUTPUT FORMAT:
FEATURE: {class.feature}

SPEC STATUS: [ABSENT | MALFORMED]

IF ABSENT:
  NEW SPECIFICATION:
    Purpose: {what the feature does in domain terms}
    Valid inputs: {domain conditions}
    Guarantees: {domain outcomes}
    Domain rules: {applicable rules}

IF MALFORMED:
  CURRENT SPEC: {what it says}
  DOMAIN REQUIRES: {what domain actually requires}
  CORRECTED SPEC: {fixed specification}

CONTRACT IMPLICATIONS:
  Precondition should: {derived from spec}
  Postcondition should: {derived from spec}

═══════════════════════════════════════════════════════════════
LAYER ALIGNMENT CHECK
═══════════════════════════════════════════════════════════════

[ ] Spec matches domain? (domain is truth)
[ ] Spec will be implementable as contract?
[ ] Spec is testable?
```

## Success Criteria
- All spec gaps addressed
- Specs align with domain
- Specs ready to drive contract writing
- Contract implications identified

## Next Step
→ M06-FIX-CONTRACTS.md

---

# M05-B: Add Missing Postconditions (Legacy - Now Part of M06)

## Context
Phase: **STRENGTHEN** (2 of 3)
Scenario: Pre-existing codebase, improvement goal

## Input Required
- Contract audit from M02 (gaps identified)
- Specification audit from M01
- Source code with preconditions
- Problem domain

## Prompt

```
TASK: Add postconditions that align with spec and domain.

INPUT:
{contract_audit_gaps}
{specification_audit}
{eiffel_source_code}
{problem_domain}

FOR EACH FEATURE WITHOUT ADEQUATE POSTCONDITIONS:

1. ANALYZE THE BODY (for commands)
   - What attributes change?
   - What is the relationship between old and new values?
   - What collections are modified?

2. ANALYZE THE RESULT (for queries)
   - What properties does Result always have?
   - What is Result's relationship to arguments?
   - What is Result's relationship to object state?

3. ANALYZE WHAT DOESN'T CHANGE (frame conditions)
   - Which attributes remain unchanged?
   - Document only if non-obvious

POSTCONDITION PATTERNS:

For COMMANDS:
    ensure
        attribute_changed: attribute = new_value
        count_updated: count = old count + 1
        item_added: has (new_item)
        other_unchanged: other = old other  -- if non-obvious

For QUERIES:
    ensure
        result_valid: Result >= 0
        result_bounded: Result <= maximum
        result_computed: Result = some_expression
        result_not_void: Result /= Void  -- only if detachable return

AVOID:
- Postconditions that just restate the implementation
- Trivial postconditions (Result = Result)
- Postconditions the compiler already guarantees

OUTPUT FORMAT:
FEATURE: {feature_name}
TYPE: [COMMAND | QUERY]
CURRENT: [no ensure | existing ensure clause]

ANALYSIS:
- Changes: {what the feature modifies/computes}
- Guarantees: {what is always true after}

PROPOSED POSTCONDITIONS:
    ensure
        tag_name: expression  -- {rationale}
        tag_name: expression  -- {rationale}

VERIFICATION: Can implementation satisfy this? [YES | NO - needs impl fix]
```

## Success Criteria
- All gap features addressed
- Commands describe state changes
- Queries describe Result properties
- Implementation compatibility verified

## Next Step
→ M06-ADD-MISSING-INVARIANTS.md
