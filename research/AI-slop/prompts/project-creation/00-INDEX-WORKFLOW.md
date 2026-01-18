# Eiffel Anti-Slop Prompt Workflow: New Project Creation

## Overview

10 sequential prompts implementing the "CAN" methodology from EIFFEL-ANTIDOTE-TO-AI-SLOP.md. Each prompt is designed for AI execution within an Eiffel project context.

## The Holistic View: Four Layers of Correctness

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPECIFICATION LAYER                          │
│  What the system should do (natural language, requirements)     │
│  Risk: Absent = no basis for correctness                        │
│  Risk: Malformed = wrong basis propagates everywhere            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      CONTRACT LAYER                             │
│  Machine-checkable intent (require, ensure, invariant)          │
│  Risk: Absent = no runtime verification                         │
│  Risk: Malformed = verifying wrong properties                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        CODE LAYER                               │
│  Implementation (feature bodies)                                │
│  Risk: Malformed = doesn't satisfy spec/contracts/domain        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                        TEST LAYER                               │
│  Verification (test cases)                                      │
│  Risk: Absent = no external validation                          │
│  Risk: Malformed = tests pass but behavior is wrong             │
└─────────────────────────────────────────────────────────────────┘
```

**Key Principle:** Each layer must be PRESENT, CORRECT, and ALIGNED with all other layers.

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPECIFICATION PHASE                          │
│                    (Working Hat: Specification)                 │
├─────────────────────────────────────────────────────────────────┤
│  01-ANALYZE-REQUIREMENTS    Extract spec from requirements      │
│           ↓                 (builds SPECIFICATION LAYER)        │
│  02-DEFINE-CLASS-STRUCTURE  Class skeletons, signatures only   │
│           ↓                                                     │
│  03-WRITE-PRECONDITIONS     require clauses (caller promises)  │
│           ↓                 (builds CONTRACT LAYER)             │
│  04-WRITE-POSTCONDITIONS    ensure clauses (feature promises)  │
│           ↓                                                     │
│  05-WRITE-INVARIANTS        Class-level truth conditions       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   IMPLEMENTATION PHASE                          │
│                   (Working Hat: Implementation)                 │
├─────────────────────────────────────────────────────────────────┤
│  06-IMPLEMENT-FEATURES      Write bodies satisfying contracts  │
│                             (builds CODE LAYER)                 │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    VERIFICATION PHASE                           │
│                    (Working Hat: Verification)                  │
├─────────────────────────────────────────────────────────────────┤
│  07-COMPILE-CHECK           Fix type/void safety errors        │
│           ↓                                                     │
│  08-RUN-CONTRACTS           Analyze runtime violations         │
│           ↓                                                     │
│  09-GENERATE-TESTS          Create spec/contract-based tests   │
│                             (builds TEST LAYER)                 │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     ITERATION PHASE                             │
│                     (Working Hat: All)                          │
├─────────────────────────────────────────────────────────────────┤
│  10-ITERATE-REFINE          Categorize failures by layer       │
│           ↓                 Route to correct phase              │
│      ┌────┴────┐                                                │
│      ↓         ↓                                                │
│   [DONE]    [Return to appropriate layer's step]                │
└─────────────────────────────────────────────────────────────────┘
```

## Prompt Files

| # | File | Phase | Purpose |
|---|------|-------|---------|
| 01 | ANALYZE-REQUIREMENTS | Spec | Extract contractual elements |
| 02 | DEFINE-CLASS-STRUCTURE | Spec | Signatures only, no bodies |
| 03 | WRITE-PRECONDITIONS | Spec | Caller obligations (require) |
| 04 | WRITE-POSTCONDITIONS | Spec | Feature guarantees (ensure) |
| 05 | WRITE-INVARIANTS | Spec | Class-wide truth (invariant) |
| 06 | IMPLEMENT-FEATURES | Impl | Bodies satisfying contracts |
| 07 | COMPILE-CHECK | Verify | Fix compiler errors |
| 08 | RUN-CONTRACTS | Verify | Analyze runtime violations |
| 09 | GENERATE-TESTS | Verify | Contract-based test cases |
| 10 | ITERATE-REFINE | Iterate | Categorize and loop |

## Key Principles

1. **Top-Down Layer Building**: Spec → Contract → Code → Test
2. **Each Layer Must Align**: Every layer checked against spec, domain, and adjacent layers
3. **No Contract Changes During Implementation**: Step 06 forbidden from modifying require/ensure/invariant
4. **Failures Route to Correct Layer**: Step 10 identifies WHICH LAYER has the defect
5. **Tests Verify All Layers**: Tests check spec behavior, not just code behavior

## Anti-Slop Guarantees (By Layer)

| Layer | Slop Type | Prevented By |
|-------|-----------|--------------|
| Specification | Missing/vague requirements | 01 (explicit spec extraction) |
| Contract | Missing error handling | 03 (preconditions) |
| Contract | Wrong behavior specification | 04 (postconditions) |
| Contract | Invalid state | 05 (invariants) |
| Code | Type errors | 07 (compilation) |
| Code | Logic errors | 08 (contract runtime) |
| Test | Missing coverage | 09 (systematic test generation) |
| Test | Wrong expectations | 09 (tests derived from spec) |

## Layer Alignment Matrix

| Check | Description |
|-------|-------------|
| Spec ↔ Domain | Does spec capture domain requirements? |
| Contract ↔ Spec | Do contracts match spec? |
| Contract ↔ Domain | Do contracts enforce domain rules? |
| Code ↔ Contract | Does code satisfy contracts? |
| Code ↔ Spec | Does code behavior match spec? |
| Test ↔ Spec | Do tests verify spec'd behavior? |
| Test ↔ Contract | Do tests exercise contract boundaries? |

## Usage

Execute prompts in order. Each prompt specifies:
- **Input Required**: What to feed the AI
- **Output Format**: Expected structure
- **Rules**: Constraints on AI behavior
- **Success Criteria**: When to proceed
- **Next Step**: Which prompt follows
