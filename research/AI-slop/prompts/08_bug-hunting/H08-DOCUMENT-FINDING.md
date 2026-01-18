# H08: Document Finding

## Context
Phase: **VERIFY** (3 of 3)
Scenario: Bug hunt complete, documenting for fix

## Input Required
- Root cause analysis from H07
- Exploit test from H06
- Problem domain description

## Prompt

```
TASK: Document the discovered bug comprehensively for handoff to fixing workflow.

INPUT:
{root_cause_analysis}
{exploit_test}
{problem_domain}

═══════════════════════════════════════════════════════════════
BUG REPORT TEMPLATE
═══════════════════════════════════════════════════════════════

=== BUG REPORT: {short_title} ===

SUMMARY:
{One paragraph in DOMAIN terms - what goes wrong for users}

SEVERITY: [CRITICAL | HIGH | MEDIUM | LOW]

MALFORMATION TYPE: [ABSENT | MALFORMED_CONTRACT | MALFORMED_CODE | COMBINED]

LOCATION:
  Class: {CLASS_NAME}
  Feature: {feature_name}
  Line: {approximate line}

DOMAIN CONTEXT:
  Feature purpose: {what it should do in domain terms}
  Domain rules: {relevant business rules}
  Domain impact: {what users experience when bug occurs}

THE MALFORMATION:

  IF ABSENT:
    Missing: [PRECONDITION | POSTCONDITION | INVARIANT]
    Should be: {the contract that should exist}
    Why it matters: {what the absence allows}

  IF MALFORMED_CONTRACT:
    Current contract: {existing contract}
    Problem: {why it's semantically wrong}
    Correct contract: {what it should be}
    Domain mismatch: {how it violates domain rules}

  IF MALFORMED_CODE:
    Current behavior: {what code does}
    Problem: {why it's semantically wrong}
    Correct behavior: {what domain requires}
    Domain violation: {specific rule violated}

  IF COMBINED:
    Contract issue: {description}
    Code issue: {description}
    Interaction: {how they compound}

REPRODUCTION:
```eiffel
{minimal test case from H06}
```

RECOMMENDED FIX:

  Fix Type: [ADD_CONTRACT | FIX_CONTRACT | FIX_CODE | FIX_BOTH]

  IF CONTRACT FIX:
    Add/Change:
        require/ensure/invariant
            {correct_contract}

  IF CODE FIX:
    Change behavior to: {description}
    Pseudo-code: {outline}

PREVENTION (for future):
  Contract that would catch similar bugs:
      {preventive contract pattern}

  Code review question to ask:
      "{question that would have caught this}"

REGRESSION TEST:
  The exploit test (H06) becomes the regression test.
  After fix, this test must PASS.

=== END REPORT ===
```

## Success Criteria
- Malformation type clearly documented
- Domain impact articulated
- Fix recommendation includes correct contract/code
- Prevention guidance provided

## Handoff
This document is the input to the **Bug Fixing** workflow (F01-F07).

The bug report explicitly identifies:
- WHETHER contract is absent vs malformed vs code is malformed
- WHAT the correct contract/code should be
- WHY in domain terms

Hunt complete.
