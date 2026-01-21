# R02: Domain Analysis

## Context
Phase: **ANALYSIS** (2 of 3)
Purpose: Identify domain concepts and relationships

## Mindset

**The domain model is the foundation of good design.**

Object-oriented design starts with understanding the domain:
- What entities exist?
- What are their properties?
- How do they relate?
- What are the domain rules?

Get this right and the class design falls out naturally.

## Input Required
- Parsed research from R01
- Domain knowledge

## Prompt

```
TASK: Analyze the domain to identify concepts, relationships, and rules.

INPUT:
{parsed_research_from_R01}

═══════════════════════════════════════════════════════════════
NOUN EXTRACTION
What things exist in this domain?
═══════════════════════════════════════════════════════════════

From requirements and use cases, extract nouns:

NOUNS FOUND:
  From FR-001: {noun1}, {noun2}
  From FR-002: {noun3}, {noun4}
  From UC-001: {noun5}, {noun6}
  ...

NOUN CATEGORIES:
  Physical things: {list}
  Concepts: {list}
  Events: {list}
  Roles: {list}
  Documents/Records: {list}

═══════════════════════════════════════════════════════════════
DOMAIN CONCEPT IDENTIFICATION
Which nouns become classes?
═══════════════════════════════════════════════════════════════

For each noun, decide:

NOUN: {noun}
  Is it:
    [ ] A class (has state and behavior)
    [ ] An attribute (property of another class)
    [ ] External (outside our system)
    [ ] Synonym (same as another noun)

DOMAIN CONCEPTS (Classes):
  CONCEPT: {name}
    Definition: {what it represents}
    Has state: [YES | NO]
    Has behavior: [YES | NO]
    From requirements: {FR-X, FR-Y}

ATTRIBUTES (Not classes):
  {noun}: attribute of {CONCEPT}

EXTERNAL ENTITIES:
  {noun}: outside our system boundary

═══════════════════════════════════════════════════════════════
VERB EXTRACTION
What actions occur in this domain?
═══════════════════════════════════════════════════════════════

From requirements and use cases, extract verbs:

VERBS FOUND:
  From FR-001: {verb1}, {verb2}
  From UC-001: {verb3}, {verb4}
  ...

VERB → OPERATION MAPPING:
  {verb}: becomes {operation} on {CONCEPT}

═══════════════════════════════════════════════════════════════
RELATIONSHIP ANALYSIS
How do concepts relate?
═══════════════════════════════════════════════════════════════

For each pair of related concepts:

RELATIONSHIP: {CONCEPT_A} → {CONCEPT_B}
  Type: [ASSOCIATION | AGGREGATION | COMPOSITION | INHERITANCE]
  Cardinality: [1:1 | 1:N | N:M]
  Direction: [UNIDIRECTIONAL | BIDIRECTIONAL]
  Name: {relationship name, e.g., "contains", "manages"}

RELATIONSHIP DIAGRAM:
```
[CONCEPT_A] ──────1──┬──N──────> [CONCEPT_B]
                     │
                     └──1──────> [CONCEPT_C]

[CONCEPT_D] ────is-a────> [CONCEPT_E] (inheritance)
```

═══════════════════════════════════════════════════════════════
IS-A VS HAS-A ANALYSIS
What are the true inheritance relationships?
═══════════════════════════════════════════════════════════════

For each potential inheritance:

POTENTIAL IS-A: {CHILD} is-a {PARENT}

LISKOV TEST:
  "A {CHILD} is always a valid {PARENT}": [TRUE | FALSE]

SEMANTIC TEST:
  In the domain, is {CHILD} truly a kind of {PARENT}?: [YES | NO]

VERDICT: [IS-A | HAS-A]
  If HAS-A: {CHILD} has-a {component}

═══════════════════════════════════════════════════════════════
DOMAIN RULES
What rules govern this domain?
═══════════════════════════════════════════════════════════════

Extract rules from requirements:

DOMAIN RULE: DR-{id}
  Source: {requirement or constraint}
  Statement: {the rule}
  Enforced by: {which concept}
  Contract type: [INVARIANT | PRECONDITION | POSTCONDITION]

RULE CATEGORIES:
  Validity rules: {what makes data valid}
  Business rules: {what business requires}
  Consistency rules: {what must stay consistent}
  Temporal rules: {ordering requirements}

═══════════════════════════════════════════════════════════════
DOMAIN VOCABULARY
Define precise terminology
═══════════════════════════════════════════════════════════════

GLOSSARY:
  TERM: {term}
    Definition: {precise definition}
    Synonyms: {other terms used}
    Not to be confused with: {similar but different}

NAMING CONVENTIONS:
  {CONCEPT} will be named: {CLASS_NAME}
  {relationship} will be named: {feature_name}

═══════════════════════════════════════════════════════════════
BOUNDARY IDENTIFICATION
What's inside vs outside the system?
═══════════════════════════════════════════════════════════════

SYSTEM BOUNDARY:
```
┌─────────────────────────────────────────────────────────────┐
│                      OUR SYSTEM                             │
│                                                             │
│   [CONCEPT_A]  [CONCEPT_B]  [CONCEPT_C]                    │
│                                                             │
└───────────────────────┬─────────────────────────────────────┘
                        │ interfaces with
                        ▼
              [EXTERNAL_SYSTEM_1]  [EXTERNAL_SYSTEM_2]
```

INTERFACES:
  With {EXTERNAL}: {what interactions}

═══════════════════════════════════════════════════════════════
STATE ANALYSIS
What states do concepts have?
═══════════════════════════════════════════════════════════════

For concepts with lifecycle:

STATE MODEL: {CONCEPT}
  States: {state1}, {state2}, {state3}
  Transitions:
    {state1} → {state2}: when {event}
    {state2} → {state3}: when {event}

STATE DIAGRAM:
```
[INITIAL] ──create──> [STATE_A] ──process──> [STATE_B]
                          │
                          └──cancel──> [CANCELLED]
```

═══════════════════════════════════════════════════════════════
DOMAIN MODEL SUMMARY
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

# DOMAIN MODEL: {project_name}

## Domain Overview
{Description of the domain}

## Core Concepts

### {CONCEPT_1}
- Definition: {what it is}
- Attributes: {list}
- Operations: {list}
- Rules: {DR-X, DR-Y}

### {CONCEPT_2}
...

## Relationships

| From | To | Type | Cardinality | Name |
|------|----|----- |-------------|------|
| A | B | composition | 1:N | contains |

## Relationship Diagram
{ASCII diagram}

## Domain Rules
| ID | Rule | Enforced By | Type |
|----|------|-------------|------|
| DR-001 | {rule} | {class} | invariant |

## State Models
{For concepts with states}

## Glossary
| Term | Definition |
|------|------------|
| {term} | {definition} |

## System Boundaries
{What's in, what's out}
```

## Success Criteria
- All domain concepts identified
- Relationships mapped
- Is-a vs has-a decided
- Domain rules extracted
- Vocabulary defined

## Next Step
→ R03-CHALLENGE-ASSUMPTIONS.md
