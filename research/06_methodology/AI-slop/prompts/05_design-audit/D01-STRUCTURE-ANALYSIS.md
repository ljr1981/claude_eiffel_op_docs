# D01: Structure Analysis

## Context
Phase: **DETECTION** (1 of 4)
Purpose: Map the class structure and relationships

## Mindset

**Understand the architecture before judging it.**

Before identifying problems, you must understand:
- What classes exist
- How they relate to each other
- What the inheritance hierarchy looks like
- Where the dependencies are

## Input Required
- ECF file
- All source files
- Test files

## Prompt

```
TASK: Analyze the class structure and relationships of the codebase.

INPUT:
{ecf_file}
{source_files}

═══════════════════════════════════════════════════════════════
INHERITANCE HIERARCHY
Who inherits from whom?
═══════════════════════════════════════════════════════════════

For each class, identify:

CLASS: {name}
  Inherits from: {parent classes}
  Inherited by: {child classes}
  Depth in hierarchy: {number}
  Multiple inheritance: [YES | NO]

INHERITANCE TREE:
```
ANY
 └── {BASE_CLASS}
      ├── {CHILD_1}
      │    └── {GRANDCHILD}
      └── {CHILD_2}
```

INHERITANCE METRICS:
  Total classes: {count}
  Max depth: {number}
  Classes with multiple parents: {count}
  Root classes (direct from ANY): {count}

═══════════════════════════════════════════════════════════════
DEPENDENCY ANALYSIS
Who uses whom?
═══════════════════════════════════════════════════════════════

For each class, identify dependencies:

CLASS: {name}
  Uses (attributes): {classes referenced as attribute types}
  Uses (parameters): {classes referenced as parameter types}
  Uses (returns): {classes returned from features}
  Uses (creates): {classes instantiated}

DEPENDENCY GRAPH:
```
[CLASS_A] ──uses──> [CLASS_B]
    │                   │
    └──creates──> [CLASS_C]
```

DEPENDENCY METRICS:
  Afferent coupling (Ca): {classes that depend on this class}
  Efferent coupling (Ce): {classes this class depends on}
  Instability: Ce / (Ca + Ce)

═══════════════════════════════════════════════════════════════
CLIENT/SUPPLIER ANALYSIS
Who provides, who consumes?
═══════════════════════════════════════════════════════════════

SUPPLIERS (provide services):
  - {class}: provides {what services}

CLIENTS (consume services):
  - {class}: consumes {what services}

FACADE IDENTIFICATION:
  Classes that are main entry points: {list}
  Classes used only internally: {list}

═══════════════════════════════════════════════════════════════
CLASS SIZE ANALYSIS
How big are the classes?
═══════════════════════════════════════════════════════════════

For each class:

CLASS: {name}
  Lines of code: {count}
  Features: {count}
  Public features: {count}
  Private features: {count}
  Attributes: {count}
  Creation procedures: {count}

SIZE METRICS:
  Largest class: {name} ({lines} lines, {features} features)
  Smallest class: {name} ({lines} lines, {features} features)
  Average features per class: {number}
  Classes with > 20 features: {count} ← POTENTIAL GOD CLASSES

═══════════════════════════════════════════════════════════════
FEATURE DISTRIBUTION
Where do features live?
═══════════════════════════════════════════════════════════════

FEATURE DENSITY:
| Class | Queries | Commands | Attributes | Total |
|-------|---------|----------|------------|-------|
| {name} | {n} | {n} | {n} | {n} |

OUTLIERS:
  Classes with > 30 features: {list}
  Classes with only 1-2 features: {list}

═══════════════════════════════════════════════════════════════
GENERICITY USAGE
Where are generics used?
═══════════════════════════════════════════════════════════════

GENERIC CLASSES:
  - {CLASS} [G]: {purpose}
  - {CLASS} [G, H -> CONSTRAINT]: {purpose}

NON-GENERIC COLLECTIONS/STRUCTURES:
  - {class}: uses {what} for {purpose}

GENERICITY RATIO: {generic_classes}/{total_classes}

═══════════════════════════════════════════════════════════════
DEFERRED/EFFECTIVE ANALYSIS
What's abstract, what's concrete?
═══════════════════════════════════════════════════════════════

DEFERRED CLASSES:
  - {name}: {deferred_features}

EFFECTIVE CLASSES:
  - {name}: implements {from_which_parent}

ABSTRACTION RATIO: {deferred}/{total}

═══════════════════════════════════════════════════════════════
COHESION ANALYSIS (Initial)
Do classes have single responsibility?
═══════════════════════════════════════════════════════════════

For each class, list feature groups:

CLASS: {name}
  Feature group 1: {features} - {apparent purpose}
  Feature group 2: {features} - {apparent purpose}

POTENTIAL RESPONSIBILITY ISSUES:
  Classes with > 2 apparent responsibilities: {list}

═══════════════════════════════════════════════════════════════
MODEL-BASED CONTRACT COVERAGE
Are contracts complete specifications?
═══════════════════════════════════════════════════════════════

For container/collection classes, check model coverage:

MODEL COVERAGE AUDIT:
| Class | Has Model Query | Model Type | Postconditions Model-Based | Invariants Model-Based |
|-------|-----------------|------------|---------------------------|------------------------|
| {class} | YES/NO | MML_{TYPE} | YES/NO | YES/NO |

PRIMITIVE vs MODEL-BASED CONTRACTS:
| Primitive (Incomplete) | Model-Based (Complete) |
|------------------------|------------------------|
| count = old count + 1  | model |=| old model & x |
| has (x)                | model.has (x)          |
| is_empty               | model.is_empty         |

MODEL COVERAGE GAPS:
  {class}.{feature}: Uses primitive postcondition, could be model-based
    CURRENT: ensure count = old count + 1
    UPGRADE: ensure model |=| old model & x

CLASSES NEEDING MODELS:
  - {class}: Container pattern, no model query
    Recommended: model: MML_{TYPE} [{PARAMS}]

MODEL COVERAGE SCORE:
  Classes with model queries: {X}/{Y} ({percent}%)
  Model-based postconditions: {X}/{Y} ({percent}%)
  Model-based invariants: {X}/{Y} ({percent}%)

OUTPUT FORMAT:

# STRUCTURE ANALYSIS: {library_name}

## Summary
- Classes: {count}
- Max inheritance depth: {n}
- Average features per class: {n}
- Generic classes: {n} ({percent}%)
- Deferred classes: {n} ({percent}%)

## Inheritance Hierarchy
{Tree diagram}

## Dependency Map
{Dependency graph}

## Class Inventory
| Class | LOC | Features | Inherits | Depth | Coupling |
|-------|-----|----------|----------|-------|----------|
| {name} | {n} | {n} | {parents} | {n} | {Ce} |

## Potential Design Issues (Initial)
- Large classes (> 20 features): {list}
- Deep hierarchies (> 4 levels): {list}
- High coupling (> 5 dependencies): {list}
- No genericity where expected: {list}

## Visualization
{ASCII art showing main relationships}
```

## Success Criteria
- All classes inventoried
- Inheritance hierarchy mapped
- Dependencies identified
- Size metrics calculated
- Initial issues flagged

## Next Step
→ D02-SMELL-DETECTION.md
