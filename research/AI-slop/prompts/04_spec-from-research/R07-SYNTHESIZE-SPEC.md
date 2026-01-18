# R07: Synthesize Specification

## Context
Phase: **SYNTHESIS** (1 of 2)
Purpose: Combine all design work into formal specification documents

## Mindset

**The specification is the contract between design and implementation.**

Assemble all the design work into a complete, formal specification that:
- Defines exactly what to build
- Specifies all contracts
- Documents all decisions
- Guides implementation

## Input Required
- All previous design documents (R01-R06)

## Prompt

```
TASK: Synthesize a complete formal specification from all design work.

INPUT:
{parsed_research_R01}
{domain_model_R02}
{challenge_results_R03}
{class_design_R04}
{contract_design_R05}
{interface_design_R06}

═══════════════════════════════════════════════════════════════
SPECIFICATION DOCUMENT STRUCTURE
═══════════════════════════════════════════════════════════════

Create the following specification documents:

1. OVERVIEW.md - Executive summary
2. DOMAIN-MODEL.md - Domain concepts and rules
3. CLASS-SPECS/ - Per-class specifications
4. CONTRACTS.md - All contracts
5. INTERFACES.md - Public API
6. CONSTRAINTS.md - System invariants
7. DESIGN-RATIONALE.md - Decision reasoning

═══════════════════════════════════════════════════════════════
OVERVIEW DOCUMENT
Executive summary of the specification
═══════════════════════════════════════════════════════════════

# {PROJECT_NAME} Specification

## Purpose
{From research: what problem this solves}

## Scope
{What's included and excluded}

## Key Concepts
{Brief description of main domain concepts}

## Class Summary
| Class | Responsibility |
|-------|----------------|
| SIMPLE_{X} | {responsibility} |

## Quick Start
```eiffel
{Minimal usage example}
```

## Phases
{If phased delivery, what's in each phase}

═══════════════════════════════════════════════════════════════
DOMAIN MODEL DOCUMENT
Complete domain specification
═══════════════════════════════════════════════════════════════

# Domain Model

## Problem Domain
{Description of the domain}

## Core Concepts

### {CONCEPT_1}
- Definition: {precise definition}
- Properties: {list}
- Rules: {domain rules}

### {CONCEPT_2}
...

## Relationships
{Diagram and descriptions}

## Domain Rules
| ID | Rule | Enforced By |
|----|------|-------------|
| DR-001 | {rule} | {class.invariant} |

## Glossary
| Term | Definition |
|------|------------|
| {term} | {definition} |

═══════════════════════════════════════════════════════════════
CLASS SPECIFICATION DOCUMENTS
One document per class
═══════════════════════════════════════════════════════════════

Create specs/CLASS-SPECS/{CLASS_NAME}.md:

# {CLASS_NAME} Specification

## Identity
- **Name**: {CLASS_NAME}
- **Role**: {FACADE | ENGINE | DATA | HELPER}
- **Responsibility**: {single responsibility}

## Domain Concept
{What domain concept this represents}

## Inheritance
- Inherits: {parent classes}
- Inherited by: {child classes}

## Genericity
```eiffel
class {CLASS_NAME} [G -> {CONSTRAINT}]
```

## Creation

### make
```eiffel
make
    -- Default creation.
  ensure
    {postconditions}
  end
```

### make_from_{source}
```eiffel
make_from_{source} (a_source: {TYPE})
    -- Create from {source}.
  require
    {preconditions}
  ensure
    {postconditions}
  end
```

## Queries

### {query_name}
```eiffel
{query_name}: {RETURN_TYPE}
    -- {Description}.
  require
    {preconditions}
  ensure
    {postconditions}
  end
```

## Commands

### {command_name}
```eiffel
{command_name} (a_param: {TYPE})
    -- {Description}.
  require
    {preconditions}
  ensure
    {postconditions}
  end
```

## Invariant
```eiffel
invariant
  {invariant_clauses}
```

## Implementation Notes
{Any guidance for implementers}

═══════════════════════════════════════════════════════════════
CONTRACTS DOCUMENT
Consolidated contract reference
═══════════════════════════════════════════════════════════════

# Contracts

## Contract Philosophy
{How contracts are used in this library}

## Invariants by Class

### {CLASS_NAME}
```eiffel
invariant
  {all invariants}
```

## Preconditions by Feature

### {CLASS_NAME}.{feature}
```eiffel
require
  {all preconditions}
```

## Postconditions by Feature

### {CLASS_NAME}.{feature}
```eiffel
ensure
  {all postconditions}
```

## Semantic Postconditions
{Postconditions that verify meaning, not just syntax}

## Contract Traceability
| Requirement | Contract |
|-------------|----------|
| FR-001 | {class}.{feature} ensure {x} |

═══════════════════════════════════════════════════════════════
INTERFACES DOCUMENT
Public API reference
═══════════════════════════════════════════════════════════════

# Public Interfaces

## Facade: SIMPLE_{X}

### Configuration
```eiffel
set_{option} (a_value: TYPE): like Current
```

### Operations
```eiffel
{operation} (a_input: INPUT): RESULT
```

## Result Classes

### {RESULT_NAME}
```eiffel
{queries and commands}
```

## Error Handling
{Error pattern and codes}

## Usage Examples
{Code examples for common scenarios}

═══════════════════════════════════════════════════════════════
CONSTRAINTS DOCUMENT
System-wide rules
═══════════════════════════════════════════════════════════════

# System Constraints

## Validity Rules
{What makes data valid}

## Consistency Rules
{What must stay consistent}

## Temporal Rules
{What must happen in order}

## Resource Rules
{Limits and bounds}

## Non-Functional Constraints
{Performance, security, etc.}

═══════════════════════════════════════════════════════════════
DESIGN RATIONALE DOCUMENT
Why decisions were made
═══════════════════════════════════════════════════════════════

# Design Rationale

## Key Decisions

### Decision: {decision}
- **Alternatives**: {what else was considered}
- **Rationale**: {why this was chosen}
- **Trade-offs**: {what was given up}
- **Implications**: {what this means for implementation}

## Pattern Choices

### {Pattern}
- **Why used**: {rationale}
- **Where used**: {classes}

## OOSC2 Application

### How Single Responsibility was applied
{Explanation}

### How Open/Closed was applied
{Explanation}

### How genericity was used
{Explanation}

═══════════════════════════════════════════════════════════════
SPECIFICATION ASSEMBLY
Create the complete spec folder
═══════════════════════════════════════════════════════════════

OUTPUT STRUCTURE:

```
specs/
├── OVERVIEW.md
├── DOMAIN-MODEL.md
├── CLASS-SPECS/
│   ├── SIMPLE_{X}.md
│   ├── {ENGINE}.md
│   ├── {RESULT}.md
│   └── ...
├── CONTRACTS.md
├── INTERFACES.md
├── CONSTRAINTS.md
└── DESIGN-RATIONALE.md
```

OUTPUT FORMAT:

# SPECIFICATION SYNTHESIS COMPLETE

## Documents Created

| Document | Purpose | Size |
|----------|---------|------|
| OVERVIEW.md | Executive summary | {lines} |
| DOMAIN-MODEL.md | Domain specification | {lines} |
| CLASS-SPECS/*.md | Per-class specs | {count} files |
| CONTRACTS.md | Contract reference | {lines} |
| INTERFACES.md | API reference | {lines} |
| CONSTRAINTS.md | System rules | {lines} |
| DESIGN-RATIONALE.md | Decision record | {lines} |

## Specification Statistics

- Classes specified: {count}
- Features specified: {count}
- Contracts defined: {count}
- Requirements traced: {count}/{total}

## Ready for Implementation

The specification is complete and ready for implementation review.
```

## Success Criteria
- All documents created
- All classes specified
- All contracts documented
- All requirements traceable
- Design decisions recorded

## Next Step
→ R08-VALIDATE-DESIGN.md
