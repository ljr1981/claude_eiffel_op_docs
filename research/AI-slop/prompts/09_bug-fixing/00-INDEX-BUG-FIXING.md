# Bug Fixing Workflow: Correcting Known Bugs in Eiffel Code

## CRITICAL: ANTI-SLOP ENFORCEMENT

**Before using this workflow, read: `../ANTI-SLOP-RULES.md`**

This workflow requires ACTUAL:
- Bug reproduction (compile, run, observe failure)
- Fix implementation (edit code)
- Verification (compile, run tests, paste output)

**FORBIDDEN**: Describing what "would happen" without actually doing it.

---

## Overview

7 sequential prompts for systematically fixing known bugs in an existing Eiffel codebase when the bug is already identified and specified.

## The Holistic View: Bug Fixes Touch All Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPECIFICATION LAYER                          │
│  May need: Add missing spec, fix malformed spec                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      CONTRACT LAYER                             │
│  May need: Add missing contract, fix malformed contract        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        CODE LAYER                               │
│  May need: Fix malformed code                                  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        TEST LAYER                               │
│  ALWAYS need: Add regression test, maybe fix malformed test    │
└─────────────────────────────────────────────────────────────────┘
```

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     ANALYZE PHASE                               │
├─────────────────────────────────────────────────────────────────┤
│  F01-REPRODUCE-BUG          Confirm bug exists as described   │
│           ↓                                                     │
│  F02-CLASSIFY-FIX-LAYERS    Which layers need fixing?         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                       FIX PHASE                                 │
│              (Top-down: Spec → Contract → Code → Test)          │
├─────────────────────────────────────────────────────────────────┤
│  F03-FIX-SPEC-CONTRACT      Fix spec and/or contract          │
│           ↓                                                     │
│  F04-FIX-CODE               Fix implementation                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     VALIDATE PHASE                              │
├─────────────────────────────────────────────────────────────────┤
│  F05-VERIFY-FIX             Original bug no longer triggers   │
│           ↓                                                     │
│  F06-VERIFY-NO-REGRESSION   Existing tests still pass         │
│           ↓                                                     │
│  F07-ADD-REGRESSION-TEST    Prevent bug from returning        │
└─────────────────────────────────────────────────────────────────┘
```

## Prompt Files

| # | File | Phase | Layer | Purpose |
|---|------|-------|-------|---------|
| F01 | REPRODUCE-BUG | Analyze | All | Confirm bug as described |
| F02 | CLASSIFY-FIX-TYPE | Analyze | All | Identify root layer |
| F03 | ADD-MISSING-CONTRACT | Fix | Spec+Contract | Fix spec and/or contract |
| F04 | FIX-IMPLEMENTATION | Fix | Code | Fix implementation |
| F05 | VERIFY-FIX | Validate | All | Bug no longer triggers |
| F06 | VERIFY-NO-REGRESSION | Validate | All | Nothing else broke |
| F07 | ADD-REGRESSION-TEST | Validate | Test | Strengthen test layer |

**Note:** File names are legacy; content updated for holistic 4-layer approach.

## Key Principles

**Every bug has a root layer.**
The bug's root cause is in ONE layer, but may have been hidden by
absences or malformations in OTHER layers.

**Fix top-down.**
If spec is wrong, fix spec first. Then fix contract to match spec.
Then fix code to satisfy contract. Then fix/add tests to verify.

**Every bug fix MUST include a test.**
A bug fix without a regression test is incomplete—the bug WILL return.

**Malformed tests can hide bugs.**
If a test exists but expects wrong behavior, fixing the code will
"break" the test. The test needs fixing too.

## Classification Matrix

| Root Cause Layer | What to Fix |
|------------------|-------------|
| Spec absent/malformed | Spec → Contract → Code → Test |
| Contract absent/malformed | Contract → Code → Test |
| Code malformed | Code → Test |
| Test absent | Test |
| Test malformed | Test (and verify code is actually correct) |

## MANDATORY: Execution Steps

At EACH step, Claude Code MUST:

```bash
# 1. COMPILE (after any code change)
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# 2. RUN TESTS (to verify fix)
./EIFGENs/{lib}_tests/W_code/{lib}.exe

# 3. PASTE OUTPUT (prove it worked)
[actual output here, not "expected: ..."]
```

## Verification Checkpoint Format

After F01 (reproduce), F04 (fix), F05 (verify), F06 (regression), F07 (test):

```markdown
## VERIFICATION CHECKPOINT

### Command Executed
```
[actual command]
```

### Output
```
[PASTE ACTUAL OUTPUT - not a summary]
```

### Result
- Bug reproduced: YES/NO
- Fix successful: YES/NO
- All tests pass: YES/NO
```

**If you cannot paste actual output, the step is NOT complete.**
