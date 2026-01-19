# 7S-03: Requirements

## Context
Step: **3 of 7**
Purpose: Define detailed requirements based on scope and landscape

## Mindset

**Requirements are the contract between stakeholders and builders.**

Requirements must be:
- **Complete**: Cover all needs
- **Correct**: Accurately reflect needs
- **Consistent**: No contradictions
- **Verifiable**: Can be tested
- **Prioritized**: Ranked by importance

## Input Required
- Scope definition from 7S-01
- Landscape analysis from 7S-02

## Prompt

```
TASK: Define comprehensive requirements for this solution.

INPUT:
{scope_definition_from_7S01}
{landscape_analysis_from_7S02}

═══════════════════════════════════════════════════════════════
FUNCTIONAL REQUIREMENTS
What must the system DO?
═══════════════════════════════════════════════════════════════

For each capability:

FR-{id}: {requirement_name}
  Description: {what the system must do}
  Rationale: {why this is needed}
  Priority: [MUST | SHOULD | COULD | WONT]
  Source: {who requested / where identified}
  Acceptance criteria:
    - {criterion 1}
    - {criterion 2}
  Related: {other requirements}

FUNCTIONAL REQUIREMENTS:

| ID | Requirement | Priority | Acceptance |
|----|-------------|----------|------------|
| FR-001 | {desc} | MUST | {criteria} |
| FR-002 | {desc} | SHOULD | {criteria} |

MUST HAVE (Core):
  FR-001: {requirement}
  FR-002: {requirement}

SHOULD HAVE (Important):
  FR-003: {requirement}
  FR-004: {requirement}

COULD HAVE (Nice to have):
  FR-005: {requirement}

WONT HAVE (Explicitly excluded):
  FR-006: {requirement} - {reason for exclusion}

═══════════════════════════════════════════════════════════════
NON-FUNCTIONAL REQUIREMENTS
How must the system BEHAVE?
═══════════════════════════════════════════════════════════════

PERFORMANCE:
  NFR-P-001: {requirement}
    Metric: {what to measure}
    Target: {specific value}
    Validation: {how to test}

RELIABILITY:
  NFR-R-001: {requirement}
    Metric: {what to measure}
    Target: {specific value}

USABILITY:
  NFR-U-001: {requirement}
    Metric: {what to measure}
    Target: {specific value}

SECURITY:
  NFR-S-001: {requirement}
    Threat: {what it protects against}
    Control: {how achieved}

COMPATIBILITY:
  NFR-C-001: {requirement}
    Must work with: {systems}
    Must not conflict with: {systems}

MAINTAINABILITY:
  NFR-M-001: {requirement}
    Standard: {coding/documentation standards}

NON-FUNCTIONAL SUMMARY:

| ID | Category | Requirement | Target |
|----|----------|-------------|--------|
| NFR-P-001 | Performance | {desc} | {target} |
| NFR-R-001 | Reliability | {desc} | {target} |

═══════════════════════════════════════════════════════════════
CONSTRAINTS
What limits our solution space?
═══════════════════════════════════════════════════════════════

TECHNICAL CONSTRAINTS:
  C-T-001: {constraint}
    Impact: {how it affects design}
    Non-negotiable: [YES | NO]

BUSINESS CONSTRAINTS:
  C-B-001: {constraint}
    Impact: {how it affects scope}
    Non-negotiable: [YES | NO]

REGULATORY CONSTRAINTS:
  C-R-001: {constraint}
    Compliance: {what standard/law}
    Impact: {what we must do}

CONSTRAINT SUMMARY:

| ID | Type | Constraint | Negotiable |
|----|------|------------|------------|
| C-T-001 | Technical | {desc} | NO |
| C-B-001 | Business | {desc} | YES |

═══════════════════════════════════════════════════════════════
USE CASE SPECIFICATIONS
Detailed user interactions
═══════════════════════════════════════════════════════════════

USE CASE: UC-{id} - {name}

  Actor: {who performs this}
  Goal: {what they want to achieve}
  Preconditions:
    - {what must be true before}
  Postconditions:
    - {what is true after}

  Main Success Scenario:
    1. {Actor does X}
    2. {System responds with Y}
    3. {Actor does Z}
    4. ...

  Extensions (Alternative Flows):
    2a. {If condition}:
        1. {Alternative step}

  Error Conditions:
    E1. {Error}: {how handled}
    E2. {Error}: {how handled}

  Requirements Satisfied: FR-{x}, FR-{y}

═══════════════════════════════════════════════════════════════
INPUT/OUTPUT SPECIFICATIONS
What data flows through the system?
═══════════════════════════════════════════════════════════════

INPUTS:
  INPUT: {name}
    Type: {data type}
    Source: {where it comes from}
    Format: {structure/format}
    Validation: {rules}
    Example: {sample}

OUTPUTS:
  OUTPUT: {name}
    Type: {data type}
    Destination: {where it goes}
    Format: {structure/format}
    Example: {sample}

═══════════════════════════════════════════════════════════════
DATA REQUIREMENTS
What data must be managed?
═══════════════════════════════════════════════════════════════

DATA ENTITIES:
  ENTITY: {name}
    Attributes: {list}
    Constraints: {rules}
    Volume: {expected size}
    Retention: {how long kept}

DATA RELATIONSHIPS:
  {Entity A} → {Entity B}: {relationship}

═══════════════════════════════════════════════════════════════
INTERFACE REQUIREMENTS
How does the system connect?
═══════════════════════════════════════════════════════════════

USER INTERFACE:
  IR-UI-001: {requirement}
    User: {who}
    Interaction: {how}

API INTERFACE:
  IR-API-001: {requirement}
    Consumer: {who uses it}
    Protocol: {how to call}
    Format: {data format}

EXTERNAL SYSTEM INTERFACE:
  IR-EXT-001: {requirement}
    System: {external system}
    Integration: {how to connect}

═══════════════════════════════════════════════════════════════
REQUIREMENT DEPENDENCIES
Which requirements depend on others?
═══════════════════════════════════════════════════════════════

DEPENDENCY MAP:
  FR-003 depends on: FR-001, FR-002
  FR-005 depends on: FR-004

ORDER OF IMPLEMENTATION:
  1. FR-001, FR-002 (no dependencies)
  2. FR-003, FR-004 (depends on above)
  3. FR-005 (depends on above)

═══════════════════════════════════════════════════════════════
REQUIREMENT GAPS
What questions remain?
═══════════════════════════════════════════════════════════════

GAPS:
  GAP-001: {unclear requirement}
    Needs: {clarification from whom}

ASSUMPTIONS:
  ASM-001: {assumption made}
    Risk: {if wrong}

OUTPUT FORMAT:

# REQUIREMENTS: {project_name}

## Requirements Summary
| Type | MUST | SHOULD | COULD | Total |
|------|------|--------|-------|-------|
| Functional | {n} | {n} | {n} | {n} |
| Non-Functional | {n} | - | - | {n} |
| Constraints | {n} | - | - | {n} |

## Functional Requirements

### Core (MUST)
| ID | Requirement | Acceptance Criteria |
|----|-------------|---------------------|
| FR-001 | {desc} | {criteria} |

### Important (SHOULD)
{Table}

### Nice to Have (COULD)
{Table}

## Non-Functional Requirements
| ID | Category | Requirement | Target |
|----|----------|-------------|--------|
| NFR-001 | {cat} | {desc} | {target} |

## Constraints
| ID | Constraint | Impact |
|----|------------|--------|
| C-001 | {desc} | {impact} |

## Use Cases
{Detailed use case specifications}

## Data Requirements
{Data entities and relationships}

## Interface Requirements
{Interface specifications}

## Dependencies
{Requirement dependency graph}

## Open Questions
{Gaps and assumptions}
```

## Success Criteria
- All functional requirements prioritized
- Non-functional requirements quantified
- Constraints identified
- Use cases detailed
- Dependencies mapped

## Next Step
→ 7S-04-DECISIONS.md

CRITICAL: STOP here and tell me what the next step is and prompt me to tell you to continue with my added instruction.
