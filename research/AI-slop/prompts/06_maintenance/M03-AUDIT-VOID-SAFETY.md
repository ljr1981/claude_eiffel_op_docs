# M03: Audit Code

## Context
Phase: **AUDIT** (1 of 3)
Scenario: Pre-existing codebase, improvement goal

## Input Required
- Eiffel source files
- Specification audit from M01
- Contract audit from M02
- Problem domain description

## Prompt

```
TASK: Audit code for domain alignment and structural quality.

INPUT:
{eiffel_source_code}
{specification_audit}
{contract_audit}
{problem_domain}

═══════════════════════════════════════════════════════════════
CODE-SPECIFICATION ALIGNMENT
Does the code do what the spec says?
═══════════════════════════════════════════════════════════════

FOR EACH FEATURE:
[ ] Code behavior matches spec description?
[ ] Code handles documented edge cases?
[ ] Code handles documented error conditions?

MISALIGNMENT PATTERNS:
- Spec says X, code does Y
- Spec documents edge case, code doesn't handle it
- Code has behavior not documented in spec

═══════════════════════════════════════════════════════════════
CODE-CONTRACT ALIGNMENT
Does the code satisfy its contracts?
═══════════════════════════════════════════════════════════════

FOR EACH FEATURE:
[ ] Code assumes preconditions are true? (doesn't re-check)
[ ] Code achieves postconditions?
[ ] Code maintains invariant?

MISALIGNMENT PATTERNS:
- Postcondition says X, code might produce Y
- Code can violate invariant
- Code re-checks preconditions (redundant/wrong)

═══════════════════════════════════════════════════════════════
CODE-DOMAIN ALIGNMENT
Does the code respect domain rules?
═══════════════════════════════════════════════════════════════

FOR EACH FEATURE:
[ ] Algorithm matches domain requirements?
[ ] Domain rules enforced in code?
[ ] Domain state machine respected?

MALFORMATION PATTERNS:
- Algorithm wrong for domain (e.g., FIFO vs LIFO)
- Domain rule violated (e.g., negative balance allowed)
- State transition invalid (e.g., shipped → pending)
- Data transformation loses domain meaning

═══════════════════════════════════════════════════════════════
STRUCTURAL QUALITY
Traditional code smells
═══════════════════════════════════════════════════════════════

[ ] Command-Query Separation violations?
[ ] Feature organization issues?
[ ] Naming convention violations?
[ ] Void safety issues?
[ ] Long features (>50 lines)?
[ ] Deep nesting (>3 levels)?

OUTPUT FORMAT:
FEATURE: {class.feature}

CODE-SPEC ALIGNMENT: [ALIGNED | MISALIGNED | NO_SPEC]
  If MISALIGNED: Spec says {X}, code does {Y}

CODE-CONTRACT ALIGNMENT: [ALIGNED | MISALIGNED | NO_CONTRACT]
  If MISALIGNED: Contract says {X}, code does {Y}

CODE-DOMAIN ALIGNMENT: [ALIGNED | MISALIGNED]
  If MISALIGNED: Domain requires {X}, code does {Y}

STRUCTURAL ISSUES:
  {list of code smells found}

CODE QUALITY SCORE: {X}/10

PRIORITY FIXES:
1. {most critical code issue}
2. {next priority}
```

## Success Criteria
- Code checked against spec, contract, AND domain
- Misalignments identified at each level
- Structural quality assessed
- Priorities ranked

## Next Step
→ M04-AUDIT-TESTS.md
