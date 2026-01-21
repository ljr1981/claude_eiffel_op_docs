# 7S-01: Scope

## Context
Step: **1 of 7**
Purpose: Define what we're researching and why

## Mindset

**A problem well-defined is half-solved.**

Before researching, we must know:
- What problem are we solving?
- Who has this problem?
- What does success look like?
- What's definitely NOT in scope?

Without clear scope, research wanders aimlessly.

## Input Required
- Initial idea description
- Context and motivation
- Any known constraints

## Prompt

```
TASK: Define the scope and boundaries for this research effort.

INPUT:
IDEA: {one-sentence description}
CONTEXT: {why we want this}
CONSTRAINTS: {known limitations}

═══════════════════════════════════════════════════════════════
PROBLEM STATEMENT
What problem are we solving?
═══════════════════════════════════════════════════════════════

STATE THE PROBLEM:

In one sentence:
  {The problem is...}

In detail:
  What's wrong today: {current pain}
  Who experiences this: {affected users}
  How often: {frequency}
  What's the impact: {consequence of not solving}

PROBLEM VALIDATION:
  Is this a real problem? {evidence}
  Is it worth solving? {justification}
  Has it been solved before? {preliminary check}

═══════════════════════════════════════════════════════════════
TARGET USERS
Who are we building for?
═══════════════════════════════════════════════════════════════

PRIMARY USERS:
  User type: {description}
  Needs: {what they need}
  Current solution: {how they cope today}
  Pain level: [HIGH | MEDIUM | LOW]

SECONDARY USERS:
  User type: {description}
  Needs: {what they need}

NON-USERS (explicitly):
  Not for: {who this is NOT for}
  Reason: {why excluded}

═══════════════════════════════════════════════════════════════
SUCCESS CRITERIA
How will we know we've succeeded?
═══════════════════════════════════════════════════════════════

SUCCESS DEFINED:

Minimum success (MVP):
  - {criterion 1}: {measurable outcome}
  - {criterion 2}: {measurable outcome}

Full success:
  - {criterion 3}: {measurable outcome}
  - {criterion 4}: {measurable outcome}

Stretch goals:
  - {criterion 5}: {measurable outcome}

ANTI-SUCCESS (what would be failure):
  - {failure criterion}: {what we must avoid}

═══════════════════════════════════════════════════════════════
SCOPE BOUNDARIES
What's in and what's out?
═══════════════════════════════════════════════════════════════

IN SCOPE:
  Core capabilities (MUST):
    - {capability 1}
    - {capability 2}

  Extended capabilities (SHOULD):
    - {capability 3}
    - {capability 4}

OUT OF SCOPE:
  Explicitly excluded:
    - {excluded 1}: {reason}
    - {excluded 2}: {reason}

  Deferred to future:
    - {future 1}: {why later}
    - {future 2}: {why later}

SCOPE BOUNDARY QUESTIONS:
  {Specific questions about what's in/out that need resolution}

═══════════════════════════════════════════════════════════════
CONSTRAINTS
What limits our solution space?
═══════════════════════════════════════════════════════════════

TECHNICAL CONSTRAINTS:
  Platform: {Eiffel, specific versions}
  Dependencies: {what we must/can't use}
  Integration: {what we must work with}

RESOURCE CONSTRAINTS:
  Time: {deadline if any}
  Team: {who's available}
  Skills: {what expertise exists}

DESIGN CONSTRAINTS:
  Must follow: {patterns, standards}
  Must avoid: {anti-patterns}
  Compatibility: {what must it work with}

═══════════════════════════════════════════════════════════════
USE CASES (High-Level)
What will users do with this?
═══════════════════════════════════════════════════════════════

PRIMARY USE CASES:

UC-1: {Use case name}
  Actor: {who}
  Goal: {what they want to accomplish}
  Typical scenario: {brief description}

UC-2: {Use case name}
  ...

EDGE CASES TO CONSIDER:
  - {edge case 1}
  - {edge case 2}

═══════════════════════════════════════════════════════════════
ASSUMPTIONS
What are we assuming to be true?
═══════════════════════════════════════════════════════════════

ASSUMPTIONS:
  A-1: {assumption}
    Evidence: {support or "assumed"}
    Risk if false: {consequence}

  A-2: {assumption}
    ...

ASSUMPTIONS TO VALIDATE IN RESEARCH:
  - {assumption that needs verification}

═══════════════════════════════════════════════════════════════
RESEARCH QUESTIONS
What do we need to learn?
═══════════════════════════════════════════════════════════════

QUESTIONS TO ANSWER:

About the problem:
  - {question 1}
  - {question 2}

About existing solutions:
  - {question 3}
  - {question 4}

About our approach:
  - {question 5}
  - {question 6}

About feasibility:
  - {question 7}

═══════════════════════════════════════════════════════════════
STAKEHOLDERS
Who cares about this?
═══════════════════════════════════════════════════════════════

STAKEHOLDERS:
  {Stakeholder}: {interest}
  {Stakeholder}: {interest}

DECISION MAKERS:
  {Who decides to proceed}

INFORMATION SOURCES:
  {Who has knowledge we need}

OUTPUT FORMAT:

# SCOPE DEFINITION: {project_name}

## Problem Statement
{Clear problem description}

## Target Users
| User Type | Needs | Pain Level |
|-----------|-------|------------|
| {type} | {needs} | HIGH |

## Success Criteria
| Level | Criterion | Measure |
|-------|-----------|---------|
| MVP | {criterion} | {measure} |
| Full | {criterion} | {measure} |

## Scope

### In Scope
- {capability}

### Out of Scope
- {excluded}: {reason}

### Deferred
- {future item}

## Constraints
| Type | Constraint |
|------|------------|
| Technical | {constraint} |
| Resource | {constraint} |

## High-Level Use Cases
{Brief use case descriptions}

## Assumptions
| ID | Assumption | Needs Validation |
|----|------------|------------------|
| A-1 | {assumption} | YES/NO |

## Research Questions
{Questions to answer in subsequent steps}

## Stakeholders
{Who's involved}
```

## Success Criteria
- Problem clearly articulated
- Users identified
- Scope boundaries defined
- Success criteria measurable
- Research questions identified

## Next Step
→ 7S-02-LANDSCAPE.md

CRITICAL: STOP here and tell me what the next step is and prompt me to tell you to continue with my added instruction.
