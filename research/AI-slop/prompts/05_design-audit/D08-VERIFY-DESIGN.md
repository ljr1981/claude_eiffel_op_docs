# D08: Verify Design

## Context
Phase: **REFACTORING** (4 of 4)
Purpose: Validate improvements and document the new design

## Mindset

**Prove the design is better. Document for the future.**

After refactoring:
1. Verify all changes are correct
2. Measure the improvement
3. Document the new design
4. Create guidelines to maintain quality

## Input Required
- All previous reports (D01-D07)
- Final source code
- Test suite

## Prompt

```
TASK: Verify all design improvements and create final documentation.

INPUT:
{all_previous_reports}
{final_source_files}
{test_suite}

═══════════════════════════════════════════════════════════════
REGRESSION TESTING
Verify existing functionality still works
═══════════════════════════════════════════════════════════════

Execute full test suite:

TEST RESULTS:
  Original tests: {pass}/{total}
  New tests: {pass}/{total}
  Total: {pass}/{total}

REGRESSIONS:
  [List any failures and root cause]

All tests pass: [YES | NO]

═══════════════════════════════════════════════════════════════
SMELL RE-SCAN
Verify smells are eliminated
═══════════════════════════════════════════════════════════════

Re-run smell detection from D02:

BEFORE vs AFTER:

| Smell | Before | After | Eliminated |
|-------|--------|-------|------------|
| God Class | {n} | {n} | {n} |
| Feature Envy | {n} | {n} | {n} |
| Long Param List | {n} | {n} | {n} |
| Data Clumps | {n} | {n} | {n} |
| Primitive Obsession | {n} | {n} | {n} |
| Dead Code | {n} | {n} | {n} |

SMELL REDUCTION: {percent}%

═══════════════════════════════════════════════════════════════
INHERITANCE RE-AUDIT
Verify inheritance is correct
═══════════════════════════════════════════════════════════════

Re-run inheritance audit from D03:

BEFORE vs AFTER:

| Metric | Before | After |
|--------|--------|-------|
| LSP Violations | {n} | {n} |
| Refused Bequest | {n} | {n} |
| Implementation-only inheritance | {n} | {n} |
| Max depth | {n} | {n} |

All inheritance correct: [YES | NO]

═══════════════════════════════════════════════════════════════
GENERICITY RE-SCAN
Verify genericity is applied
═══════════════════════════════════════════════════════════════

Re-run genericity scan from D04:

BEFORE vs AFTER:

| Metric | Before | After |
|--------|--------|-------|
| Generic classes | {n} | {n} |
| Duplicate classes | {n} | {n} |
| ANY usages | {n} | {n} |
| Type casts | {n} | {n} |

Genericity improvement: {percent}%

═══════════════════════════════════════════════════════════════
METRICS COMPARISON
Measure overall improvement
═══════════════════════════════════════════════════════════════

STRUCTURAL METRICS:

| Metric | Before | After | Target | Met |
|--------|--------|-------|--------|-----|
| Classes | {n} | {n} | - | - |
| Avg features/class | {n} | < 20 | {n} | [Y/N] |
| Max inheritance depth | {n} | < 4 | {n} | [Y/N] |
| Generic classes % | {n}% | > 30% | {n}% | [Y/N] |
| Code duplication | {n}% | < 5% | {n}% | [Y/N] |

QUALITY METRICS:

| Metric | Before | After | Improved |
|--------|--------|-------|----------|
| Test coverage | {n}% | {n}% | [Y/N] |
| Coupling (avg) | {n} | {n} | [Y/N] |
| Cohesion (avg) | {n} | {n} | [Y/N] |

═══════════════════════════════════════════════════════════════
DESIGN PRINCIPLES AUDIT
Verify OOSC2 principles followed
═══════════════════════════════════════════════════════════════

PRINCIPLE: Abstraction
  Before: {assessment}
  After: {assessment}
  Improved: [YES | NO]

PRINCIPLE: Information Hiding
  Before: {assessment}
  After: {assessment}
  Improved: [YES | NO]

PRINCIPLE: Modularity
  Before: {assessment}
  After: {assessment}
  Improved: [YES | NO]

PRINCIPLE: Reusability
  Before: {assessment}
  After: {assessment}
  Improved: [YES | NO]

PRINCIPLE: Extensibility
  Before: {assessment}
  After: {assessment}
  Improved: [YES | NO]

═══════════════════════════════════════════════════════════════
NEW DESIGN DOCUMENTATION
Document the improved design
═══════════════════════════════════════════════════════════════

CLASS DIAGRAM (ASCII):
```
┌─────────────────────────────────────────────────────────────┐
│                    NEW ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [FACADE] ─────────────────────────────────────────────┐   │
│      │                                                  │   │
│      ├── [ENGINE] ──> [RESULT]                         │   │
│      │                    │                            │   │
│      │                    └── [ITEM] (generic)         │   │
│      │                                                  │   │
│      └── [HELPER] [G -> CONSTRAINT]                    │   │
│                                                         │   │
└─────────────────────────────────────────────────────────────┘
```

CLASS RESPONSIBILITIES:
  - {CLASS_1}: {single responsibility}
  - {CLASS_2}: {single responsibility}
  - {CLASS_3}: {single responsibility}

INHERITANCE HIERARCHY:
```
  DEFERRED_BASE
       │
       ├── CONCRETE_A
       │
       └── CONCRETE_B
```

GENERICITY USAGE:
  - {GENERIC_CLASS} [G]: used for {purpose}
  - {GENERIC_CLASS} [G -> CONSTRAINT]: used for {purpose}

═══════════════════════════════════════════════════════════════
DESIGN GUIDELINES
Rules to maintain the improved design
═══════════════════════════════════════════════════════════════

GUIDELINES FOR THIS CODEBASE:

1. INHERITANCE RULES:
   - Only inherit for true IS-A relationships
   - Use composition for HAS-A
   - Max inheritance depth: {n}

2. CLASS SIZE RULES:
   - Max features per class: {n}
   - Extract class if responsibility grows

3. GENERICITY RULES:
   - Prefer generic classes over type-specific
   - Use constrained genericity when calling features on G
   - Avoid ANY in type signatures

4. NAMING RULES:
   - {specific conventions}

5. CONTRACT RULES:
   - All public features have contracts
   - Use invariants for consistency

═══════════════════════════════════════════════════════════════
FUTURE REFACTORING OPPORTUNITIES
What could be improved next?
═══════════════════════════════════════════════════════════════

REMAINING ISSUES (Not addressed this cycle):
  - {issue}: {why not addressed} - {future recommendation}

POTENTIAL IMPROVEMENTS:
  - {improvement}: {benefit}

TECHNICAL DEBT REMAINING:
  - {debt}: {priority}

═══════════════════════════════════════════════════════════════
FINAL ARTIFACTS
═══════════════════════════════════════════════════════════════

FILES CREATED DURING AUDIT:
  - design-audit/STRUCTURE-MAP.md
  - design-audit/SMELL-REPORT.md
  - design-audit/INHERITANCE-AUDIT.md
  - design-audit/GENERICITY-REPORT.md
  - design-audit/REFACTOR-PLAN.md
  - design-audit/EXTRACTION-LOG.md
  - design-audit/GENERICITY-LOG.md
  - design-audit/DESIGN-REVIEW.md

SOURCE FILES CREATED:
  - {list of new class files}

SOURCE FILES MODIFIED:
  - {list of changed files}

SOURCE FILES DELETED:
  - {list of removed files}

OUTPUT FORMAT:

# DESIGN AUDIT FINAL REPORT: {library_name}

## Executive Summary

The Design Audit workflow has been completed on {library_name}.

**Key Improvements:**
- Smells eliminated: {count} ({percent}%)
- Inheritance issues fixed: {count}
- Generic classes introduced: {count}
- Code duplication removed: {percent}%
- Classes refactored: {count}

**Assessment:** The codebase now follows OOSC2 principles more closely.

## Metrics Comparison

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Classes | {n} | {n} | {±n} |
| Avg features/class | {n} | {n} | {-n} |
| Generic classes | {n} | {n} | {+n} |
| Smells | {n} | {n} | {-n} |
| LSP violations | {n} | {n} | {-n} |

## Design Principles Score

| Principle | Before | After |
|-----------|--------|-------|
| Abstraction | {1-5} | {1-5} |
| Information Hiding | {1-5} | {1-5} |
| Modularity | {1-5} | {1-5} |
| Reusability | {1-5} | {1-5} |
| Extensibility | {1-5} | {1-5} |
| **Average** | **{n}** | **{n}** |

## Changes Made

### New Classes
{List with purpose}

### Modified Classes
{List with changes}

### Deleted Classes
{List with replacement}

## New Design

{Class diagram}

{Responsibility descriptions}

## Guidelines
{Design rules for this codebase}

## Remaining Work
{Future improvements}

## Certification

This codebase has completed the Design Audit workflow.

- Audited by: {AI model}
- Date: {date}
- Original smells: {n}
- Final smells: {n}
- Improvement: {percent}%

The design is certified as **IMPROVED** following OOSC2 principles.
```

## Success Criteria
- All tests pass
- Smells reduced
- Metrics improved
- Documentation complete
- Guidelines established

## Workflow Complete

The Design Audit workflow is now complete. The codebase has been:
1. Analyzed for structure
2. Scanned for smells
3. Audited for inheritance
4. Scanned for genericity opportunities
5. Planned for refactoring
6. Refactored with extractions
7. Generified where appropriate
8. Verified and documented

The code now follows better object-oriented design principles.
