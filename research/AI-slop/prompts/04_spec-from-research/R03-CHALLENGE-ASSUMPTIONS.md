# R03: Challenge Assumptions

## Context
Phase: **ANALYSIS** (3 of 3)
Purpose: Question everything before committing to design

## Mindset

**Attack the research mercilessly.**

Research often contains:
- Implicit assumptions never questioned
- Requirements that aren't actually necessary
- Decisions made without full information
- Over-engineering or under-engineering

Challenge EVERYTHING before designing. It's cheaper to fix requirements than fix code.

## Input Required
- Parsed research from R01
- Domain model from R02

## Prompt

```
TASK: Challenge all assumptions, requirements, and decisions from the research.

INPUT:
{parsed_research_from_R01}
{domain_model_from_R02}

═══════════════════════════════════════════════════════════════
ASSUMPTION EXTRACTION
What does the research assume?
═══════════════════════════════════════════════════════════════

Read through all research and extract implicit assumptions:

ASSUMPTION: A-{id}
  Location: {where in research}
  Statement: {what is assumed}
  Evidence: {is there evidence}
  Questioned: [YES | NO]

ASSUMPTIONS FOUND:
  About users: {list}
  About technology: {list}
  About requirements: {list}
  About constraints: {list}

═══════════════════════════════════════════════════════════════
ASSUMPTION CHALLENGE
Are these assumptions valid?
═══════════════════════════════════════════════════════════════

For each assumption:

CHALLENGE: A-{id}
  Assumption: {statement}

  Questions:
    - Is this always true? {analysis}
    - What if it's false? {consequence}
    - How would we know if false? {detection}

  Verdict: [VALID | QUESTIONABLE | INVALID]
  Action: {keep | modify | remove}

INVALID ASSUMPTIONS:
  A-{id}: {why invalid} → {action}

QUESTIONABLE ASSUMPTIONS:
  A-{id}: {concern} → {how to resolve}

═══════════════════════════════════════════════════════════════
REQUIREMENT NECESSITY
Are all requirements truly necessary?
═══════════════════════════════════════════════════════════════

For each MUST requirement:

NECESSITY CHECK: FR-{id}
  Requirement: {statement}

  Challenge:
    - What happens if we don't do this? {consequence}
    - Is this really a MUST or actually a SHOULD? {analysis}
    - Who demanded this and why? {source}
    - Is there a simpler way? {alternative}

  Verdict: [KEEP | DOWNGRADE | REMOVE | SIMPLIFY]

REQUIREMENT CHANGES:
  FR-{id}: {original priority} → {new priority} because {reason}

═══════════════════════════════════════════════════════════════
COMPLETENESS CHECK
What requirements are MISSING?
═══════════════════════════════════════════════════════════════

Consider what wasn't mentioned:

MISSING REQUIREMENTS:

Error handling:
  - What happens when {operation} fails? {missing FR}
  - How are errors reported? {missing FR}

Edge cases:
  - What about empty input? {missing FR}
  - What about maximum size? {missing FR}

Security:
  - Who can do what? {missing FR}
  - How is input validated? {missing FR}

Operations:
  - How is it deployed? {missing NFR}
  - How is it monitored? {missing NFR}

PROPOSED NEW REQUIREMENTS:
  FR-NEW-{id}: {requirement}
    Rationale: {why needed}
    Priority: {MUST | SHOULD}

═══════════════════════════════════════════════════════════════
DECISION CHALLENGE
Were the right decisions made?
═══════════════════════════════════════════════════════════════

For each decision from research:

DECISION CHALLENGE: D-{id}
  Decision: {what was decided}
  Alternatives considered: {list}

  Questions:
    - Was this the right choice? {analysis}
    - Were alternatives fairly evaluated? {analysis}
    - What are the long-term implications? {implications}
    - Is this decision still valid? {current validity}

  Verdict: [AFFIRM | RECONSIDER | REVERSE]

DECISIONS TO RECONSIDER:
  D-{id}: {concern} → {recommended action}

═══════════════════════════════════════════════════════════════
SCOPE CHALLENGE
Is the scope right?
═══════════════════════════════════════════════════════════════

SCOPE TOO LARGE?
  Features that could be cut:
    - {feature}: {why cuttable}

  Impact of cutting: {what we lose}

SCOPE TOO SMALL?
  Features that should be added:
    - {feature}: {why needed}

  Impact of adding: {what we gain}

SCOPE VERDICT:
  Current scope: [TOO_LARGE | ABOUT_RIGHT | TOO_SMALL]
  Recommended changes: {list}

═══════════════════════════════════════════════════════════════
RISK CHALLENGE
Were risks properly identified?
═══════════════════════════════════════════════════════════════

MISSING RISKS:
  RISK-NEW-{id}: {risk}
    Why missed: {explanation}
    Mitigation: {proposed}

OVERBLOWN RISKS:
  RISK-{id}: {why not as serious}
    Downgrade to: {new level}

UNDERESTIMATED RISKS:
  RISK-{id}: {why more serious}
    Upgrade to: {new level}
    Additional mitigation: {what}

═══════════════════════════════════════════════════════════════
DOMAIN MODEL CHALLENGE
Is the domain model correct?
═══════════════════════════════════════════════════════════════

CONCEPT CHALLENGES:

CONCEPT: {concept}
  Challenge: Is this the right abstraction?
  Alternative: {different way to model}
  Verdict: [KEEP | CHANGE]

RELATIONSHIP CHALLENGES:

RELATIONSHIP: {A} → {B}
  Challenge: Is this the right relationship?
  Alternative: {different relationship}
  Verdict: [KEEP | CHANGE]

═══════════════════════════════════════════════════════════════
EIFFEL-SPECIFIC CHALLENGES
Will this work well in Eiffel?
═══════════════════════════════════════════════════════════════

Consider Eiffel constraints:

VOID SAFETY:
  - Can everything be non-void? {analysis}
  - What must be detachable? {list}

SCOOP COMPATIBILITY:
  - Any shared mutable state? {analysis}
  - Any thread-unsafe patterns? {list}

CONTRACT FEASIBILITY:
  - Can requirements be expressed as contracts? {analysis}
  - Any unverifiable requirements? {list}

LIBRARY COMPATIBILITY:
  - What simple_* libraries are available? {list}
  - Any conflicts? {analysis}

═══════════════════════════════════════════════════════════════
CHALLENGE SUMMARY
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

# ASSUMPTION CHALLENGE REPORT: {project_name}

## Summary
- Assumptions challenged: {count}
- Invalid assumptions found: {count}
- Requirements questioned: {count}
- Requirements changed: {count}
- Missing requirements found: {count}
- Decisions to reconsider: {count}

## Invalid Assumptions
{List with actions}

## Requirement Changes

### Removed/Downgraded
| ID | Original | Change | Reason |
|----|----------|--------|--------|
| FR-X | MUST | SHOULD | {reason} |

### Added
| ID | Requirement | Priority | Reason |
|----|-------------|----------|--------|
| FR-NEW-1 | {desc} | MUST | {reason} |

## Decisions to Reconsider
| ID | Decision | Concern | Recommendation |
|----|----------|---------|----------------|
| D-X | {decision} | {concern} | {action} |

## Missing Risks
| ID | Risk | Mitigation |
|----|------|------------|
| RISK-NEW-1 | {risk} | {mitigation} |

## Scope Recommendations
{Changes to scope}

## Domain Model Adjustments
{Changes to domain model}

## Eiffel Considerations
{Technical notes for design phase}

## Final Requirements (Revised)
{Updated requirements list}
```

## Success Criteria
- All assumptions identified and challenged
- Requirements validated or changed
- Missing requirements identified
- Decisions affirmed or flagged
- Ready for design with confidence

## Next Step
→ R04-CLASS-DESIGN.md
