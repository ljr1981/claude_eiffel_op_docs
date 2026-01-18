# Maintenance Workflow: Improving Pre-Existing Eiffel Code

## CRITICAL: ANTI-SLOP ENFORCEMENT

**Before using this workflow, read: `../ANTI-SLOP-RULES.md`**

This workflow requires ACTUAL:
- Code reading (using Read tool for audit)
- Code modification (using Edit tool for strengthening)
- Compilation after each modification
- Test execution to verify changes

**FORBIDDEN**:
- Claiming code "needs improvement" without reading it
- Suggesting contract changes without implementing them
- Documenting improvements without compiling and testing

---

## Overview

10 sequential prompts for systematically improving an existing Eiffel codebase by auditing and strengthening all layers: specifications, contracts, code, and tests.

## The Holistic View

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPECIFICATION LAYER                          │
│  Audit: Is intent documented? Is it correct?                   │
│  Strengthen: Add missing specs, fix malformed specs            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      CONTRACT LAYER                             │
│  Audit: Are contracts present? Are they correct?               │
│  Strengthen: Add missing contracts, fix malformed contracts    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        CODE LAYER                               │
│  Audit: Does code match spec and contracts?                    │
│  Strengthen: Fix code that violates spec/contract/domain       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        TEST LAYER                               │
│  Audit: Is there coverage? Do tests expect correct behavior?   │
│  Strengthen: Add missing tests, fix malformed tests            │
└─────────────────────────────────────────────────────────────────┘
```

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      AUDIT PHASE                                │
│              (Examine all four layers)                          │
├─────────────────────────────────────────────────────────────────┤
│  M01-AUDIT-CONTRACTS        Spec + Contract audit             │
│           ↓                                                     │
│  M02-AUDIT-STRUCTURE        Contract structure                │
│           ↓                                                     │
│  M03-AUDIT-VOID-SAFETY      Code quality + domain alignment   │
│           ↓                                                     │
│  M04-ADD-MISSING-PRE...     Test coverage and correctness     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    STRENGTHEN PHASE                             │
│              (Fix top-down: Spec → Contract)                    │
├─────────────────────────────────────────────────────────────────┤
│  M05-ADD-MISSING-POST...    Fix specifications                │
│           ↓                                                     │
│  M06-ADD-MISSING-INV...     Fix contracts (all types)         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    VALIDATE PHASE                               │
│              (Verify all layers aligned)                        │
├─────────────────────────────────────────────────────────────────┤
│  M07-COMPILE-VALIDATE       Verify build succeeds             │
│           ↓                                                     │
│  M08-TEST-NEW-CONTRACTS     Validate all layers, surface bugs │
└─────────────────────────────────────────────────────────────────┘
```

## Prompt Files

| # | File | Phase | Layer | Purpose |
|---|------|-------|-------|---------|
| M01 | AUDIT-CONTRACTS | Audit | Spec+Contract | Audit specifications AND contracts |
| M02 | AUDIT-STRUCTURE | Audit | Contract | Contract structure and coverage |
| M03 | AUDIT-VOID-SAFETY | Audit | Code | Code quality and domain alignment |
| M04 | ADD-MISSING-PRECONDITIONS | Audit | Test | Audit tests (was preconditions) |
| M05 | ADD-MISSING-POSTCONDITIONS | Strengthen | Spec | Fix specifications |
| M06 | ADD-MISSING-INVARIANTS | Strengthen | Contract | Fix contracts (all types) |
| M07 | COMPILE-VALIDATE | Validate | All | Compile and check |
| M08 | TEST-NEW-CONTRACTS | Validate | All | Validate all layers aligned |

**Note:** File names are legacy; content updated for holistic 4-layer approach.

## Key Principles

**Audit all layers before fixing any.**
Understand the full picture before making changes.

**Fix top-down: Spec → Contract → Code → Test.**
Each layer derives from the one above. Fix the source of truth first.

**Adding correctness surfaces latent bugs.**
Code that "worked" but violates new specs/contracts/tests was always broken.

**Malformed artifacts are worse than absent ones.**
Absent spec = no guidance. Malformed spec = wrong guidance.

## MANDATORY: Execution Gates

**After M01-M04 (Audit Phase)**:
- Read actual source files (paste file paths read)
- Document audit findings with line numbers

**After M05-M06 (Strengthen Phase)**:

```bash
# After EACH modification:
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# Verify:
# "System Recompiled." = change successful
# Any errors = fix before proceeding
```

**M07 (Compile Validate) and M08 (Test) are verification gates**:

```bash
# M07: Full compile
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# M08: Run all tests
./EIFGENs/{lib}_tests/W_code/{lib}.exe
```

## Verification Checkpoint Format

After M05, M06, M07, M08:

```markdown
## MAINTENANCE CHECKPOINT - Step M[N]

### Changes Made
- [file:line] - [description of change]

### Compilation
```
[PASTE ACTUAL ec.sh OUTPUT]
```

### Test Results (M08 only)
```
[PASTE ACTUAL test output]
```

### Status
- Compiles: YES/NO
- Tests pass: [N]/[M]
```

**If you cannot paste actual output, the step is NOT complete.**
