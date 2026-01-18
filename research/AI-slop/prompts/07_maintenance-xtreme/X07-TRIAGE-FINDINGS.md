# X07: Triage Findings

## Context
Phase: **FIX** (1 of 4)
Purpose: Prioritize all discovered issues before fixing

## Mindset

**Fix the right bugs in the right order.**

After the attack phase, you have a pile of findings:
- Contract failures from X03
- Test failures from X04
- Stress failures from X05
- Surviving mutations from X06

Before fixing anything, prioritize ruthlessly. Not all bugs are equal.

## Input Required
- Contract assault report (X03)
- Adversarial test report (X04)
- Stress attack report (X05)
- Mutation warfare report (X06)

## Prompt

```
TASK: Consolidate and prioritize all findings from attack phases.

INPUT:
{contract_assault_report_X03}
{adversarial_test_report_X04}
{stress_attack_report_X05}
{mutation_warfare_report_X06}

═══════════════════════════════════════════════════════════════
FINDING CONSOLIDATION
Gather all findings into one list
═══════════════════════════════════════════════════════════════

From X03 Contract Assault:
  - {BUG-001}: {description}
  - {BUG-002}: {description}

From X04 Adversarial Tests:
  - {BUG-003}: {description}
  - {BUG-004}: {description}

From X05 Stress Attack:
  - {BUG-005}: {description}
  - {LIMIT-001}: {description}

From X06 Mutation Warfare:
  - {WEAKNESS-001}: {description}
  - {WEAKNESS-002}: {description}

TOTAL FINDINGS: {count}

═══════════════════════════════════════════════════════════════
FINDING CLASSIFICATION
Categorize each finding
═══════════════════════════════════════════════════════════════

For each finding:

FINDING: {id}
  Description: {what was found}
  Source: [CONTRACT | TEST | STRESS | MUTATION]
  Category:
    [ ] CRASH: Code crashes/exceptions
    [ ] WRONG: Code produces wrong results
    [ ] CORRUPT: Code corrupts state
    [ ] LEAK: Code leaks resources
    [ ] LIMIT: Code has undocumented limits
    [ ] WEAK: Code lacks proper protection
    [ ] SLOW: Code has performance issues

═══════════════════════════════════════════════════════════════
SEVERITY ASSESSMENT
How bad is each finding?
═══════════════════════════════════════════════════════════════

SEVERITY CRITERIA:

CRITICAL:
  - Crashes in normal use
  - Data corruption possible
  - Security vulnerability
  - No workaround exists

HIGH:
  - Wrong results in common cases
  - Resource leaks
  - Significant limitation
  - Workaround is difficult

MEDIUM:
  - Wrong results in edge cases
  - Performance issues
  - Missing protection
  - Workaround exists

LOW:
  - Cosmetic issues
  - Minor inefficiency
  - Theoretical problems
  - Easy workaround

For each finding:

FINDING: {id}
  Severity: [CRITICAL | HIGH | MEDIUM | LOW]
  Justification: {why this severity}

═══════════════════════════════════════════════════════════════
LIKELIHOOD ASSESSMENT
How likely is this to occur?
═══════════════════════════════════════════════════════════════

LIKELIHOOD:
  - CERTAIN: Will happen in normal use
  - LIKELY: Common usage patterns trigger it
  - POSSIBLE: Specific conditions trigger it
  - UNLIKELY: Rare circumstances only

For each finding:

FINDING: {id}
  Likelihood: [CERTAIN | LIKELY | POSSIBLE | UNLIKELY]
  Trigger condition: {what causes it}

═══════════════════════════════════════════════════════════════
PRIORITY MATRIX
Combine severity and likelihood
═══════════════════════════════════════════════════════════════

                    LIKELIHOOD
                CERTAIN  LIKELY  POSSIBLE  UNLIKELY
           ┌────────────────────────────────────────┐
  CRITICAL │   P1       P1        P2         P2    │
  HIGH     │   P1       P2        P2         P3    │
  MEDIUM   │   P2       P2        P3         P3    │
  LOW      │   P3       P3        P4         P4    │
           └────────────────────────────────────────┘

P1 = Fix immediately
P2 = Fix before release
P3 = Fix when convenient
P4 = Document, fix later or never

For each finding:

FINDING: {id}
  Severity: {severity}
  Likelihood: {likelihood}
  Priority: P{1-4}

═══════════════════════════════════════════════════════════════
DEPENDENCY ANALYSIS
What must be fixed first?
═══════════════════════════════════════════════════════════════

Some fixes depend on others:

DEPENDENCY: {finding_A} blocks {finding_B}
  Reason: {why A must be fixed first}

Fix order constraints:
  1. {finding} - no dependencies
  2. {finding} - depends on #1
  ...

═══════════════════════════════════════════════════════════════
EFFORT ESTIMATION
How hard is each fix?
═══════════════════════════════════════════════════════════════

EFFORT:
  - XS: One-line change
  - S: Few lines, localized
  - M: Multiple locations, moderate complexity
  - L: Significant changes, complex
  - XL: Architectural change needed

For each finding:

FINDING: {id}
  Effort: [XS | S | M | L | XL]
  Location(s): {where to fix}
  Approach: {how to fix}

═══════════════════════════════════════════════════════════════
RISK ASSESSMENT
What could go wrong fixing this?
═══════════════════════════════════════════════════════════════

FIX RISK:
  - LOW: Simple, isolated change
  - MEDIUM: Could affect related functionality
  - HIGH: Could break existing behavior

For each finding:

FINDING: {id}
  Fix risk: [LOW | MEDIUM | HIGH]
  Risk: {what could go wrong}
  Mitigation: {how to minimize risk}

═══════════════════════════════════════════════════════════════
FINAL TRIAGE
Ordered fix list
═══════════════════════════════════════════════════════════════

Sort by: Priority (P1→P4), then Effort (XS→XL)

OUTPUT FORMAT:

# TRIAGE REPORT: {library_name}

## Finding Summary
- Total findings: {count}
- P1 (Critical): {count}
- P2 (High): {count}
- P3 (Medium): {count}
- P4 (Low): {count}

## Priority 1 Findings (Fix Immediately)

### {FINDING-ID}: {title}
| Attribute | Value |
|-----------|-------|
| Source | {X03/X04/X05/X06} |
| Category | {CRASH/WRONG/etc} |
| Severity | CRITICAL |
| Likelihood | CERTAIN |
| Effort | {XS-XL} |
| Risk | {LOW-HIGH} |
| Location | {class.feature} |
| Description | {what's wrong} |
| Fix approach | {how to fix} |

## Priority 2 Findings (Fix Before Release)
{Same format}

## Priority 3 Findings (Fix When Convenient)
{Same format}

## Priority 4 Findings (Document Only)
{Same format}

## Fix Order
Based on dependencies and effort:

1. {FINDING-ID}: {reason for position}
2. {FINDING-ID}: {reason for position}
...

## Deferred Findings
These will NOT be fixed (with justification):
- {FINDING}: {why not fixing}

## Metrics
- Total bugs: {count}
- To fix: {count}
- To defer: {count}
- Estimated total effort: {XS+S+M+L+XL}
```

## Success Criteria
- All findings consolidated
- Each finding prioritized
- Fix order determined
- Effort estimated
- Risks identified

## Next Step
→ X08-SURGICAL-FIXES.md
