# Design Audit Workflow: Detecting and Fixing Poor Eiffel Design

## CRITICAL: ANTI-SLOP ENFORCEMENT

**Before using this workflow, read: `../ANTI-SLOP-RULES.md`**

This workflow requires ACTUAL:
- File reading (Read tool with actual file paths)
- Analysis based on ACTUAL code structure
- Refactoring with compilation verification

**FORBIDDEN**:
- Describing design smells without reading actual code
- Suggesting refactorings without implementing them
- Claiming "improved design" without compile/test verification

---

## Overview

8 sequential prompts for identifying poor object-oriented design in Eiffel code and refactoring to follow OOSC2 principles. This workflow exposes design smells, then systematically fixes them.

## The Problem

Even working code can have poor design:
- Inheritance misuse (is-a vs has-a confusion)
- Missed reuse opportunities
- Missing genericity
- Bloated classes (God objects)
- Feature envy
- Procedural code in OO clothing

**Well-designed code is easier to maintain, extend, and reuse.**

## Design Principles (OOSC2)

```
┌─────────────────────────────────────────────────────────────┐
│              THE FIVE OOSC2 DESIGN PILLARS                  │
├─────────────────────────────────────────────────────────────┤
│  1. ABSTRACTION: Hide implementation, expose interface      │
│  2. INFORMATION HIDING: Internals are private              │
│  3. MODULARITY: Single responsibility per class            │
│  4. REUSABILITY: Write once, use many times                │
│  5. EXTENSIBILITY: Open for extension, closed for change   │
└─────────────────────────────────────────────────────────────┘
```

## Design Smells Detected

| Smell | Description | Impact |
|-------|-------------|--------|
| **God Class** | Class does too much | Hard to maintain, test, reuse |
| **Feature Envy** | Uses another class's data more than own | Wrong responsibility allocation |
| **Inheritance Abuse** | Is-a used when has-a appropriate | Fragile base class, coupling |
| **Missing Genericity** | Duplicated code for different types | Missed reuse, maintenance burden |
| **Primitive Obsession** | Using primitives instead of domain types | Lost semantics, validation gaps |
| **Long Parameter List** | Features with many parameters | Complexity, hard to call |
| **Data Clumps** | Same data always travels together | Missing abstraction |
| **Dead Code** | Unreachable or unused code | Confusion, maintenance burden |
| **Inappropriate Intimacy** | Classes too coupled | Change ripple effects |
| **Refused Bequest** | Child doesn't use parent's features | Wrong inheritance |

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    DETECTION PHASE                              │
│              (Find design problems)                             │
├─────────────────────────────────────────────────────────────────┤
│  D01-STRUCTURE-ANALYSIS      Map class relationships           │
│           ↓                                                     │
│  D02-SMELL-DETECTION         Identify design smells            │
│           ↓                                                     │
│  D03-INHERITANCE-AUDIT       Check is-a vs has-a               │
│           ↓                                                     │
│  D04-GENERICITY-SCAN         Find missing generics             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    REFACTORING PHASE                            │
│              (Fix design problems)                              │
├─────────────────────────────────────────────────────────────────┤
│  D05-REFACTOR-PLAN           Design the improvements           │
│           ↓                                                     │
│  D06-EXTRACT-ABSTRACTIONS    Create missing classes            │
│           ↓                                                     │
│  D07-APPLY-GENERICITY        Add type parameters               │
│           ↓                                                     │
│  D08-VERIFY-DESIGN           Validate improvements             │
└─────────────────────────────────────────────────────────────────┘
```

## Prompt Files

| # | File | Phase | Purpose |
|---|------|-------|---------|
| D01 | STRUCTURE-ANALYSIS | Detection | Map inheritance, dependencies |
| D02 | SMELL-DETECTION | Detection | Find design smells |
| D03 | INHERITANCE-AUDIT | Detection | Validate inheritance decisions |
| D04 | GENERICITY-SCAN | Detection | Find genericity opportunities |
| D05 | REFACTOR-PLAN | Refactoring | Design the improvements |
| D06 | EXTRACT-ABSTRACTIONS | Refactoring | Create new classes |
| D07 | APPLY-GENERICITY | Refactoring | Add type parameters |
| D08 | VERIFY-DESIGN | Refactoring | Validate and document |

## Key Refactoring Patterns

### Inheritance to Composition
```
BEFORE:
  class CAR inherit VEHICLE, RADIO ...

AFTER:
  class CAR inherit VEHICLE
    radio: RADIO  -- has-a instead of is-a
```

### Extract Class
```
BEFORE:
  class ORDER
    customer_name, customer_address, items, total ...

AFTER:
  class ORDER
    customer: CUSTOMER
    items: LIST [ORDER_ITEM]
```

### Introduce Genericity
```
BEFORE:
  class STRING_LIST ... end
  class INTEGER_LIST ... end

AFTER:
  class MY_LIST [G] ... end
```

### Replace Primitives with Domain Types
```
BEFORE:
  email: STRING
  phone: STRING

AFTER:
  email: EMAIL_ADDRESS  -- with validation
  phone: PHONE_NUMBER   -- with formatting
```

## Eiffel-Specific Design Guidelines

### When to Use Inheritance
- IS-A relationship (Liskov Substitution)
- Subtype polymorphism needed
- Reusing implementation AND interface

### When to Use Composition
- HAS-A relationship
- Implementation reuse without interface
- Dynamic behavior changes
- Multiple "parents" of same kind

### When to Use Genericity
- Same algorithm, different types
- Type-safe collections
- Avoiding code duplication

### When to Use Deferred Classes
- Define interface without implementation
- Force subclasses to implement
- Template Method pattern

## Output Format

The workflow produces:

```
design-audit/
├── STRUCTURE-MAP.md          # Class relationships
├── SMELL-REPORT.md           # Identified design problems
├── INHERITANCE-AUDIT.md      # Is-a vs has-a analysis
├── GENERICITY-REPORT.md      # Genericity opportunities
├── REFACTOR-PLAN.md          # Improvement roadmap
├── NEW-CLASSES/              # Extracted abstractions
│   └── {CLASS_NAME}.e
├── CHANGES-LOG.md            # What was changed
└── DESIGN-REVIEW.md          # Final assessment
```

## When to Use This Workflow

1. **Before major feature additions** - Clean design first
2. **When code is hard to change** - Design problems are likely
3. **When tests are hard to write** - Often design issues
4. **During onboarding** - Understand design decisions
5. **Periodic audits** - Prevent design rot

## Metrics Tracked

- Classes per responsibility (1 is ideal)
- Average inheritance depth (< 4 is ideal)
- Features per class (< 20 is ideal)
- Parameters per feature (< 4 is ideal)
- Generic classes ratio (higher is better)
- Coupling between classes (lower is better)
