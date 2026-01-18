# 7S-06: Risks

## Context
Step: **6 of 7**
Purpose: Identify risks and plan mitigations

## Mindset

**What could go wrong WILL go wrong unless we plan for it.**

Every project has risks:
- Technical risks (it might not work)
- Schedule risks (it might take too long)
- Resource risks (we might not have what we need)
- Market risks (nobody might want it)

Identify risks early when you can still do something about them.

## Input Required
- All previous research steps
- Honest assessment of uncertainties

## Prompt

```
TASK: Identify all significant risks and plan mitigations.

INPUT:
{all_previous_research}

═══════════════════════════════════════════════════════════════
RISK IDENTIFICATION
What could go wrong?
═══════════════════════════════════════════════════════════════

RISK CATEGORIES:

Technical Risks:
  - {risk}
  - {risk}

Schedule Risks:
  - {risk}
  - {risk}

Resource Risks:
  - {risk}
  - {risk}

Scope Risks:
  - {risk}
  - {risk}

External Risks:
  - {risk}
  - {risk}

Innovation Risks:
  - {risk}
  - {risk}

═══════════════════════════════════════════════════════════════
RISK ANALYSIS TEMPLATE
For each significant risk
═══════════════════════════════════════════════════════════════

RISK: R-{id}

  Title: {brief description}

  Category: [TECHNICAL | SCHEDULE | RESOURCE | SCOPE | EXTERNAL | INNOVATION]

  Description:
    {What could go wrong}

  Trigger:
    {What would cause this risk to materialize}

  Likelihood: [HIGH | MEDIUM | LOW]
    Evidence: {why this assessment}

  Impact: [HIGH | MEDIUM | LOW]
    If it happens: {consequences}

  Risk Score: {Likelihood × Impact}

  Early Warning Signs:
    - {indicator that risk is materializing}

  Mitigation Strategies:

    Prevention (reduce likelihood):
      - {action to prevent}

    Contingency (reduce impact):
      - {action if it happens}

    Acceptance (live with it):
      - {why acceptable}

  Chosen mitigation: {which strategy}

  Owner: {who's responsible}

  Status: [IDENTIFIED | MITIGATING | OCCURRED | CLOSED]

═══════════════════════════════════════════════════════════════
TECHNICAL RISKS
Risks related to technology and implementation
═══════════════════════════════════════════════════════════════

RISK: R-TECH-001 - {Technical risk}
  Description: {what could go wrong technically}
  Likelihood: {level}
  Impact: {level}
  Mitigation: {how to address}

TECHNICAL RISK SUMMARY:
| ID | Risk | L | I | Score | Mitigation |
|----|------|---|---|-------|------------|
| R-TECH-001 | {desc} | H | H | 9 | {action} |

═══════════════════════════════════════════════════════════════
SCHEDULE RISKS
Risks related to timeline
═══════════════════════════════════════════════════════════════

RISK: R-SCHED-001 - {Schedule risk}
  Description: {what could delay us}
  Likelihood: {level}
  Impact: {level}
  Mitigation: {how to address}

SCHEDULE RISK SUMMARY:
| ID | Risk | L | I | Score | Mitigation |
|----|------|---|---|-------|------------|
| R-SCHED-001 | {desc} | M | M | 4 | {action} |

═══════════════════════════════════════════════════════════════
RESOURCE RISKS
Risks related to people, skills, tools
═══════════════════════════════════════════════════════════════

RISK: R-RES-001 - {Resource risk}
  Description: {what resource might we lack}
  Likelihood: {level}
  Impact: {level}
  Mitigation: {how to address}

═══════════════════════════════════════════════════════════════
SCOPE RISKS
Risks related to requirements and scope
═══════════════════════════════════════════════════════════════

RISK: R-SCOPE-001 - {Scope risk}
  Description: {how scope could change problematically}
  Likelihood: {level}
  Impact: {level}
  Mitigation: {how to address}

═══════════════════════════════════════════════════════════════
EXTERNAL RISKS
Risks outside our control
═══════════════════════════════════════════════════════════════

RISK: R-EXT-001 - {External risk}
  Description: {what external factor could affect us}
  Likelihood: {level}
  Impact: {level}
  Mitigation: {how to address}

═══════════════════════════════════════════════════════════════
INNOVATION RISKS
Risks specific to our novel approaches
═══════════════════════════════════════════════════════════════

RISK: R-INNOV-001 - Innovation I-{id} might not work
  Innovation: {which innovation}
  What could fail: {specific failure mode}
  Likelihood: {level}
  Impact: {level}
  Mitigation:
    - Proof of concept: {validate early}
    - Fallback: {alternative if it fails}

═══════════════════════════════════════════════════════════════
RISK MATRIX
Prioritize risks by likelihood and impact
═══════════════════════════════════════════════════════════════

RISK MATRIX:

                    IMPACT
               LOW     MEDIUM    HIGH
          ┌─────────┬─────────┬─────────┐
     HIGH │         │ R-003   │ R-001   │
          ├─────────┼─────────┼─────────┤
L    MED  │ R-005   │ R-004   │ R-002   │
          ├─────────┼─────────┼─────────┤
     LOW  │ R-007   │ R-006   │         │
          └─────────┴─────────┴─────────┘

PRIORITY ORDER:
  1. R-001: High-High (Critical)
  2. R-002: Medium-High (Major)
  3. R-003: High-Medium (Major)
  4. R-004: Medium-Medium (Moderate)
  ...

═══════════════════════════════════════════════════════════════
MITIGATION PLAN
Actions to reduce risk
═══════════════════════════════════════════════════════════════

MITIGATION ACTIONS:

| Risk | Action | When | Owner | Status |
|------|--------|------|-------|--------|
| R-001 | {action} | {timing} | {who} | PLANNED |
| R-002 | {action} | {timing} | {who} | IN_PROGRESS |

═══════════════════════════════════════════════════════════════
CONTINGENCY PLANS
What to do if risks materialize
═══════════════════════════════════════════════════════════════

CONTINGENCY: R-001 Materialized

  Trigger: {how we know it happened}

  Immediate actions:
    1. {first action}
    2. {second action}

  Escalation: {who to inform}

  Recovery plan: {how to get back on track}

═══════════════════════════════════════════════════════════════
RISK MONITORING
How to track risks over time
═══════════════════════════════════════════════════════════════

MONITORING SCHEDULE:

| Risk | Indicator | Check Frequency | Threshold |
|------|-----------|-----------------|-----------|
| R-001 | {metric} | Weekly | {limit} |

RISK REVIEW:
  Frequency: {how often}
  Attendees: {who}
  Output: {what's produced}

═══════════════════════════════════════════════════════════════
OVERALL RISK ASSESSMENT
Should we proceed?
═══════════════════════════════════════════════════════════════

RISK SUMMARY:
  Total risks: {count}
  Critical (H-H): {count}
  Major (H-M, M-H): {count}
  Moderate (M-M, etc): {count}
  Minor: {count}

OVERALL RISK LEVEL: [HIGH | MEDIUM | LOW]

PROCEED RECOMMENDATION: [YES | YES WITH CAUTION | NO]

Rationale: {why}

CONDITIONS FOR PROCEEDING:
  - {condition 1}
  - {condition 2}

OUTPUT FORMAT:

# RISK ANALYSIS: {project_name}

## Risk Summary
| Level | Count | Examples |
|-------|-------|----------|
| Critical | {n} | R-001, R-002 |
| Major | {n} | R-003 |
| Moderate | {n} | R-004, R-005 |
| Minor | {n} | R-006 |

**Overall Risk Level**: {level}
**Proceed Recommendation**: {recommendation}

## Risk Matrix
{Visual matrix}

## Critical Risks

### R-001: {Title}
| Aspect | Assessment |
|--------|------------|
| Likelihood | HIGH |
| Impact | HIGH |
| Score | 9 |

**Description**: {what}
**Mitigation**: {action}
**Contingency**: {if it happens}

## Major Risks
{Similar format}

## Moderate Risks
{Brief summary}

## Mitigation Plan
| Risk | Action | When | Owner |
|------|--------|------|-------|
| R-001 | {action} | {when} | {who} |

## Contingency Plans
{Plans for critical risks}

## Monitoring
{How risks will be tracked}

## Proceed Conditions
{Requirements to proceed}
```

## Success Criteria
- All significant risks identified
- Likelihood and impact assessed
- Mitigations planned
- Contingencies prepared
- Overall risk level assessed

## Next Step
→ 7S-07-RECOMMENDATION.md
