# R01: Parse Research

## Context
Phase: **ANALYSIS** (1 of 3)
Purpose: Extract and structure requirements from research output

## Mindset

**Dissect the research systematically.**

The 7-step research product contains valuable information, but it's not in a form suitable for design. Extract:
- Explicit requirements
- Implicit requirements
- Constraints
- Quality attributes
- Use cases

## Input Required
- Complete 7-step research product output
- Any supplementary materials

## Prompt

```
TASK: Parse and structure the 7-step research output for specification design.

INPUT:
{7_step_research_output}
{supplementary_materials}

═══════════════════════════════════════════════════════════════
STEP 1: SCOPE EXTRACTION
What problem are we solving?
═══════════════════════════════════════════════════════════════

From STEP-1-SCOPE.md extract:

PROBLEM STATEMENT:
  {Clear statement of the problem}

SCOPE BOUNDARIES:
  IN SCOPE:
    - {capability 1}
    - {capability 2}

  OUT OF SCOPE:
    - {explicitly excluded 1}
    - {explicitly excluded 2}

TARGET USERS:
  - {user type 1}: {their needs}
  - {user type 2}: {their needs}

SUCCESS CRITERIA:
  - {criterion 1}
  - {criterion 2}

═══════════════════════════════════════════════════════════════
STEP 2: LANDSCAPE EXTRACTION
What already exists?
═══════════════════════════════════════════════════════════════

From STEP-2-LANDSCAPE.md extract:

EXISTING SOLUTIONS:
  Solution: {name}
    Strengths: {what works}
    Weaknesses: {what doesn't}
    Relevance: {how it applies to us}

PATTERNS TO ADOPT:
  - {pattern}: {why}

PATTERNS TO AVOID:
  - {anti-pattern}: {why}

COMPETITIVE REQUIREMENTS:
  Must match: {feature}
  Must exceed: {feature}

═══════════════════════════════════════════════════════════════
STEP 3: REQUIREMENTS EXTRACTION
What must the solution do?
═══════════════════════════════════════════════════════════════

From STEP-3-REQUIREMENTS.md extract:

FUNCTIONAL REQUIREMENTS:
  FR-{id}: {requirement}
    Priority: [MUST | SHOULD | COULD | WONT]
    Rationale: {why needed}
    Acceptance: {how to verify}

NON-FUNCTIONAL REQUIREMENTS:
  NFR-{id}: {requirement}
    Category: [PERFORMANCE | SECURITY | RELIABILITY | USABILITY]
    Measure: {how to quantify}
    Target: {specific value}

CONSTRAINTS:
  C-{id}: {constraint}
    Type: [TECHNICAL | BUSINESS | REGULATORY]
    Immutable: [YES | NO]

═══════════════════════════════════════════════════════════════
STEP 4: DECISIONS EXTRACTION
What has already been decided?
═══════════════════════════════════════════════════════════════

From STEP-4-DECISIONS.md extract:

DECISIONS:
  D-{id}: {decision}
    Alternatives considered: {list}
    Rationale: {why chosen}
    Implications: {what this means for design}
    Reversible: [YES | NO]

DESIGN CONSTRAINTS (from decisions):
  - {constraint}: {from decision D-X}

═══════════════════════════════════════════════════════════════
STEP 5: INNOVATIONS EXTRACTION
What novel approaches are proposed?
═══════════════════════════════════════════════════════════════

From STEP-5-INNOVATIONS.md extract:

INNOVATIONS:
  I-{id}: {innovation}
    Problem solved: {what it addresses}
    Approach: {how it works}
    Design impact: {what this means for structure}

NOVEL REQUIREMENTS (from innovations):
  FR-{id}: {requirement from innovation}

═══════════════════════════════════════════════════════════════
STEP 6: RISKS EXTRACTION
What could go wrong?
═══════════════════════════════════════════════════════════════

From STEP-6-RISKS.md extract:

RISKS:
  RISK-{id}: {risk}
    Likelihood: [HIGH | MEDIUM | LOW]
    Impact: [HIGH | MEDIUM | LOW]
    Mitigation: {how to prevent/handle}
    Design implication: {what this means for design}

DEFENSIVE REQUIREMENTS (from risks):
  FR-{id}: {requirement to mitigate RISK-X}

═══════════════════════════════════════════════════════════════
STEP 7: RECOMMENDATION SYNTHESIS
What is the final direction?
═══════════════════════════════════════════════════════════════

From STEP-7-RECOMMENDATION.md extract:

RECOMMENDED APPROACH:
  {Summary of recommended direction}

KEY FEATURES:
  - {feature 1}
  - {feature 2}

PHASING (if specified):
  Phase 1: {features}
  Phase 2: {features}

═══════════════════════════════════════════════════════════════
REQUIREMENTS CONSOLIDATION
Master requirements list
═══════════════════════════════════════════════════════════════

Consolidate all requirements into single list:

MASTER REQUIREMENTS:

| ID | Requirement | Priority | Source | Category |
|----|-------------|----------|--------|----------|
| FR-001 | {desc} | MUST | STEP-3 | Functional |
| NFR-001 | {desc} | MUST | STEP-3 | Performance |
| FR-002 | {desc} | SHOULD | STEP-5 | Innovation |

TOTAL: {count} requirements
  MUST: {count}
  SHOULD: {count}
  COULD: {count}

═══════════════════════════════════════════════════════════════
USE CASE EXTRACTION
How will users interact?
═══════════════════════════════════════════════════════════════

For each capability, define use cases:

USE CASE: UC-{id}
  Name: {name}
  Actor: {who}
  Precondition: {what must be true}
  Main flow:
    1. {step}
    2. {step}
    ...
  Postcondition: {what is true after}
  Variations:
    - {alternative flow}

OUTPUT FORMAT:

# PARSED RESEARCH: {project_name}

## Problem Summary
{Consolidated problem statement}

## Scope
### In Scope
{List}

### Out of Scope
{List}

## Requirements

### Functional Requirements
| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|---------------------|
| FR-001 | {desc} | MUST | {criteria} |

### Non-Functional Requirements
| ID | Requirement | Measure | Target |
|----|-------------|---------|--------|
| NFR-001 | {desc} | {measure} | {target} |

### Constraints
| ID | Constraint | Type | Rationale |
|----|------------|------|-----------|
| C-001 | {desc} | TECHNICAL | {why} |

## Design Decisions (Already Made)
| ID | Decision | Rationale |
|----|----------|-----------|
| D-001 | {decision} | {why} |

## Innovations
| ID | Innovation | Impact |
|----|------------|--------|
| I-001 | {innovation} | {design impact} |

## Risks to Address
| ID | Risk | Mitigation |
|----|------|------------|
| RISK-001 | {risk} | {mitigation} |

## Use Cases
{Use case specifications}

## Phase 1 Scope
{What's in first release}
```

## Success Criteria
- All requirements extracted and numbered
- Use cases defined
- Decisions documented
- Risks captured
- Ready for domain analysis

## Next Step
→ R02-DOMAIN-ANALYSIS.md
