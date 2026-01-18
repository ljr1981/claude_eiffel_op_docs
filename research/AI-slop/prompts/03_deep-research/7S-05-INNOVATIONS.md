# 7S-05: Innovations

## Context
Step: **5 of 7**
Purpose: Identify what's novel or different about our approach

## Mindset

**If we're not doing anything new, why are we building this?**

Every project should have something innovative:
- A new approach to an old problem
- A better design than existing solutions
- A unique combination of features
- An improved user experience

If there's nothing new, reconsider whether to build at all.

## Input Required
- Landscape from 7S-02 (what exists)
- Requirements from 7S-03 (what we need)
- Decisions from 7S-04 (what we chose)

## Prompt

```
TASK: Identify and articulate what's innovative about our approach.

INPUT:
{landscape_from_7S02}
{requirements_from_7S03}
{decisions_from_7S04}

═══════════════════════════════════════════════════════════════
DIFFERENTIATION ANALYSIS
How are we different from existing solutions?
═══════════════════════════════════════════════════════════════

Compare our planned approach to existing solutions:

VS {EXISTING_SOLUTION_1}:
  They do: {what}
  We do differently: {how we're different}
  Why better: {value of the difference}

VS {EXISTING_SOLUTION_2}:
  They do: {what}
  We do differently: {how we're different}
  Why better: {value of the difference}

DIFFERENTIATION SUMMARY:
| Aspect | Existing | Ours | Improvement |
|--------|----------|------|-------------|
| {aspect} | {how they do} | {how we do} | {why better} |

═══════════════════════════════════════════════════════════════
INNOVATION IDENTIFICATION
What's genuinely new?
═══════════════════════════════════════════════════════════════

For each potential innovation:

INNOVATION: I-{id}

  Title: {brief name}

  Type: [APPROACH | DESIGN | FEATURE | COMBINATION | UX]

  Description:
    {What's new or different}

  Problem solved:
    {What problem this addresses}

  Novelty assessment:
    New to world: [YES | NO]
    New to Eiffel: [YES | NO]
    New to Simple ecosystem: [YES | NO]
    New combination: [YES | NO]

  Evidence of novelty:
    {How we know this is new}

  Value:
    {Why this matters}

  Risks:
    {What could go wrong with this innovation}

═══════════════════════════════════════════════════════════════
APPROACH INNOVATIONS
New ways of solving the problem
═══════════════════════════════════════════════════════════════

APPROACH INNOVATION: I-APPROACH-001

  Traditional approach:
    {How others solve this}

  Our approach:
    {How we're solving it differently}

  Why different:
    {What insight led to this}

  Benefits:
    + {benefit}
    + {benefit}

  Potential drawbacks:
    - {risk}

═══════════════════════════════════════════════════════════════
DESIGN INNOVATIONS
Novel architectural or design patterns
═══════════════════════════════════════════════════════════════

DESIGN INNOVATION: I-DESIGN-001

  Standard design:
    {Common design approach}

  Our design:
    {Our different design}

  Novelty:
    {What's new about this design}

  Eiffel-specific advantage:
    {How Eiffel enables this}

═══════════════════════════════════════════════════════════════
FEATURE INNOVATIONS
Unique features not found elsewhere
═══════════════════════════════════════════════════════════════

FEATURE INNOVATION: I-FEAT-001

  Feature: {feature name}

  Not found in:
    - {solution A}: {why they don't have it}
    - {solution B}: {why they don't have it}

  Our implementation:
    {How we're doing it}

  Value:
    {Why users will benefit}

═══════════════════════════════════════════════════════════════
COMBINATION INNOVATIONS
Novel combinations of existing ideas
═══════════════════════════════════════════════════════════════

COMBINATION INNOVATION: I-COMBO-001

  Combined elements:
    - {Element A} from {source}
    - {Element B} from {source}
    - {Element C} from {source}

  Novel combination:
    {How combining these is new}

  Synergy:
    {Why the combination is greater than sum of parts}

═══════════════════════════════════════════════════════════════
EIFFEL-SPECIFIC INNOVATIONS
Taking advantage of Eiffel's unique features
═══════════════════════════════════════════════════════════════

EIFFEL INNOVATION: I-EIFFEL-001

  Eiffel feature used:
    {Design by Contract | Genericity | Multiple Inheritance | etc}

  Innovation:
    {How we're using it in a novel way}

  Not possible in:
    {Languages where this wouldn't work}

  Benefit:
    {What Eiffel gives us}

═══════════════════════════════════════════════════════════════
VALUE PROPOSITION
Why should anyone use our solution?
═══════════════════════════════════════════════════════════════

VALUE PROPOSITION:

For: {target users}
Who need: {their problem}
Our solution: {what we provide}
Unlike: {alternatives}
Provides: {key differentiator}

UNIQUE SELLING POINTS:
  1. {USP 1}
  2. {USP 2}
  3. {USP 3}

═══════════════════════════════════════════════════════════════
INNOVATION VALIDATION
Are these innovations real and valuable?
═══════════════════════════════════════════════════════════════

For each innovation:

VALIDATION: I-{id}

  Is it really new?
    Evidence: {proof}
    Confidence: [HIGH | MEDIUM | LOW]

  Is it really valuable?
    To whom: {beneficiaries}
    How much: {magnitude of benefit}
    Confidence: [HIGH | MEDIUM | LOW]

  Is it achievable?
    Technical feasibility: [HIGH | MEDIUM | LOW]
    Effort: {estimate}

  Risk assessment:
    Innovation risk: {what could fail}
    Mitigation: {how to reduce risk}

INNOVATION VIABILITY:
| Innovation | Novel | Valuable | Achievable | Viable |
|------------|-------|----------|------------|--------|
| I-001 | HIGH | HIGH | HIGH | ✓ |
| I-002 | HIGH | MEDIUM | LOW | ? |

═══════════════════════════════════════════════════════════════
INNOVATION PROTECTION
How do we maintain our advantage?
═══════════════════════════════════════════════════════════════

COMPETITIVE MOAT:
  Why can't others copy: {barriers}
  How long advantage lasts: {estimate}
  How to extend advantage: {strategy}

OUTPUT FORMAT:

# INNOVATIONS: {project_name}

## Innovation Summary
| ID | Innovation | Type | Novelty | Value |
|----|------------|------|---------|-------|
| I-001 | {title} | APPROACH | HIGH | HIGH |

## Value Proposition
For {users} who need {solution}, our {product} provides {value} unlike {alternatives} because {differentiator}.

## Unique Selling Points
1. {USP}
2. {USP}
3. {USP}

## Key Innovations

### I-001: {Title}
**Type**: {type}
**Description**: {what's new}
**Problem Solved**: {why it matters}
**Validation**: {evidence it's novel and valuable}

### I-002: {Title}
...

## Differentiation from Alternatives
| Aspect | {Alt A} | {Alt B} | Ours |
|--------|---------|---------|------|
| {aspect} | {how} | {how} | {how + why better} |

## Eiffel Advantages
{How we leverage Eiffel's unique features}

## Innovation Risks
| Innovation | Risk | Mitigation |
|------------|------|------------|
| I-001 | {risk} | {mitigation} |

## Competitive Moat
{How we maintain advantage}
```

## Success Criteria
- At least 1-3 genuine innovations identified
- Novelty validated
- Value articulated
- Risks acknowledged
- Clear differentiation from alternatives

## Next Step
→ 7S-06-RISKS.md
