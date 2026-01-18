# 7S-04: Decisions

## Context
Step: **4 of 7**
Purpose: Make and document key design decisions

## Mindset

**Decisions shape the solution. Make them deliberately.**

Every project requires decisions about:
- Technology choices
- Architectural patterns
- Trade-offs between competing concerns
- Scope prioritization

Document decisions and rationale so future you (and others) understand WHY.

## Input Required
- Scope from 7S-01
- Landscape from 7S-02
- Requirements from 7S-03

## Prompt

```
TASK: Identify and make key design decisions for this project.

INPUT:
{scope_from_7S01}
{landscape_from_7S02}
{requirements_from_7S03}

═══════════════════════════════════════════════════════════════
DECISION IDENTIFICATION
What decisions need to be made?
═══════════════════════════════════════════════════════════════

DECISIONS TO MAKE:

Category: Technology/Platform
  - {decision needed}

Category: Architecture/Design
  - {decision needed}

Category: Features/Scope
  - {decision needed}

Category: Integration
  - {decision needed}

Category: Build vs Buy
  - {decision needed}

═══════════════════════════════════════════════════════════════
DECISION TEMPLATE
For each major decision
═══════════════════════════════════════════════════════════════

DECISION: D-{id}

  Title: {brief description}

  Context:
    {Background and why this decision is needed}

  Decision Statement:
    We will {what we decided}

  Options Considered:

    Option A: {name}
      Description: {what it is}
      Pros:
        + {advantage}
      Cons:
        - {disadvantage}
      Effort: {estimate}
      Risk: {level}

    Option B: {name}
      Description: {what it is}
      Pros:
        + {advantage}
      Cons:
        - {disadvantage}
      Effort: {estimate}
      Risk: {level}

    Option C: {name}
      ...

  Evaluation:
    | Criterion | Weight | Option A | Option B | Option C |
    |-----------|--------|----------|----------|----------|
    | {crit 1}  | {1-3}  | {1-5}    | {1-5}    | {1-5}    |
    | {crit 2}  | {1-3}  | {1-5}    | {1-5}    | {1-5}    |
    | Weighted  |        | {total}  | {total}  | {total}  |

  Chosen Option: {A, B, or C}

  Rationale:
    {Why this option was chosen}

  Trade-offs Accepted:
    {What we're giving up}

  Consequences:
    {What this means for the project}

  Reversibility: [EASY | HARD | IRREVERSIBLE]

  Status: [PROPOSED | DECIDED | DEPRECATED]

═══════════════════════════════════════════════════════════════
BUILD VS BUY DECISION
Final decision on creating vs adopting
═══════════════════════════════════════════════════════════════

DECISION: D-BUILD-BUY

Based on landscape analysis:

BUILD:
  Effort: {estimate}
  Control: Full
  Customization: Full
  Risk: {level}
  Justification: {why build makes sense}

BUY/ADOPT: {specific solution}
  Effort: {estimate}
  Fit: {percent}%
  Gaps: {what's missing}
  Risk: {level}
  Justification: {why adopt makes sense}

ADAPT: {base solution}
  Effort: {estimate}
  Base fit: {percent}%
  Changes needed: {list}
  Risk: {level}
  Justification: {why adapt makes sense}

DECISION: {BUILD | BUY | ADAPT}
  Rationale: {why}

═══════════════════════════════════════════════════════════════
ARCHITECTURE DECISIONS
High-level design choices
═══════════════════════════════════════════════════════════════

DECISION: D-ARCH-001 - Overall Architecture

  Pattern chosen: {FACADE | LAYERED | PLUGIN | etc}
  Rationale: {why this pattern}
  Implications: {what this means}

DECISION: D-ARCH-002 - Class Structure

  Approach: {approach description}
  Key classes: {list}
  Rationale: {why this structure}

DECISION: D-ARCH-003 - Extensibility

  Mechanism: {how extended}
  Extension points: {where}
  Rationale: {why this approach}

═══════════════════════════════════════════════════════════════
TECHNOLOGY DECISIONS
Technical choices
═══════════════════════════════════════════════════════════════

DECISION: D-TECH-001 - Dependencies

  External dependencies:
    - {dependency}: {why needed}

  Simple ecosystem:
    - {simple_*}: {why used}

  Rationale: {overall approach to dependencies}

DECISION: D-TECH-002 - Genericity

  Use generics: [YES | NO | PARTIALLY]
  Where: {which classes}
  Rationale: {why this approach}

═══════════════════════════════════════════════════════════════
FEATURE DECISIONS
What's in, what's out
═══════════════════════════════════════════════════════════════

DECISION: D-FEAT-001 - Phase 1 Scope

  In Phase 1:
    - FR-001: {requirement}
    - FR-002: {requirement}

  Deferred to Phase 2:
    - FR-005: {requirement}

  Rationale: {why this split}

DECISION: D-FEAT-002 - Feature X

  Include: [YES | NO]
  Rationale: {why}
  If NO, alternative: {what instead}

═══════════════════════════════════════════════════════════════
TRADE-OFF DECISIONS
Balancing competing concerns
═══════════════════════════════════════════════════════════════

TRADE-OFF: T-001

  Competing concerns:
    - {Concern A}: wants {X}
    - {Concern B}: wants {Y}

  Conflict: {why they conflict}

  Resolution:
    Favor: {A or B}
    Rationale: {why}
    Mitigation for unfavored: {how to reduce impact}

═══════════════════════════════════════════════════════════════
DEFERRED DECISIONS
What we're NOT deciding now
═══════════════════════════════════════════════════════════════

DEFERRED: DD-001
  Topic: {what}
  Why defer: {reason}
  Decide by: {when or what trigger}
  Default if not decided: {fallback}

═══════════════════════════════════════════════════════════════
DECISION LOG
Summary of all decisions
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

# DECISIONS: {project_name}

## Decision Summary
| ID | Decision | Option Chosen | Reversibility |
|----|----------|---------------|---------------|
| D-001 | {title} | {option} | EASY |

## Key Decisions

### D-001: {Title}
**Context**: {why needed}

**Options Considered**:
- Option A: {name} - {brief assessment}
- Option B: {name} - {brief assessment}

**Decision**: {what was decided}

**Rationale**: {why}

**Consequences**: {what this means}

---

### D-002: {Title}
...

## Build vs Buy
**Decision**: {BUILD | BUY | ADAPT}
**Rationale**: {explanation}

## Architecture Decisions
{Architecture decision records}

## Technology Decisions
{Technology decision records}

## Feature Decisions
{Scope decisions}

## Trade-offs
| Trade-off | Favored | Unfavored | Mitigation |
|-----------|---------|-----------|------------|
| {T-001} | {concern} | {concern} | {action} |

## Deferred Decisions
| ID | Topic | Decide By | Default |
|----|-------|-----------|---------|
| DD-001 | {topic} | {when} | {fallback} |
```

## Success Criteria
- All key decisions documented
- Options fairly evaluated
- Rationale clearly stated
- Trade-offs acknowledged
- Consequences understood

## Next Step
→ 7S-05-INNOVATIONS.md
