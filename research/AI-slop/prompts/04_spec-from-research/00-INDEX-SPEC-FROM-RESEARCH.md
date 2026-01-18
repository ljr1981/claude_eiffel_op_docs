# Spec-from-Research Workflow: Designing Specifications with OOSC2 Excellence

## CRITICAL: ANTI-SLOP ENFORCEMENT

**Before using this workflow, read: `../ANTI-SLOP-RULES.md`**

This workflow requires ACTUAL:
- Reading the research document (Read tool)
- Generating specs in actual files (Write/Edit tools)
- Verification that specs compile (if generating .e stubs)

**FORBIDDEN**:
- Summarizing research without reading the actual document
- Generating specs without writing them to files
- Claiming "spec complete" without actual deliverables

---

## Overview

8 sequential prompts for transforming a 7-step research product into a well-designed Eiffel specification. This workflow applies OOSC2 principles, Design by Contract, and good Eiffel design patterns to create specifications that lead to excellent code.

## The Problem

Research produces WHAT to build, not HOW to build it well:
- Requirements are domain-focused, not implementation-focused
- No guidance on class structure
- No consideration of reuse and extensibility
- No contract definitions
- No consideration of Eiffel idioms

**Solution:** Transform research output through a rigorous design process that applies object-oriented principles and Eiffel best practices.

## The Design Process

```
┌─────────────────────────────────────────────────────────────┐
│              GOOD EIFFEL DESIGN PRINCIPLES                  │
├─────────────────────────────────────────────────────────────┤
│  1. SINGLE CHOICE: Decisions made in one place             │
│  2. OPEN/CLOSED: Open for extension, closed for change     │
│  3. COMMAND/QUERY: Queries return, commands modify         │
│  4. UNIFORM ACCESS: Attribute and function interchangeable │
│  5. DESIGN BY CONTRACT: Pre, post, invariant everywhere    │
│  6. INFORMATION HIDING: Implementation is private          │
│  7. GENERICITY: Parameterize over types                    │
│  8. REUSE: Inherit interface AND implementation            │
└─────────────────────────────────────────────────────────────┘
```

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    ANALYSIS PHASE                               │
│              (Attack and dismantle the research)                │
├─────────────────────────────────────────────────────────────────┤
│  R01-PARSE-RESEARCH          Extract requirements              │
│           ↓                                                     │
│  R02-DOMAIN-ANALYSIS         Identify domain concepts          │
│           ↓                                                     │
│  R03-CHALLENGE-ASSUMPTIONS   Question every assumption         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    DESIGN PHASE                                 │
│              (Apply OOSC2 principles)                           │
├─────────────────────────────────────────────────────────────────┤
│  R04-CLASS-DESIGN            Design class structure            │
│           ↓                                                     │
│  R05-CONTRACT-DESIGN         Define all contracts              │
│           ↓                                                     │
│  R06-INTERFACE-DESIGN        Design public APIs                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    SYNTHESIS PHASE                              │
│              (Assemble final specification)                     │
├─────────────────────────────────────────────────────────────────┤
│  R07-SYNTHESIZE-SPEC         Create formal specification       │
│           ↓                                                     │
│  R08-VALIDATE-DESIGN         Verify design quality             │
└─────────────────────────────────────────────────────────────────┘
```

## Prompt Files

| # | File | Phase | Purpose |
|---|------|-------|---------|
| R01 | PARSE-RESEARCH | Analysis | Extract requirements from research |
| R02 | DOMAIN-ANALYSIS | Analysis | Identify domain concepts |
| R03 | CHALLENGE-ASSUMPTIONS | Analysis | Question everything |
| R04 | CLASS-DESIGN | Design | Design class structure |
| R05 | CONTRACT-DESIGN | Design | Define all contracts |
| R06 | INTERFACE-DESIGN | Design | Design public APIs |
| R07 | SYNTHESIZE-SPEC | Synthesis | Create formal specification |
| R08 | VALIDATE-DESIGN | Synthesis | Verify design quality |

## Input: 7-Step Research Product

This workflow expects input from the 7-step research process:

```
research-output/
├── STEP-1-SCOPE.md           # Problem scope and boundaries
├── STEP-2-LANDSCAPE.md       # Existing solutions analysis
├── STEP-3-REQUIREMENTS.md    # Functional and non-functional requirements
├── STEP-4-DECISIONS.md       # Key design decisions made
├── STEP-5-INNOVATIONS.md     # Novel approaches identified
├── STEP-6-RISKS.md           # Risks and mitigations
├── STEP-7-RECOMMENDATION.md  # Final recommendation
└── APPENDIX/                 # Supporting materials
```

## Design Quality Criteria

The specification must satisfy:

### OOSC2 Compliance
- [ ] Single Responsibility per class
- [ ] Open/Closed principle followed
- [ ] Liskov Substitution preserved
- [ ] Interface Segregation applied
- [ ] Dependency Inversion where appropriate

### Eiffel Excellence
- [ ] Command-Query Separation
- [ ] Uniform Access Principle
- [ ] Design by Contract (full coverage)
- [ ] Appropriate genericity
- [ ] Inheritance used correctly
- [ ] Information hiding enforced

### Practical Quality
- [ ] Void-safe design
- [ ] SCOOP-compatible
- [ ] Testable architecture
- [ ] Clear documentation

## Output Format

The workflow produces:

```
spec/
├── DOMAIN-MODEL.md           # Domain concepts and relationships
├── CLASS-HIERARCHY.md        # Inheritance structure
├── CLASS-SPECS/              # Per-class specifications
│   ├── {FACADE}.md
│   ├── {ENGINE}.md
│   └── ...
├── CONTRACTS.md              # All contracts documented
├── INTERFACES.md             # Public API specifications
├── CONSTRAINTS.md            # System-wide invariants
└── DESIGN-RATIONALE.md       # Why decisions were made
```

## When to Use This Workflow

1. **After completing 7-step research** - Transform findings into design
2. **Starting a new library** - Design before coding
3. **Major feature additions** - Design the extension properly
4. **Refactoring to better design** - Create target specification

## Key Principles Applied

### Attack Mindset
Don't accept research at face value. Challenge:
- Are these the right requirements?
- Are there missing requirements?
- Are assumptions valid?
- What could go wrong?

### Design Excellence
Apply rigorous design:
- Every class has one responsibility
- Every feature has contracts
- Every decision has rationale
- Every pattern has justification

### Eiffel Idioms
Use Eiffel patterns:
- Facade for library entry
- Engine for algorithms
- Result objects for complex returns
- Builder pattern for configuration
- Once functions for singletons
