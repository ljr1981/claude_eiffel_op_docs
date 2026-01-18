# Spec Extraction Workflow: Deriving Specifications from Existing Code

## CRITICAL: ANTI-SLOP ENFORCEMENT

**Before using this workflow, read: `../ANTI-SLOP-RULES.md`**

This workflow requires ACTUAL:
- File reading (Read tool for ECF, .e files, tests)
- Extraction based on ACTUAL code content
- Specs referencing ACTUAL file paths and line numbers

**FORBIDDEN**:
- Imagining what a class "might do"
- Describing contracts without reading actual require/ensure
- Generating specs without referencing actual source files

---

## Overview

8 sequential prompts for systematically extracting formal specifications from an existing Eiffel codebase that lacks explicit specification documents. This workflow reverse-engineers the intended behavior from ECF configuration, class structure, feature signatures, contracts, and tests.

## The Problem

Many existing codebases lack explicit specification documents:
- No `specs/` folder
- Requirements exist only in developers' heads
- Intent is implicit in code structure
- Contracts capture SOME intent but not WHY

**Solution:** Derive specifications deterministically from what EXISTS, assuming the existing implementation is correct (or mostly correct).

## The Extraction Pyramid

```
                    ┌─────────────────────┐
                    │   DOMAIN MODEL      │  ← Extracted from class names,
                    │   (What problem?)   │     relationships, note clauses
                    └─────────────────────┘
                              ↓
              ┌───────────────────────────────┐
              │     BEHAVIORAL SPECS          │  ← Extracted from feature
              │   (What does each do?)        │     signatures, contracts
              └───────────────────────────────┘
                              ↓
        ┌─────────────────────────────────────────┐
        │         CONSTRAINT SPECS                │  ← Extracted from
        │   (What must always be true?)           │     invariants, preconditions
        └─────────────────────────────────────────┘
                              ↓
    ┌─────────────────────────────────────────────────┐
    │            BOUNDARY SPECS                       │  ← Extracted from
    │   (What are the limits and edge cases?)         │     tests, error handling
    └─────────────────────────────────────────────────┘
```

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    DISCOVERY PHASE                              │
│              (Gather all source material)                       │
├─────────────────────────────────────────────────────────────────┤
│  S01-INVENTORY-PROJECT       ECF, clusters, dependencies       │
│           ↓                                                     │
│  S02-EXTRACT-DOMAIN-MODEL    Classes → domain concepts         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    EXTRACTION PHASE                             │
│              (Derive specs from artifacts)                      │
├─────────────────────────────────────────────────────────────────┤
│  S03-EXTRACT-CLASS-SPECS     Per-class responsibility          │
│           ↓                                                     │
│  S04-EXTRACT-FEATURE-SPECS   Per-feature behavior              │
│           ↓                                                     │
│  S05-EXTRACT-CONSTRAINTS     Invariants → rules                │
│           ↓                                                     │
│  S06-EXTRACT-BOUNDARIES      Tests → edge cases                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    SYNTHESIS PHASE                              │
│              (Assemble formal specification)                    │
├─────────────────────────────────────────────────────────────────┤
│  S07-SYNTHESIZE-SPEC         Combine into formal document      │
│           ↓                                                     │
│  S08-VALIDATE-SPEC           Cross-check spec vs code          │
└─────────────────────────────────────────────────────────────────┘
```

## Prompt Files

| # | File | Phase | Purpose |
|---|------|-------|---------|
| S01 | INVENTORY-PROJECT | Discovery | Map ECF structure, dependencies |
| S02 | EXTRACT-DOMAIN-MODEL | Discovery | Derive domain concepts from classes |
| S03 | EXTRACT-CLASS-SPECS | Extraction | Per-class responsibility specs |
| S04 | EXTRACT-FEATURE-SPECS | Extraction | Per-feature behavioral specs |
| S05 | EXTRACT-CONSTRAINTS | Extraction | Invariant-derived rules |
| S06 | EXTRACT-BOUNDARIES | Extraction | Test-derived edge cases |
| S07 | SYNTHESIZE-SPEC | Synthesis | Combine into formal spec |
| S08 | VALIDATE-SPEC | Synthesis | Cross-check completeness |

## Key Principles

**Code is a form of specification.**
Every design decision is captured somewhere: names, types, contracts, tests.

**Contracts are executable specs.**
Preconditions = valid inputs. Postconditions = guaranteed outputs. Invariants = always-true rules.

**Tests encode expectations.**
Each test asserts what the developer expected. This IS specification.

**Absence is also information.**
Missing contracts = assumed always valid. Missing tests = assumed not critical.

**Assume correctness, document uncertainty.**
The code is assumed correct. Where ambiguity exists, document it for human review.

## Output Format

The workflow produces a `specs/` folder with:

```
specs/
├── DOMAIN-MODEL.md           # Problem domain, concepts, relationships
├── CLASS-SPECS/              # Per-class specifications
│   ├── CLASS_NAME.md
│   └── ...
├── CONSTRAINTS.md            # System-wide rules and invariants
├── BOUNDARIES.md             # Edge cases, limits, error conditions
└── SPEC-SUMMARY.md           # High-level specification overview
```

## When to Use This Workflow

1. **Before maintenance-xtreme** - Need specs to know what "correct" means
2. **Onboarding to legacy code** - Document undocumented systems
3. **Pre-refactoring** - Capture current behavior before changing
4. **Audit preparation** - Create formal specification from informal code
