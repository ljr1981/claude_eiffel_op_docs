# 7S-07: Recommendation

## Context
Step: **7 of 7**
Purpose: Synthesize all research into a clear recommendation

## Mindset

**After research comes action. Make a clear recommendation.**

This final step:
- Synthesizes all previous research
- Makes a clear GO/NO-GO recommendation
- Defines next steps if proceeding
- Provides executive summary for stakeholders

## Input Required
- All previous research steps (7S-01 through 7S-06)

## Prompt

```
TASK: Synthesize all research into a final recommendation.

INPUT:
{scope_from_7S01}
{landscape_from_7S02}
{requirements_from_7S03}
{decisions_from_7S04}
{innovations_from_7S05}
{risks_from_7S06}

═══════════════════════════════════════════════════════════════
EXECUTIVE SUMMARY
One-page summary for stakeholders
═══════════════════════════════════════════════════════════════

PROJECT: {name}

PURPOSE:
  {One-sentence description of what we're building and why}

RECOMMENDATION: [PROCEED | PROCEED WITH CAUTION | DO NOT PROCEED]

KEY FINDINGS:
  1. {Most important finding}
  2. {Second most important finding}
  3. {Third most important finding}

KEY RISKS:
  1. {Top risk and mitigation}
  2. {Second risk and mitigation}

INVESTMENT:
  Effort: {estimate}
  Timeline: {estimate}
  Resources: {what's needed}

EXPECTED RETURN:
  {What we gain from this investment}

═══════════════════════════════════════════════════════════════
RESEARCH SYNTHESIS
Summary of all research phases
═══════════════════════════════════════════════════════════════

SCOPE (from 7S-01):
  Problem: {problem statement}
  Users: {target users}
  Success criteria: {key criteria}

LANDSCAPE (from 7S-02):
  Alternatives found: {count}
  Best alternative: {name}
  Our differentiation: {how we're different}

REQUIREMENTS (from 7S-03):
  MUST have: {count}
  SHOULD have: {count}
  Key non-functional: {most important NFRs}

DECISIONS (from 7S-04):
  Build/Buy/Adapt: {decision}
  Architecture: {chosen pattern}
  Key trade-offs: {what we're trading}

INNOVATIONS (from 7S-05):
  Unique value: {our differentiation}
  Novel approaches: {count}
  Eiffel advantages: {how we leverage Eiffel}

RISKS (from 7S-06):
  Critical risks: {count}
  Overall risk level: {level}
  Top mitigation: {most important mitigation}

═══════════════════════════════════════════════════════════════
GO/NO-GO ASSESSMENT
Should we proceed?
═══════════════════════════════════════════════════════════════

GO FACTORS (reasons to proceed):
  + {strong reason 1}
  + {strong reason 2}
  + {strong reason 3}

NO-GO FACTORS (reasons not to proceed):
  - {concern 1}
  - {concern 2}

ASSESSMENT:

| Factor | Weight | Score | Weighted |
|--------|--------|-------|----------|
| Problem value | 3 | {1-5} | {total} |
| Solution viability | 3 | {1-5} | {total} |
| Competitive advantage | 2 | {1-5} | {total} |
| Risk level | 3 | {1-5} | {total} |
| Resource availability | 2 | {1-5} | {total} |
| Strategic fit | 2 | {1-5} | {total} |
| **Total** | 15 | | **{sum}** |

SCORE INTERPRETATION:
  60-75: Strong GO
  45-59: GO with conditions
  30-44: Proceed with caution
  15-29: NO-GO or major rethink needed

FINAL SCORE: {total}/75

═══════════════════════════════════════════════════════════════
RECOMMENDATION
What should we do?
═══════════════════════════════════════════════════════════════

PRIMARY RECOMMENDATION: [PROCEED | PROCEED WITH CONDITIONS | DEFER | ABANDON]

RECOMMENDATION STATEMENT:
  {Clear statement of what to do}

RATIONALE:
  {Why this recommendation}

CONDITIONS (if applicable):
  - {condition that must be met}
  - {condition that must be met}

ALTERNATIVES CONSIDERED:
  Alternative 1: {what}
    Why not: {reason}

  Alternative 2: {what}
    Why not: {reason}

═══════════════════════════════════════════════════════════════
IMPLEMENTATION ROADMAP
If proceeding, what's next?
═══════════════════════════════════════════════════════════════

PHASE 1: Foundation
  Duration: {estimate}
  Deliverables:
    - {deliverable 1}
    - {deliverable 2}
  Key milestone: {what marks completion}

PHASE 2: Core Features
  Duration: {estimate}
  Deliverables:
    - {deliverable 1}
    - {deliverable 2}
  Key milestone: {what marks completion}

PHASE 3: Extended Features
  Duration: {estimate}
  Deliverables:
    - {deliverable 1}
  Key milestone: {what marks completion}

ROADMAP VISUALIZATION:
```
Phase 1: Foundation     Phase 2: Core        Phase 3: Extended
├──────────────────────┼───────────────────┼──────────────────┤
|████████████████████||█████████████████||████████████████|
├──────────────────────┼───────────────────┼──────────────────┤
Milestone: {M1}        Milestone: {M2}     Milestone: {M3}
```

═══════════════════════════════════════════════════════════════
IMMEDIATE NEXT STEPS
What to do right now
═══════════════════════════════════════════════════════════════

NEXT STEPS:

1. {Immediate action 1}
   Owner: {who}
   Deadline: {when}
   Output: {what's produced}

2. {Immediate action 2}
   Owner: {who}
   Deadline: {when}
   Output: {what's produced}

3. {Immediate action 3}
   ...

DECISION REQUIRED:
  Decision: {what needs to be decided}
  By whom: {decision maker}
  By when: {deadline}
  Options: {choices}

═══════════════════════════════════════════════════════════════
RESOURCE REQUIREMENTS
What's needed to proceed
═══════════════════════════════════════════════════════════════

PEOPLE:
  Role: {role}
    Skills: {required skills}
    Effort: {time commitment}

TOOLS:
  - {tool 1}: {why needed}
  - {tool 2}: {why needed}

DEPENDENCIES:
  - {dependency 1}: {status}
  - {dependency 2}: {status}

BUDGET (if applicable):
  {Cost breakdown}

═══════════════════════════════════════════════════════════════
SUCCESS METRICS
How we'll know if we succeeded
═══════════════════════════════════════════════════════════════

METRICS:

| Metric | Target | Measurement |
|--------|--------|-------------|
| {metric 1} | {target} | {how measured} |
| {metric 2} | {target} | {how measured} |

CHECKPOINTS:
  Week 2: {what should be done}
  Week 4: {what should be done}
  Week 8: {what should be done}

═══════════════════════════════════════════════════════════════
APPENDIX REFERENCES
Links to detailed research
═══════════════════════════════════════════════════════════════

RESEARCH DOCUMENTS:
  - STEP-1-SCOPE.md
  - STEP-2-LANDSCAPE.md
  - STEP-3-REQUIREMENTS.md
  - STEP-4-DECISIONS.md
  - STEP-5-INNOVATIONS.md
  - STEP-6-RISKS.md

SUPPORTING MATERIALS:
  - {additional documents}

OUTPUT FORMAT:

# RECOMMENDATION: {project_name}

## Executive Summary

**Recommendation**: {PROCEED | PROCEED WITH CONDITIONS | DEFER | ABANDON}

{One paragraph summary}

## Key Findings
1. {Finding}
2. {Finding}
3. {Finding}

## Key Risks
1. {Risk}: {Mitigation}
2. {Risk}: {Mitigation}

## Go/No-Go Score
**{score}/75** - {interpretation}

## Recommendation Details

### What We Should Do
{Clear recommendation}

### Why
{Rationale}

### Conditions (if applicable)
{List}

## Roadmap
{Phase diagram}

## Next Steps
| Action | Owner | Deadline |
|--------|-------|----------|
| {action} | {who} | {when} |

## Resources Required
{Summary}

## Success Metrics
| Metric | Target |
|--------|--------|
| {metric} | {target} |

## Appendix
{Links to detailed research}

---

*Research completed: {date}*
*Ready for: Spec-from-Research workflow*
```

## Success Criteria
- Clear GO/NO-GO recommendation
- Rationale is evidence-based
- Next steps are actionable
- Resources identified
- Success metrics defined

## Research Complete

The 7-Step Research workflow is now complete.

If PROCEED recommended:
→ Feed output to Spec-from-Research workflow (R01-R08)

If DEFER recommended:
→ Document what conditions would trigger re-evaluation

If ABANDON recommended:
→ Document learnings for future reference

CRITICAL: STOP here and tell me what the next step is and prompt me to tell you to continue with my added instruction.
