# S02: Extract Domain Model

## Context
Phase: **DISCOVERY** (2 of 3)
Purpose: Derive problem domain concepts from class structure

## Input Required
- Project inventory from S01
- All class source files
- Note clauses and header comments

## Prompt

```
TASK: Extract domain model from existing class structure.

INPUT:
{project_inventory_from_S01}
{class_source_files}

═══════════════════════════════════════════════════════════════
DOMAIN CONCEPT EXTRACTION
What problem domain does this library address?
═══════════════════════════════════════════════════════════════

For each non-test class, analyze:
1. Class name → domain concept
2. Note clause → stated purpose
3. Public features → responsibilities
4. Relationships → domain associations

NAMING CONVENTION ANALYSIS:
- SIMPLE_{X} → Facade for domain X
- {X}_ENGINE → Core algorithm for X
- {X}_RESULT → Output data structure
- {X}_LINE / {X}_ITEM → Element in collection
- {X}_BUILDER → Construction pattern
- {X}_RENDERER → Output formatting

═══════════════════════════════════════════════════════════════
RELATIONSHIP MAPPING
How do domain concepts relate?
═══════════════════════════════════════════════════════════════

Extract relationships from:
- Attribute types: A has-a B
- Feature parameters: A uses B
- Return types: A produces B
- Inheritance: A is-a B
- Creation: A creates B

RELATIONSHIP DIAGRAM:
```
[FACADE] ──creates──> [ENGINE]
    │                    │
    │                    ├──produces──> [RESULT]
    │                    │                 │
    │                    │                 └──contains──> [ITEM]
    │                    │
    └──delegates-to──> [HELPER]
```

═══════════════════════════════════════════════════════════════
DOMAIN VOCABULARY
What terms does this library define?
═══════════════════════════════════════════════════════════════

Extract vocabulary from:
- Class names (nouns = entities)
- Feature names (verbs = operations)
- Parameter names (context terms)
- Comment terminology

VOCABULARY TABLE:
| Term | Source | Definition (inferred) |
|------|--------|----------------------|
| {term} | {class/feature} | {what it means} |

═══════════════════════════════════════════════════════════════
RESPONSIBILITY ALLOCATION
What does each domain concept DO?
═══════════════════════════════════════════════════════════════

For each class:
CLASS: {name}
  Domain Concept: {what it represents}
  Primary Responsibility: {one sentence}
  Collaborators: {list of other classes it works with}
  State: {what attributes it maintains}
  Behavior: {what operations it performs}

═══════════════════════════════════════════════════════════════
DOMAIN INVARIANTS
What must always be true in this domain?
═══════════════════════════════════════════════════════════════

Extract from class invariants:
- Business rules
- Consistency requirements
- Valid state conditions

INVARIANT: {class}
  Rule: {what the invariant enforces}
  Domain meaning: {why this matters}

OUTPUT FORMAT:

DOMAIN MODEL: {library_name}

PROBLEM DOMAIN:
  {2-3 sentence description of what problem this library solves}

CORE CONCEPTS:
  - {Concept1}: {definition}
  - {Concept2}: {definition}
  ...

RELATIONSHIPS:
  - {Concept1} --{relationship}--> {Concept2}
  ...

VOCABULARY:
  {term}: {definition}
  ...

RESPONSIBILITIES:
  {Class}: {primary responsibility}
  ...

DOMAIN RULES:
  - {rule1}
  - {rule2}
  ...

OPEN QUESTIONS:
  - {Any ambiguities that need human clarification}
```

## Success Criteria
- Problem domain clearly articulated
- All core concepts identified and defined
- Relationships between concepts mapped
- Domain vocabulary extracted
- Responsibilities clearly allocated
- Domain invariants identified

## Next Step
→ S03-EXTRACT-CLASS-SPECS.md
