# 7S-02: Landscape

## Context
Step: **2 of 7**
Purpose: Survey what already exists in this problem space

## Mindset

**Don't reinvent the wheel. Understand what's already been built.**

Before designing a solution, know:
- What solutions already exist?
- What are their strengths and weaknesses?
- What patterns have emerged?
- What mistakes have been made?

Learn from others' work, whether to adopt, adapt, or avoid.

## Input Required
- Scope definition from 7S-01
- Access to search/research tools

## Prompt

```
TASK: Research existing solutions in this problem space.

INPUT:
{scope_definition_from_7S01}

═══════════════════════════════════════════════════════════════
EXISTING SOLUTIONS INVENTORY
What's already out there?
═══════════════════════════════════════════════════════════════

For each existing solution:

SOLUTION: {name}
  Type: [LIBRARY | FRAMEWORK | TOOL | SERVICE | STANDARD]
  Language/Platform: {what it's built in/for}
  URL: {where to find it}
  Maturity: [MATURE | GROWING | EXPERIMENTAL | DEPRECATED]
  Popularity: {metrics if available}
  Active: [YES | NO]
  License: {license type}

SOLUTIONS FOUND:
| Name | Type | Platform | Maturity | License |
|------|------|----------|----------|---------|
| {name} | {type} | {platform} | {maturity} | {license} |

═══════════════════════════════════════════════════════════════
SOLUTION ANALYSIS
Deep dive on each solution
═══════════════════════════════════════════════════════════════

For each relevant solution:

ANALYSIS: {solution_name}

  PURPOSE:
    What it does: {description}
    Problem it solves: {specific problem}

  STRENGTHS:
    + {strength 1}
    + {strength 2}
    + {strength 3}

  WEAKNESSES:
    - {weakness 1}
    - {weakness 2}
    - {weakness 3}

  FEATURES:
    Core features:
      - {feature 1}
      - {feature 2}

    Notable features:
      - {feature 3}

    Missing features (for our needs):
      - {missing 1}
      - {missing 2}

  ARCHITECTURE:
    Pattern: {architectural pattern}
    Key abstractions: {main concepts}

  API SAMPLE:
```{language}
{code sample showing how it's used}
```

  ADOPTION:
    Users: {who uses it}
    Use cases: {common applications}

  RELEVANCE TO US:
    Addresses our needs: {percent}%
    Could adopt: [YES | NO | PARTIAL]
    Could adapt: [YES | NO]
    Learn from: {what we can learn}

═══════════════════════════════════════════════════════════════
PATTERN IDENTIFICATION
What patterns have emerged?
═══════════════════════════════════════════════════════════════

COMMON PATTERNS:

Pattern: {name}
  Seen in: {solutions}
  Description: {what it is}
  Purpose: {why it's used}
  Should adopt: [YES | NO | ADAPT]

ANTI-PATTERNS:

Anti-Pattern: {name}
  Seen in: {solutions}
  Problem: {why it's bad}
  Avoid by: {how to prevent}

═══════════════════════════════════════════════════════════════
COMPARATIVE ANALYSIS
How do solutions compare?
═══════════════════════════════════════════════════════════════

COMPARISON MATRIX:

| Feature | Solution A | Solution B | Solution C |
|---------|------------|------------|------------|
| {feature 1} | ✓ | ✓ | ✗ |
| {feature 2} | ✓ | ✗ | ✓ |
| {feature 3} | ✗ | ✓ | ✓ |

SCORES:

| Criterion | Solution A | Solution B | Solution C |
|-----------|------------|------------|------------|
| Completeness | {1-5} | {1-5} | {1-5} |
| Ease of use | {1-5} | {1-5} | {1-5} |
| Performance | {1-5} | {1-5} | {1-5} |
| Documentation | {1-5} | {1-5} | {1-5} |
| Maintenance | {1-5} | {1-5} | {1-5} |
| **Total** | {sum} | {sum} | {sum} |

═══════════════════════════════════════════════════════════════
EIFFEL-SPECIFIC RESEARCH
What exists in the Eiffel ecosystem?
═══════════════════════════════════════════════════════════════

EIFFEL SOLUTIONS:

ISE Libraries:
  - {library}: {relevance}

Simple Eiffel Ecosystem:
  - {simple_*}: {relevance}

GOBO:
  - {library}: {relevance}

Third-party Eiffel:
  - {library}: {relevance}

GAPS:
  Not available in Eiffel: {what's missing}
  Would require: {effort to create}

═══════════════════════════════════════════════════════════════
BUILD VS BUY VS ADAPT
Initial assessment of our options
═══════════════════════════════════════════════════════════════

OPTION 1: BUILD (Create from scratch)
  Effort: {HIGH | MEDIUM | LOW}
  Risk: {HIGH | MEDIUM | LOW}
  Benefit: {what we gain}
  Drawback: {what we lose}

OPTION 2: BUY/ADOPT (Use existing solution)
  Candidate: {solution name}
  Fit: {how well it fits}
  Gaps: {what's missing}
  Effort to integrate: {estimate}

OPTION 3: ADAPT (Modify existing)
  Base: {what to start from}
  Changes needed: {modifications}
  Effort: {estimate}
  Licensing: {issues if any}

INITIAL RECOMMENDATION: {BUILD | BUY | ADAPT}
  Rationale: {why}

═══════════════════════════════════════════════════════════════
LESSONS LEARNED
What can we learn from existing solutions?
═══════════════════════════════════════════════════════════════

DO:
  - {practice to adopt}
  - {pattern to use}
  - {approach to follow}

DON'T:
  - {mistake to avoid}
  - {anti-pattern to skip}
  - {approach that failed}

OPEN QUESTIONS:
  - {question raised by research}
  - {uncertainty needing resolution}

═══════════════════════════════════════════════════════════════
REFERENCES
Sources consulted
═══════════════════════════════════════════════════════════════

DOCUMENTATION:
  - {url}: {what we learned}

REPOSITORIES:
  - {url}: {relevant code}

ARTICLES:
  - {url}: {key insight}

DISCUSSIONS:
  - {url}: {community perspective}

OUTPUT FORMAT:

# LANDSCAPE ANALYSIS: {project_name}

## Summary
{Executive summary of landscape}

## Existing Solutions

### {Solution 1}
| Aspect | Assessment |
|--------|------------|
| Maturity | {level} |
| Relevance | {percent}% |
| Recommendation | {adopt/adapt/avoid} |

**Strengths**: {list}
**Weaknesses**: {list}

### {Solution 2}
...

## Comparison Matrix
{Feature comparison table}

## Patterns Identified
| Pattern | Description | Adopt? |
|---------|-------------|--------|
| {pattern} | {desc} | YES/NO |

## Eiffel Ecosystem
| Library | Relevance | Usable? |
|---------|-----------|---------|
| {lib} | {relevance} | YES/NO |

## Build vs Buy vs Adapt
| Option | Effort | Risk | Fit |
|--------|--------|------|-----|
| Build | {level} | {level} | {percent}% |
| Buy | {level} | {level} | {percent}% |
| Adapt | {level} | {level} | {percent}% |

**Initial Recommendation**: {option}

## Lessons Learned
### Do
{List}

### Don't
{List}

## References
{List of sources}
```

## Success Criteria
- At least 3 alternatives researched
- Strengths/weaknesses documented
- Patterns identified
- Comparison completed
- Initial build/buy/adapt assessment

## Next Step
→ 7S-03-REQUIREMENTS.md

CRITICAL: STOP here and tell me what the next step is and prompt me to tell you to continue with my added instruction.
