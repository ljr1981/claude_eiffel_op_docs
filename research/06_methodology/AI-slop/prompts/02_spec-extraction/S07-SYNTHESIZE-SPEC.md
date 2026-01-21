# S07: Synthesize Specification

## Context
Phase: **SYNTHESIS** (1 of 2)
Purpose: Combine all extracted information into formal specification

## Input Required
- Project inventory (S01)
- Domain model (S02)
- Class specifications (S03)
- Feature specifications (S04)
- Constraints (S05)
- Boundaries (S06)

## Prompt

```
TASK: Synthesize a formal specification document from all extracted information.

INPUT:
{project_inventory}
{domain_model}
{class_specifications}
{feature_specifications}
{constraints}
{boundaries}

═══════════════════════════════════════════════════════════════
SPECIFICATION STRUCTURE
Organize into standard specification format
═══════════════════════════════════════════════════════════════

The synthesized specification should follow this structure:

1. EXECUTIVE SUMMARY
2. SCOPE AND PURPOSE
3. DOMAIN MODEL
4. SYSTEM ARCHITECTURE
5. CLASS SPECIFICATIONS
6. BEHAVIORAL SPECIFICATIONS
7. CONSTRAINTS AND INVARIANTS
8. BOUNDARY CONDITIONS
9. ERROR HANDLING
10. QUALITY ATTRIBUTES
11. OPEN QUESTIONS

═══════════════════════════════════════════════════════════════
EXECUTIVE SUMMARY
One-page overview
═══════════════════════════════════════════════════════════════

Write a 1-page summary covering:
- What this library does
- Who it's for
- Key capabilities
- Design principles
- Quality guarantees

═══════════════════════════════════════════════════════════════
SCOPE AND PURPOSE
Formal problem statement
═══════════════════════════════════════════════════════════════

PURPOSE:
  {Library name} provides {what capability} for {what users}
  by {how it works}.

SCOPE:
  IN SCOPE:
    - {capability 1}
    - {capability 2}

  OUT OF SCOPE:
    - {explicitly not supported}

ASSUMPTIONS:
  - {assumption 1}
  - {assumption 2}

═══════════════════════════════════════════════════════════════
SYSTEM ARCHITECTURE
How components fit together
═══════════════════════════════════════════════════════════════

ARCHITECTURE DIAGRAM:
```
┌─────────────────────────────────────────────┐
│                 FACADE                       │
│            {facade_class}                   │
└─────────────────┬───────────────────────────┘
                  │ delegates to
        ┌─────────┼─────────┐
        ▼         ▼         ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ ENGINE  │ │ RENDERER│ │ APPLIER │
   └────┬────┘ └─────────┘ └─────────┘
        │ produces
        ▼
   ┌─────────┐
   │ RESULT  │──contains──> [ITEMS]
   └─────────┘
```

COMPONENT RESPONSIBILITIES:
  - {Component}: {responsibility}

═══════════════════════════════════════════════════════════════
CONSOLIDATED CLASS SPECIFICATIONS
Clean specification for each class
═══════════════════════════════════════════════════════════════

For each class, create formal spec:

## {CLASS_NAME}

### Purpose
{Single sentence purpose}

### Invariants
```eiffel
invariant
  {invariant1}
  {invariant2}
```

### Creation
| Procedure | Purpose | Preconditions | Postconditions |
|-----------|---------|---------------|----------------|
| {make} | {purpose} | {require} | {ensure} |

### Queries
| Feature | Returns | Purpose | Preconditions |
|---------|---------|---------|---------------|
| {query} | {type} | {purpose} | {require} |

### Commands
| Feature | Purpose | Preconditions | Postconditions |
|---------|---------|---------------|----------------|
| {command} | {purpose} | {require} | {ensure} |

═══════════════════════════════════════════════════════════════
BEHAVIORAL SPECIFICATIONS
Key algorithms and workflows
═══════════════════════════════════════════════════════════════

WORKFLOW: {name}
  Purpose: {what this accomplishes}

  Steps:
    1. {step 1}
    2. {step 2}
    ...

  Preconditions:
    - {what must be true before}

  Postconditions:
    - {what is guaranteed after}

═══════════════════════════════════════════════════════════════
CONSTRAINTS SUMMARY
All system-wide rules
═══════════════════════════════════════════════════════════════

Consolidate constraints into categories:

INTEGRITY RULES:
  I1: {rule}
  I2: {rule}

VALIDITY RULES:
  V1: {rule}
  V2: {rule}

BUSINESS RULES:
  B1: {rule}
  B2: {rule}

═══════════════════════════════════════════════════════════════
QUALITY ATTRIBUTES
Non-functional requirements derived from design
═══════════════════════════════════════════════════════════════

VOID SAFETY: [FULL | PARTIAL | NONE]
  {Evidence and assessment}

SCOOP COMPATIBILITY: [YES | NO | PARTIAL]
  {Evidence and assessment}

TESTABILITY: [HIGH | MEDIUM | LOW]
  {Evidence and assessment}

CONTRACT COVERAGE: {X}%
  {Assessment of specification completeness}

OUTPUT FORMAT:

Create a complete specification document:

# {LIBRARY_NAME} Specification

## 1. Executive Summary
{1-page overview}

## 2. Scope and Purpose
{Problem statement, scope, assumptions}

## 3. Domain Model
{Concepts, relationships, vocabulary}

## 4. System Architecture
{Component diagram and responsibilities}

## 5. Class Specifications
{Per-class formal specifications}

## 6. Behavioral Specifications
{Key workflows and algorithms}

## 7. Constraints
{System-wide rules}

## 8. Boundary Conditions
{Edge cases and limits}

## 9. Error Handling
{Error conditions and recovery}

## 10. Quality Attributes
{Non-functional characteristics}

## 11. Open Questions
{Ambiguities requiring human review}

## Appendix A: Contract Gaps
{Missing contracts identified}

## Appendix B: Test Coverage Gaps
{Missing tests identified}
```

## Success Criteria
- Complete, coherent specification document
- All extracted information integrated
- Consistent terminology throughout
- Clear structure and navigation
- Actionable gaps identified

## Output Location
Create: `specs/SPEC-SUMMARY.md`

## Next Step
→ S08-VALIDATE-SPEC.md
