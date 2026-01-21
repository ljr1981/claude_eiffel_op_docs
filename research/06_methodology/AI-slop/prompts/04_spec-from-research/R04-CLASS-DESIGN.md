# R04: Class Design

## Context
Phase: **DESIGN** (1 of 3)
Purpose: Design the class structure following OOSC2 principles

## Mindset

**Classes should model real concepts with single responsibilities.**

Apply these principles:
1. Each class has ONE responsibility
2. Classes model domain concepts
3. Inheritance for IS-A only
4. Composition for HAS-A
5. Information hiding enforced
6. Genericity where appropriate

## Input Required
- Validated requirements from R03
- Domain model from R02

## Prompt

```
TASK: Design the class structure for the specification.

INPUT:
{validated_requirements_from_R03}
{domain_model_from_R02}

═══════════════════════════════════════════════════════════════
CONCEPT → CLASS MAPPING
Transform domain concepts into classes
═══════════════════════════════════════════════════════════════

For each domain concept:

CONCEPT: {domain_concept}
  CLASS: {CLASS_NAME}
  Role: [FACADE | ENGINE | DATA | HELPER | RENDERER | BUILDER]
  Single responsibility: {one sentence}
  Justification: {why this is a class}

CLASS INVENTORY:
| Domain Concept | Class Name | Role | Responsibility |
|----------------|------------|------|----------------|
| {concept} | {CLASS} | FACADE | {responsibility} |

═══════════════════════════════════════════════════════════════
FACADE DESIGN
Design the main entry point
═══════════════════════════════════════════════════════════════

FACADE: SIMPLE_{X}
  Purpose: Single entry point for library functionality
  Responsibility: Coordinate and delegate to engine classes

  Public interface:
    Creation:
      - make: default creation
      - make_with_{options}: creation with configuration

    Configuration (builder pattern):
      - set_{option}: configure behavior, returns like Current

    Core operations:
      - {operation1}: {what it does}
      - {operation2}: {what it does}

  Hidden behind facade:
    - {ENGINE_CLASS}: algorithm implementation
    - {HELPER_CLASS}: utility functions

═══════════════════════════════════════════════════════════════
ENGINE DESIGN
Design the core algorithm classes
═══════════════════════════════════════════════════════════════

ENGINE: {ENGINE_NAME}
  Purpose: Implement core algorithm/logic
  Responsibility: {single responsibility}

  Key features:
    - {feature}: {purpose}

  State managed:
    - {attribute}: {purpose}

  Produces:
    - {RESULT_CLASS}: output of processing

═══════════════════════════════════════════════════════════════
DATA CLASS DESIGN
Design result and data classes
═══════════════════════════════════════════════════════════════

DATA CLASS: {DATA_NAME}
  Purpose: Hold structured data
  Immutable: [YES | NO]

  Attributes:
    - {attr1}: {TYPE} -- {purpose}
    - {attr2}: {TYPE} -- {purpose}

  Computed queries:
    - {query}: {what it computes}

═══════════════════════════════════════════════════════════════
INHERITANCE HIERARCHY DESIGN
Design proper inheritance
═══════════════════════════════════════════════════════════════

For each inheritance relationship:

INHERITANCE: {CHILD} inherit {PARENT}
  IS-A justified: {why this is true IS-A}
  Liskov test: {CHILD} can replace {PARENT} everywhere
  Features inherited: {list}
  Features redefined: {list}

HIERARCHY DIAGRAM:
```
    DEFERRED_BASE
         │
    ├────┴────┐
    │         │
 CONCRETE_A  CONCRETE_B
```

DEFERRED CLASSES:
  {DEFERRED_CLASS}
    Deferred features:
      - {feature}: {why deferred}

═══════════════════════════════════════════════════════════════
COMPOSITION DESIGN
Design has-a relationships
═══════════════════════════════════════════════════════════════

For each composition:

COMPOSITION: {CONTAINER} has {COMPONENT}
  Attribute: {component_name}: {COMPONENT_CLASS}
  Cardinality: [1 | N | 0..1 | 0..N]
  Lifecycle: [CREATED_BY_CONTAINER | EXTERNAL]
  Delegation: {features that delegate to component}

═══════════════════════════════════════════════════════════════
GENERICITY DESIGN
Design generic classes
═══════════════════════════════════════════════════════════════

For classes that should be generic:

GENERIC CLASS: {NAME} [G]
  Purpose: {why generic}
  Type parameter: G
  Constraint: [NONE | G -> {CONSTRAINT}]

  Features using G:
    - {feature} (a_param: G): G

GENERIC USAGE:
  {CLIENT_CLASS} uses {GENERIC} [{ACTUAL_TYPE}]

═══════════════════════════════════════════════════════════════
INFORMATION HIDING DESIGN
Decide what's public vs private
═══════════════════════════════════════════════════════════════

For each class:

CLASS: {NAME}

  Public features (API):
    - {feature}: {why public}

  Private features (implementation):
    - {feature}: {why private}

  Feature categories:
    feature -- Access
      {public queries}

    feature -- Operations
      {public commands}

    feature {NONE} -- Implementation
      {private features}

═══════════════════════════════════════════════════════════════
RESPONSIBILITY VERIFICATION
Verify single responsibility
═══════════════════════════════════════════════════════════════

For each class:

CLASS: {NAME}
  Stated responsibility: {one sentence}

  Features analysis:
    - {feature}: supports responsibility? [YES | NO]
    - {feature}: supports responsibility? [YES | NO]

  VERDICT: [SINGLE_RESPONSIBILITY | NEEDS_SPLIT]

  If NEEDS_SPLIT:
    Extract to: {NEW_CLASS} for {second responsibility}

═══════════════════════════════════════════════════════════════
COUPLING ANALYSIS
Verify loose coupling
═══════════════════════════════════════════════════════════════

CLASS: {NAME}
  Depends on: {list of other classes}
  Coupling count: {number}

  VERDICT: [ACCEPTABLE | TOO_COUPLED]

  If TOO_COUPLED:
    Introduce: {interface or abstraction}

═══════════════════════════════════════════════════════════════
CLASS DIAGRAM
Visual representation
═══════════════════════════════════════════════════════════════

```
┌─────────────────────────────────────────────────────────────┐
│                    SIMPLE_{X}                               │
│                    (Facade)                                 │
├─────────────────────────────────────────────────────────────┤
│ + make                                                      │
│ + set_option: like Current                                  │
│ + operation: RESULT                                         │
├─────────────────────────────────────────────────────────────┤
│ - engine: ENGINE                                            │
└────────────────────────────┬────────────────────────────────┘
                             │ delegates to
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    ENGINE                                   │
├─────────────────────────────────────────────────────────────┤
│ + compute: RESULT                                           │
├─────────────────────────────────────────────────────────────┤
│ - internal_state                                            │
└────────────────────────────┬────────────────────────────────┘
                             │ produces
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    RESULT                                   │
├─────────────────────────────────────────────────────────────┤
│ + data: DATA                                                │
│ + has_error: BOOLEAN                                        │
└─────────────────────────────────────────────────────────────┘
```

OUTPUT FORMAT:

# CLASS DESIGN: {project_name}

## Class Inventory
| Class | Role | Responsibility |
|-------|------|----------------|
| SIMPLE_{X} | Facade | Coordinate library use |
| {ENGINE} | Engine | Core algorithm |
| {RESULT} | Data | Hold results |

## Facade
{SIMPLE_X detailed design}

## Engine Classes
{Engine detailed designs}

## Data Classes
{Data class designs}

## Inheritance Hierarchy
{Diagram and descriptions}

## Generic Classes
{Generic class designs}

## Class Diagram
{ASCII diagram}

## Design Rationale
{Why key decisions were made}
```

## Success Criteria
- All classes identified with single responsibilities
- Inheritance properly used (IS-A only)
- Composition used for HAS-A
- Genericity applied where appropriate
- Information hiding enforced
- Low coupling achieved

## Next Step
→ R05-CONTRACT-DESIGN.md
