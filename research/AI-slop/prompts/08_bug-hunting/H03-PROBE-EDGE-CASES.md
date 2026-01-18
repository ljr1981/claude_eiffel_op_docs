# H03: Probe Edge Cases

## Context
Phase: **PROBE** (2 of 3)
Scenario: Bug hunting, testing boundaries and domain limits

## Input Required
- Hunt targets from H02
- Source code
- Problem domain description

## Prompt

```
TASK: Generate edge case inputs targeting both technical and domain boundaries.

INPUT:
{hunt_targets}
{eiffel_source_code}
{problem_domain}

═══════════════════════════════════════════════════════════════
TECHNICAL EDGE CASES (Traditional)
═══════════════════════════════════════════════════════════════

NUMERIC:
  - Zero, One, Negative one
  - Max/Min integer values
  - Boundary ± 1

STRING:
  - Empty, Single char, Very long
  - Whitespace only, Special chars, Unicode

COLLECTION:
  - Empty, Single element, At capacity

INDEX:
  - First (1), Last (count), Invalid (0, count+1)

═══════════════════════════════════════════════════════════════
DOMAIN EDGE CASES (Semantic)
These probe whether code respects domain rules.
═══════════════════════════════════════════════════════════════

DOMAIN BOUNDARIES:
  For each domain concept, identify:
  - Minimum valid value in domain
  - Maximum valid value in domain
  - Values just outside domain validity
  - Special domain values (e.g., leap years, fiscal year boundaries)

DOMAIN RULES:
  For each business rule, probe:
  - Input that should trigger the rule
  - Input that should NOT trigger the rule
  - Input at the rule's threshold

DOMAIN STATES:
  For each domain state machine:
  - Valid state transitions
  - Invalid state transitions (should be rejected)
  - Edge states (newly created, about to close)

DOMAIN RELATIONSHIPS:
  For each entity relationship:
  - Orphan entities (missing required relationships)
  - Circular relationships (if forbidden)
  - Relationship at capacity

═══════════════════════════════════════════════════════════════
MALFORMED CONTRACT PROBES
If H02 found malformed contracts, probe them specifically.
═══════════════════════════════════════════════════════════════

If precondition TOO WEAK:
  - Find input that passes precondition but breaks feature

If precondition TOO STRONG:
  - Find valid input that's incorrectly rejected

If postcondition INCOMPLETE:
  - Find case where postcondition passes but result is wrong

OUTPUT FORMAT:
TARGET: {class.feature}
FINDING FROM H02: {what was identified}

TECHNICAL PROBES:
  Probe 1: Input={value}, Category={type}, Expected={behavior}
  Probe 2: ...

DOMAIN PROBES:
  Probe 1: Input={domain_value}, Rule={which_rule}, Expected={domain_behavior}
  Probe 2: ...

CONTRACT PROBES (if malformed):
  Probe 1: Input={value}, Exploits={contract_weakness}, Expected={failure}

TEST CODE:
```eiffel
test_{feature}_domain_edge_{rule}
    local
        l_obj: {CLASS}
    do
        create l_obj.make
        -- Setup domain edge condition
        {setup}
        -- Probe domain boundary
        l_obj.{feature} ({domain_edge_input})
        -- Verify domain rule respected
        assert ("{domain_rule}", {domain_expectation})
    end
```
```

## Success Criteria
- Technical AND domain edges probed
- Malformed contracts specifically targeted
- Domain rules explicitly tested
- Executable test code generated

## Next Step
→ H04-PROBE-STATE-SEQUENCES.md
