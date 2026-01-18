# Bug Hunting Workflow: Finding Unknown Bugs in Eiffel Code

## CRITICAL: ANTI-SLOP ENFORCEMENT

**Before using this workflow, read: `../ANTI-SLOP-RULES.md`**

This workflow requires ACTUAL:
- Code reading (using Read tool, not imagination)
- Test writing and execution (compile, run, paste output)
- Bug reproduction (actual failing test case)

**FORBIDDEN**:
- Claiming "this code might have bugs" without evidence
- Writing tests without compiling and running them
- Documenting bugs without actual reproduction

---

## Overview

8 sequential prompts for systematically hunting bugs in an existing Eiffel codebase when you don't know what's wrong—proactive defect detection.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      SURVEY PHASE                               │
├─────────────────────────────────────────────────────────────────┤
│  H01-SURVEY-RISK-AREAS      Identify high-risk code regions   │
│           ↓                                                     │
│  H02-ANALYZE-SEMANTICS      Contract/code/test/spec alignment │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      PROBE PHASE                                │
├─────────────────────────────────────────────────────────────────┤
│  H03-PROBE-EDGE-CASES       Boundary values, domain limits    │
│           ↓                                                     │
│  H04-PROBE-STATE-SEQUENCES  Operation ordering bugs           │
│           ↓                                                     │
│  H05-PROBE-CONCURRENCY      SCOOP/threading issues            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      VERIFY PHASE                               │
├─────────────────────────────────────────────────────────────────┤
│  H06-CONSTRUCT-EXPLOIT      Build failing test case           │
│           ↓                                                     │
│  H07-CONFIRM-ROOT-CAUSE     Trace to actual defect            │
│           ↓                                                     │
│  H08-DOCUMENT-FINDING       Record bug for fixing             │
└─────────────────────────────────────────────────────────────────┘
```

## The Holistic View: Where Bugs Hide

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPECIFICATION LAYER                          │
│  • ABSENT: No spec exists for this behavior                    │
│  • MALFORMED: Spec contradicts domain or is ambiguous          │
└─────────────────────────────────────────────────────────────────┘
                              ↓ derived from
┌─────────────────────────────────────────────────────────────────┐
│                      CONTRACT LAYER                             │
│  • ABSENT: No contracts guard this feature                     │
│  • MALFORMED: Contract contradicts spec or domain semantics    │
└─────────────────────────────────────────────────────────────────┘
                              ↓ implemented by
┌─────────────────────────────────────────────────────────────────┐
│                        CODE LAYER                               │
│  • MALFORMED: Code contradicts contract or domain semantics    │
│  • (Absent code = missing feature, not bug)                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓ verified by
┌─────────────────────────────────────────────────────────────────┐
│                        TEST LAYER                               │
│  • ABSENT: No test covers this scenario                        │
│  • MALFORMED: Test expects wrong behavior or tests wrong thing │
└─────────────────────────────────────────────────────────────────┘
```

## Key Principles

**Bugs hide where SPECIFICATIONS are absent.**
No documented intent means no source of truth for correctness.

**Bugs hide where SPECIFICATIONS are malformed.**
Specs that contradict domain rules or are internally inconsistent.

**Bugs hide where CONTRACTS are absent.**
The absence of a precondition means callers can pass invalid data.
The absence of a postcondition means the feature can silently fail.

**Bugs hide where CONTRACTS are malformed.**
Contracts antithetical to the purpose of the feature within the
semantic context of the class or of the class within the system.

**Bugs hide where CODE is malformed.**
Class feature code that is antithetical to the semantic context
and problem domain.

**Bugs hide where TESTS are absent.**
Scenarios that are never exercised remain unknown.

**Bugs hide where TESTS are malformed.**
Tests that assert wrong expectations or test irrelevant properties
give false confidence.

**Bugs hide in ALL OF THE ABOVE.**
The most dangerous bugs occur when multiple layers are misaligned:
spec says X, contract says Y, code does Z, test expects W.

## Prompt Files

| # | File | Phase | Purpose |
|---|------|-------|---------|
| H01 | SURVEY-RISK-AREAS | Survey | Find high-risk code (all layers) |
| H02 | ANALYZE-SEMANTICS | Survey | Spec/contract/code/test alignment |
| H03 | PROBE-EDGE-CASES | Probe | Boundary testing |
| H04 | PROBE-STATE-SEQUENCES | Probe | Operation ordering |
| H05 | PROBE-CONCURRENCY | Probe | SCOOP/threading issues |
| H06 | CONSTRUCT-EXPLOIT | Verify | Build failing test |
| H07 | CONFIRM-ROOT-CAUSE | Verify | Trace to defect layer |
| H08 | DOCUMENT-FINDING | Verify | Record for fixing |

## Malformation Taxonomy

| Layer | Absent | Malformed |
|-------|--------|-----------|
| Specification | No documented intent | Contradicts domain, ambiguous |
| Contract | No require/ensure/invariant | Wrong semantics for feature |
| Code | (Missing feature) | Violates domain rules |
| Test | No coverage | Wrong expectations |

## MANDATORY: Execution Gates

**H06 (Construct Exploit) MUST include**:

```bash
# 1. Write the test file
[Edit/Write tool]

# 2. COMPILE
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# 3. RUN and observe failure
./EIFGENs/{lib}_tests/W_code/{lib}.exe

# 4. PASTE ACTUAL OUTPUT showing the bug
```

**H08 (Document Finding) is ONLY valid if**:
- Actual test code exists in `testing/` folder
- Actual compilation output pasted
- Actual test failure output pasted

## Verification Checkpoint Format

After H06, H07, H08:

```markdown
## BUG VERIFICATION - [Bug Description]

### Exploit Test
```eiffel
[ACTUAL test code that triggers the bug]
```

### Compilation
```
[PASTE ACTUAL ec.sh OUTPUT]
```

### Execution
```
[PASTE ACTUAL test output showing failure]
```

### Bug Confirmed: YES/NO
### Root Layer: [Spec|Contract|Code|Test]
```

**If you cannot paste actual test failure, the bug is NOT confirmed.**
