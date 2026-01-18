# M01: Audit Specifications

## Context
Phase: **AUDIT** (1 of 3)
Scenario: Pre-existing codebase, improvement goal

## Input Required
- Eiffel source files
- Any existing documentation (README, docs/, comments)
- Problem domain description

## Prompt

```
TASK: Audit specification coverage and correctness.

INPUT:
{eiffel_source_code}
{existing_documentation}
{problem_domain}

═══════════════════════════════════════════════════════════════
SPECIFICATION AUDIT
Specifications = documented intent (notes, comments, docs, README)
═══════════════════════════════════════════════════════════════

FOR EACH CLASS:

1. CLASS-LEVEL SPECIFICATION
   [ ] Has 'note' clause with description?
   [ ] Description explains class responsibility?
   [ ] Description matches what class actually does?

2. FEATURE-LEVEL SPECIFICATION
   For each public feature:
   [ ] Has header comment describing purpose?
   [ ] Comment matches what feature actually does?
   [ ] Edge cases documented?
   [ ] Error conditions documented?

3. SPECIFICATION CORRECTNESS
   [ ] Spec aligns with domain requirements?
   [ ] Spec is unambiguous (single interpretation)?
   [ ] Spec is internally consistent?
   [ ] Spec doesn't describe impossible behavior?

═══════════════════════════════════════════════════════════════
COVERAGE ASSESSMENT
═══════════════════════════════════════════════════════════════

ABSENT SPECIFICATIONS:
- Classes with no description
- Features with no purpose documented
- Complex behaviors with no written requirements
- Edge cases with no defined expected behavior

MALFORMED SPECIFICATIONS:
- Specs that contradict domain rules
- Ambiguous specs (multiple interpretations)
- Internally inconsistent specs
- Specs that conflict with other specs
- Outdated specs (domain has changed)

OUTPUT FORMAT:
CLASS: {class_name}

SPECIFICATION STATUS:
  Class-level: [ABSENT | PRESENT | MALFORMED]
  If MALFORMED: {what's wrong}

  Feature-level coverage: {X}% of public features documented

FEATURE: {feature_name}
  Spec status: [ABSENT | PRESENT | MALFORMED]
  If MALFORMED:
    Current spec: {what it says}
    Problem: {why it's wrong}
    Should say: {correct spec}

SPECIFICATION SCORE: {X}/10

PRIORITY FIXES:
1. {most critical missing/malformed spec}
2. {next priority}
...
```

## Success Criteria
- Every class assessed for specification
- Every public feature checked
- Absent vs malformed distinguished
- Priorities ranked

## Next Step
→ M02-AUDIT-CONTRACTS.md
