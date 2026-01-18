# H04: Probe State Sequences

## Context
Phase: **PROBE** (2 of 3)
Scenario: Bug hunting, testing operation ordering and state integrity

## Input Required
- Source code
- Hunt targets from H02
- Problem domain (valid state transitions)

## Prompt

```
TASK: Generate operation sequences that probe state integrity and domain rules.

INPUT:
{eiffel_source_code}
{hunt_targets}
{problem_domain}

═══════════════════════════════════════════════════════════════
TECHNICAL STATE SEQUENCES
═══════════════════════════════════════════════════════════════

1. USE-BEFORE-INITIALIZE
   - Call feature before make completes
   - Access attribute before set

2. USE-AFTER-CLOSE
   - Call feature after close/dispose
   - Access released resource

3. DOUBLE OPERATIONS
   - Open/close/add/remove twice

4. INTERLEAVED OPERATIONS
   - Modify during iteration
   - Callback modifies caller

═══════════════════════════════════════════════════════════════
DOMAIN STATE SEQUENCES
These probe whether code respects domain state machines.
═══════════════════════════════════════════════════════════════

1. EXTRACT DOMAIN STATE MACHINE
   From domain knowledge, identify:
   - Valid states: {list domain states}
   - Valid transitions: {state A → state B via operation}
   - Invalid transitions: {state A cannot → state C}

2. PROBE VALID PATHS
   - Execute canonical valid sequence
   - Verify each state reached correctly

3. PROBE INVALID TRANSITIONS
   - Attempt forbidden state transition
   - Should fail (precondition) or be rejected

4. PROBE INVARIANT PRESERVATION
   After each operation:
   - Is domain invariant maintained?
   - If code allows invalid state → BUG (malformed code/contract)

═══════════════════════════════════════════════════════════════
CONTRACT-STATE ALIGNMENT
Do contracts correctly guard state transitions?
═══════════════════════════════════════════════════════════════

For each state-changing feature:
  Q: Does precondition check valid source state?
  Q: Does postcondition guarantee valid target state?
  Q: Does invariant catch invalid intermediate states?

MALFORMATION DETECTION:
  - Contract allows invalid transition → malformed contract
  - Contract correct but code violates it → malformed code
  - Both wrong → Category D from H01

OUTPUT FORMAT:
CLASS: {class_name}

DOMAIN STATE MACHINE:
  States: {domain states}
  Valid transitions:
    - {state} → {state} via {feature}
  Invalid transitions:
    - {state} cannot → {state}

SEQUENCE PROBES:

Sequence 1: {name}
  Type: [TECHNICAL | DOMAIN_VALID | DOMAIN_INVALID]
  Operations: {step-by-step}
  Expected: {what should happen}
  If malformed contract: {how it fails to guard}
  If malformed code: {how it violates domain}

TEST CODE:
```eiffel
test_sequence_{name}
    local
        l_obj: {CLASS}
    do
        create l_obj.make
        -- Sequence targeting {issue}
        {operation_1}
        assert ("state_after_1", {expected_state})
        {operation_2}
        assert ("state_after_2", {expected_state})
        -- Final domain invariant check
        assert ("domain_valid", {domain_condition})
    end
```
```

## Success Criteria
- Domain state machine extracted
- Valid and invalid paths probed
- Contract-state alignment checked
- Malformation type identified

## Next Step
→ H05-PROBE-CONCURRENCY.md
