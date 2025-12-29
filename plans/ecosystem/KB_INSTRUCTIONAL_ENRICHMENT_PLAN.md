# KB Instructional Enrichment Plan

**Goal**: Populate simple_kb with high-quality instructional pairs to maximize RAG effectiveness, with potential future use for QLoRA fine-tuning of Qwen Coder 2.5.

**Date**: 2025-12-25
**Status**: Planning Phase - Designing Study-Based Approach

## Executive Summary

This plan enriches simple_kb FAQ system through a **study-based approach**:

**Key Insight**: Rather than generating content abstractly, Claude performs actual CODE STUDIES of the Eiffel and Gobo libraries, discovers real patterns, and creates instructional pairs from those discoveries.

### Phases (Revised)

1. Content Audit - Assess what we have, identify gaps (DONE)
2. Schema Enhancement - Extend FAQ table for instructional pairs (DONE)
3. **Code Study Planning** - Design specific studies to conduct
4. **Code Studies** - Actually study codebase for patterns/usage
5. **FAQ Generation** - Create pairs from study findings
6. External Content Review - Eiffel.org, ECMA-367 (supplement studies)
7. RAG Quality Testing - Measure and iterate
8. QLoRA Export - If needed

**Ultimate Test**: Can simple_kb + RAG answer Eiffel questions well enough that QLoRA fine-tuning is unnecessary?

---

## Study-Based Approach

### Why Studies Instead of Abstract Generation?

| Approach | Quality | Authenticity | Effort |
|----------|---------|--------------|--------|
| Abstract AI generation | Medium | Low | Fast |
| **Code study + discovery** | **High** | **High** | Medium |
| Manual expert authoring | High | High | Slow |

By actually STUDYING the code:
- Patterns are real, not hypothetical
- Examples come from working code
- Coverage reflects actual usage
- Quality is grounded in evidence

---

## Phase 3: Code Study Planning

### Proposed Studies

Each study produces a report + derived FAQ/instructional pairs.

#### Study 1: Multiple Inheritance Patterns
**Scope**: EiffelStudio stdlib, Gobo, simple_* libraries
**Questions to Answer**:
- How many classes use multiple inheritance?
- What are the common parent combinations?
- How is feature renaming/redefining used?
- What are the mixin patterns?
- How is diamond inheritance resolved?

**Deliverable**: MULTIPLE_INHERITANCE_STUDY.md + 30 FAQ pairs

#### Study 2: Generics (Constrained) Usage
**Scope**: Same as above
**Questions to Answer**:
- How many generic classes exist?
- What constraints are commonly used?
- How are generic parameters named?
- What are the constrained generic patterns?
- How do formal generics interact with inheritance?

**Deliverable**: GENERICS_STUDY.md + 25 FAQ pairs

#### Study 3: Across Loop Patterns
**Scope**: Same as above
**Questions to Answer**:
- How are across loops used vs traditional loops?
- What cursor patterns exist?
- How is across used with different structures?
- What are the some/all modifier uses?

**Deliverable**: ACROSS_LOOP_STUDY.md + 20 FAQ pairs

#### Study 4: SCOOP Concurrency Patterns
**Scope**: SCOOP-enabled libraries, EiffelStudio examples
**Questions to Answer**:
- How is the separate keyword used?
- What are the inline separate patterns?
- How are wait conditions structured?
- What SCOOP patterns exist?

**Deliverable**: SCOOP_PATTERNS_STUDY.md + 40 FAQ pairs

#### Study 5: Keyword Usage Analysis
**Scope**: All accessible Eiffel code
**Questions to Answer**:
- Frequency of each Eiffel keyword
- Contextual usage patterns
- Keyword combinations
- Rare keyword usage examples

**Deliverable**: KEYWORD_USAGE_STUDY.md + 50 FAQ pairs (one per keyword context)

#### Study 6: Agent (Lambda) Patterns
**Scope**: EiffelStudio, Gobo, simple_*
**Questions to Answer**:
- How are agents declared?
- What are inline agent patterns?
- How are agents used as callbacks?
- Agent type usage patterns

**Deliverable**: AGENT_PATTERNS_STUDY.md + 20 FAQ pairs

#### Study 7: Once Feature Patterns
**Scope**: All libraries
**Questions to Answer**:
- Once per object vs once per thread vs once per process
- Singleton implementations
- Lazy initialization patterns
- Once with generics

**Deliverable**: ONCE_PATTERNS_STUDY.md + 15 FAQ pairs

#### Study 8: Contract (DBC) Patterns
**Scope**: All libraries
**Questions to Answer**:
- Precondition patterns
- Postcondition patterns (especially with old)
- Invariant patterns
- Check instruction usage
- Loop variant/invariant

**Deliverable**: DBC_PATTERNS_STUDY.md + 35 FAQ pairs

### External Resource Studies

#### Study 9: Eiffel.org Documentation Review
**Scope**: www.eiffel.org documentation pages
**Tasks**:
- Identify key tutorial content
- Extract FAQ-worthy Q&A
- Note topics to emphasize

**Deliverable**: EIFFEL_ORG_REVIEW.md + 50 FAQ pairs

#### Study 10: ECMA-367 Standard Review
**Scope**: ECMA-367 3rd edition
**Tasks**:
- Key language features defined
- Normative examples
- Clarifications for common confusions

**Deliverable**: ECMA_367_REVIEW.md + 30 FAQ pairs

---

## Study Execution Order

Recommended sequence based on dependencies and value:

1. **Keyword Usage Analysis** - Provides foundation for all other studies
2. **DBC Patterns** - Core Eiffel concept
3. **Multiple Inheritance** - Unique to Eiffel
4. **Generics** - Common advanced topic
5. **Across Loops** - Modern iteration
6. **Agent Patterns** - Functional programming
7. **Once Patterns** - Eiffel-specific
8. **SCOOP Patterns** - Concurrency
9. **Eiffel.org Review** - External validation
10. **ECMA-367 Review** - Authoritative reference

---

## Estimated FAQ Totals from Studies

| Study | Expected FAQ Pairs |
|-------|-------------------|
| Multiple Inheritance | 30 |
| Generics | 25 |
| Across Loops | 20 |
| SCOOP | 40 |
| Keyword Usage | 50 |
| Agents | 20 |
| Once Features | 15 |
| DBC Patterns | 35 |
| Eiffel.org | 50 |
| ECMA-367 | 30 |
| **Total from Studies** | **315** |

### Additional FAQ Sources

| Source | Expected Pairs |
|--------|---------------|
| Error code explanations (31 x 3) | 93 |
| Pattern implementations (14 x 3) | 42 |
| Top 100 classes (x3) | 300 |
| Cross-language (6 lang x 20) | 120 |
| **Additional** | **555** |

### Grand Total Target: ~870 FAQ pairs

---

## Phase Completion Status

- [x] Phase 1: Content Audit (DONE)
- [x] Phase 2: Schema Enhancement (DONE)
- [ ] Phase 3: Code Study Planning (THIS DOCUMENT)
- [ ] Phase 4: Execute Studies
- [ ] Phase 5: Generate FAQs from Study Findings
- [ ] Phase 6: External Content Review
- [ ] Phase 7: RAG Quality Testing
- [ ] Phase 8: QLoRA Export (if needed)

---

## Next Steps

1. Review and approve this study plan
2. Execute Study 1 (Keyword Usage Analysis) as pilot
3. Iterate based on pilot results
4. Execute remaining studies in sequence

---

## Study Report Template

Each study should produce:

```markdown
# [Topic] Study Report

**Date**: YYYY-MM-DD
**Scope**: Libraries studied
**Files Analyzed**: N

## Methodology
How the study was conducted

## Findings

### Quantitative
- Count of X: N
- Distribution: ...

### Qualitative
- Pattern 1: Description + examples
- Pattern 2: Description + examples

## FAQ Pairs Generated

### FAQ 1
**Q**: Question
**A**: Answer
**Category**: newcomer/architect/etc
**Related Classes**: [...]

### FAQ 2
...

## Recommendations
What to emphasize, what to avoid
```


---

## Comprehensive Code Pattern Study Catalog

**Focus**: CODE and ARCHITECTURE patterns as demonstrated in real Eiffel/Gobo libraries.
These are PATTERN studies, not simple how-to guides - they identify architectural best practices.

### Category A: Architectural Patterns (from actual code)

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| A1 | Facade Pattern | Single-class API | SIMPLE_JSON, SIMPLE_HTTP | 15 |
| A2 | Factory Pattern | Object creation | Gobo factories, simple_* | 12 |
| A3 | Builder Pattern | Fluent construction | simple_http request builder | 12 |
| A4 | Singleton via Once | Global instances | Throughout all libs | 10 |
| A5 | Command Pattern | Encapsulated operations | Editor, undo systems | 10 |
| A6 | Observer Pattern | Agent callbacks | Event systems | 12 |
| A7 | Template Method | Deferred hook points | Test frameworks | 10 |
| A8 | Visitor Pattern | Double dispatch | Gobo AST visitors | 8 |
| A9 | Null Object Pattern | Default implementations | Error handlers | 8 |

### Category B: Inheritance Architecture Patterns

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| B1 | Mixin Classes | Behavior injection | COMPARABLE, HASHABLE | 15 |
| B2 | Abstract Interface | All-deferred classes | ITERABLE, READABLE | 12 |
| B3 | Diamond Resolution | Select usage | Complex hierarchies | 10 |
| B4 | Feature Adaptation | rename/redefine | Gobo adaptations | 12 |
| B5 | Implementation Reuse | Non-conforming inherit | Private inheritance | 10 |

### Category C: Constrained Generic Patterns

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| C1 | Container Generics | [G] patterns | All collection classes | 15 |
| C2 | Constrained Generics | [G -> X] patterns | Sorted structures | 12 |
| C3 | Multiple Constraints | [G -> {X, Y}] | Complex generics | 8 |
| C4 | Anchor Types | like Current | Throughout | 10 |

### Category D: Contract Architecture Patterns

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| D1 | Input Validation Contracts | require patterns | All public APIs | 12 |
| D2 | State Invariant Contracts | invariant patterns | Stateful classes | 12 |
| D3 | Output Guarantee Contracts | ensure patterns | All queries | 12 |
| D4 | Delta Contracts | old expression usage | Modifier methods | 8 |

### Category E: Error Handling Architecture

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| E1 | Boolean Error Flag | has_error pattern | simple_sql, simple_http | 10 |
| E2 | Error Message Cache | last_error pattern | Throughout simple_* | 8 |
| E3 | Result Wrapper | Success/failure objects | Parsing results | 10 |
| E4 | Exception Hierarchy | Custom exceptions | Gobo exceptions | 8 |

### Category F: SCOOP Concurrency Architecture

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| F1 | Separate Processor | Region design | SCOOP examples | 12 |
| F2 | Passive Wait | Wait condition patterns | Concurrent structures | 10 |
| F3 | Inline Separate | Modern SCOOP syntax | New code | 10 |

### Category G: Data Structure Architecture

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| G1 | Linear Structures | List/array patterns | ARRAYED_LIST usage | 15 |
| G2 | Associative Structures | Hash table patterns | HASH_TABLE usage | 12 |
| G3 | Cursor Architecture | External iteration | All iterables | 10 |
| G4 | Gobo DS_* Structures | Gobo alternatives | Gobo DS library | 12 |

### Category H: API Design Architecture

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| H1 | Query/Command Split | Side-effect separation | All well-designed APIs | 12 |
| H2 | Fluent Interface | Method chaining | Builders throughout | 10 |
| H3 | Status Queries | is_*, has_*, can_* | All classes | 10 |
| H4 | Conversion Methods | to_*, as_*, from_* | String, number classes | 10 |

### Category I: External Integration Architecture

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| I1 | Inline C Pattern | inline C style | simple_* externals | 12 |
| I2 | Platform Abstraction | OS-independent | simple_file, simple_process | 10 |
| I3 | Memory Management | C pointer handling | WEL, vision2 | 10 |

### Category J: Gobo Architecture Patterns

| # | Study Name | Focus | Example Sources | Est. FAQs |
|---|-----------|-------|-----------------|-----------|
| J1 | Gobo Kernel Patterns | KL_* utilities | Gobo kernel | 10 |
| J2 | Gobo Parser Architecture | ET_* AST design | Eiffel parser | 15 |
| J3 | Gobo Structure Patterns | DS_* vs ISE structures | Gobo structures | 12 |

---

## Summary: Code Pattern Studies

| Category | Studies | Est. FAQs |
|----------|---------|-----------|
| A: Architectural Patterns | 9 | 97 |
| B: Inheritance Architecture | 5 | 59 |
| C: Generic Patterns | 4 | 45 |
| D: Contract Architecture | 4 | 44 |
| E: Error Handling | 4 | 36 |
| F: SCOOP Architecture | 3 | 32 |
| G: Data Structure Architecture | 4 | 49 |
| H: API Design Architecture | 4 | 42 |
| I: External Integration | 3 | 32 |
| J: Gobo Architecture | 3 | 37 |
| **TOTAL** | **43 studies** | **~473 FAQs** |

---

## Study Prioritization for Pilot

### Recommended Pilot Studies (High Value, Clear Scope)

1. **A1: Facade Pattern** - SIMPLE_* libraries demonstrate this clearly
2. **D1: Input Validation Contracts** - Ubiquitous, easy to scan
3. **G1: Linear Structures** - Common usage, many examples
4. **B1: Mixin Classes** - Unique Eiffel strength

### Why These Four?
- A1 shows architectural pattern discovery
- D1 shows contract pattern discovery  
- G1 shows usage pattern discovery
- B1 shows inheritance pattern discovery

Each pilot tests a different study approach.

---

## Next Action

Approve these pilot studies, then execute one to validate methodology.


---

## Karine Arnout's Pattern Classification (Foundation)

**Source**: [From Patterns to Components](https://se.inf.ethz.ch/people/arnout/patterns/) - ETH Dissertation No. 15500 (2004)

Arnout analyzed all 23 GoF patterns and classified them by componentizability in Eiffel:

### Category 1: Fully Componentizable (~65%)
Patterns that became reusable library components:

| Pattern | Eiffel Component | Key Mechanism |
|---------|-----------------|---------------|
| Observer | Event Library | Agents, publish-subscribe |
| Visitor | Visitor Library | Agents, double dispatch |
| Factory Method | Factory Library | Generics, creation |
| Abstract Factory | Factory Library | Generics, creation |
| Command | Command Library | Agents |
| Composite | Composite Library | Generics, inheritance |
| Chain of Responsibility | CoR Library | Agents |
| Mediator | - | Agents |
| Flyweight | - | Once features |

### Category 2: Built Into Eiffel
Patterns unnecessary because Eiffel already provides the mechanism:

| Pattern | Eiffel Feature | Notes |
|---------|---------------|-------|
| Prototype | Clone/copy | Built into ANY |
| Iterator | Across loops, ITERABLE | First-class iteration |
| Singleton | Once features | Language-level support |

### Category 3: Wizard/Skeleton Support
Patterns requiring code generation but following predictable structure:

| Pattern | Support Method |
|---------|---------------|
| Adapter | Pattern Wizard generates skeleton |
| Decorator | Pattern Wizard generates skeleton |
| Bridge | Pattern Wizard generates skeleton |
| Template Method | Pattern Wizard generates skeleton |

### Category 4: Context-Dependent (Non-Componentizable)
Patterns too application-specific:

| Pattern | Reason |
|---------|--------|
| Facade | Application-specific interface |
| Interpreter | Domain-specific grammar |

### Key Eiffel Mechanisms for Componentization

1. **Generics** (constrained and unconstrained) - Type-safe abstraction
2. **Multiple Inheritance** - Mixin composition
3. **Agents** - First-class functions/callbacks
4. **Design by Contract** - Interface specification

---

## Study Plan Revision: Arnout-Aligned

### What This Changes

The pattern study should be structured around Arnouts findings:

1. **Study built-in patterns**: How does Eiffel obviate the need?
2. **Study componentized patterns**: Find Event Library, etc. in Gobo/ISE
3. **Study wizard patterns**: What skeleton code is needed?
4. **Study non-patterns**: When is it just application design?

---

## Source Hierarchy for Examples

**Critical**: FAQ entries must link to LIVING CODE from canonical sources.

### Tier 1: Canonical (Human-Engineered)
- **ISE EiffelStudio stdlib** - Reference implementation
- **Gobo libraries** - Community standard, extensively reviewed
- Weight: High confidence, primary examples

### Tier 2: Canonical Extended
- **EiffelLoop** - Large, mature, human-written
- **Eiffel-Web** - Production web framework
- Weight: Good confidence

### Tier 3: Suspect (AI-Generated)
- **simple_* libraries** - Claude-generated, not human-reviewed at depth
- Weight: Use only to show pattern USAGE, not as authoritative examples
- Note: Good for showing how patterns are APPLIED, not how they SHOULD be written

### Example Entry Format

Each FAQ should include:



---

## Revised Study Categories

Based on Arnout + source hierarchy:

### A: Built-In Pattern Studies (What Eiffel Provides Natively)

| Study | Arnout Category | ISE Example | Gobo Example |
|-------|----------------|-------------|--------------|
| A1: Clone/Prototype | Built-in | ANY.twin, deep_twin | KL_* cloneable |
| A2: Iteration | Built-in | ITERABLE, across | DS_* cursors |
| A3: Singleton/Once | Built-in | Once features | - |

### B: Componentized Pattern Studies (Find the Libraries)

| Study | Arnout Category | Where to Find |
|-------|----------------|---------------|
| B1: Event/Observer | Componentized | EiffelBase ACTION_SEQUENCE |
| B2: Visitor | Componentized | Gobo ET_AST_PROCESSOR |
| B3: Factory | Componentized | Various creation patterns |
| B4: Command | Componentized | Agent-based commands |
| B5: Composite | Componentized | Tree structures |

### C: Wizard Pattern Studies (Skeleton Needed)

| Study | Arnout Category | What Skeleton Looks Like |
|-------|----------------|-------------------------|
| C1: Adapter | Wizard | Rename/redefine patterns |
| C2: Decorator | Wizard | Wrapper class structure |
| C3: Template Method | Wizard | Deferred + do-all |

### D: Inheritance Architecture (Unique to Eiffel)

| Study | Focus | ISE/Gobo Examples |
|-------|-------|-------------------|
| D1: Multiple Inheritance | Mixin patterns | COMPARABLE, HASHABLE |
| D2: Non-Conforming | Implementation reuse | {NONE} patterns |
| D3: Feature Adaptation | Rename/redefine | Throughout |

### E: Contract Patterns (DbC Architecture)

| Study | Focus | ISE/Gobo Examples |
|-------|-------|-------------------|
| E1: Precondition Idioms | require patterns | All public APIs |
| E2: Postcondition Idioms | ensure, old | Stateful operations |
| E3: Invariant Idioms | Class state | Constrained types |

---

## Next Steps

1. Download/reference Arnouts thesis for detailed pattern analysis
2. Map each componentized pattern to actual ISE/Gobo library locations
3. Create FAQ entries that link to canonical code, not AI-generated code
4. Use simple_* only as supplementary usage examples


---

## Strategic Context: The Quality Loop

### Purpose Beyond RAG

The KB is not just for answering questions - it serves a larger strategic purpose:

**Phase 1: Learn from Canonical**
- Study ISE EiffelStudio stdlib (human-engineered, production-proven)
- Study Gobo libraries (community-standard, extensively reviewed)
- Study Eiffel.org documentation (authoritative)
- Extract patterns, idioms, best practices

**Phase 2: Encode in KB**
- Create FAQ/instructional pairs grounded in canonical examples
- Link each pattern to living code in canonical libraries
- Document the "Eiffel way" of solving problems

**Phase 3: Refactor simple_* Ecosystem**
- Use KB to validate simple_* code against canonical patterns
- AI-assisted refactoring using KB as quality reference
- Elevate AI-generated code to canonical quality standards

**Phase 4: Continuous Improvement**
- simple_* becomes more canonical over time
- KB grows with validated patterns from improved simple_*
- Quality loop reinforces itself

### Why This Matters

| Current State | Target State |
|--------------|--------------|
| simple_* is AI-generated, unreviewed | simple_* matches canonical quality |
| Patterns may be incorrect/suboptimal | Patterns validated against Arnout/Meyer |
| No authoritative reference | KB provides quality benchmark |
| Human review bottleneck | AI-assisted with KB validation |

### The KB as Quality Oracle

The KB becomes the authoritative source for:
1. "Is this the right pattern for this problem?"
2. "Does this code follow Eiffel idioms?"
3. "How should this be structured according to canonical sources?"

### Success Metrics

- simple_* code passes KB pattern validation
- Refactored code matches canonical structure
- AI can cite KB sources for design decisions
- Quality improves measurably over refactoring cycles

---

## Revised Pilot Study Selection

Given this strategic context, pilot studies should:

1. **Establish canonical baselines** - What does good look like?
2. **Create validation criteria** - How do we measure compliance?
3. **Enable refactoring** - What patterns need fixing in simple_*?

### Recommended Pilots

| Study | Strategic Value |
|-------|----------------|
| B1: Event/Observer (ACTION_SEQUENCE) | Validate simple_* callback patterns |
| D1: Multiple Inheritance | Validate simple_* mixin usage |
| E1: Precondition Idioms | Validate simple_* contract quality |
| A2: Iteration (across) | Validate simple_* loop patterns |

These four studies will:
- Establish canonical examples from ISE/Gobo
- Create FAQ entries with validation criteria
- Enable AI-assisted refactoring of simple_* code



---

## Source Canonicity Declaration

### Canonical Sources (Authoritative for KB)
- ISE EiffelStudio stdlib
- Gobo libraries (github.com/gobo-project)
- Eiffel.org documentation
- ECMA-367 standard
- Arnout dissertation (pattern componentization)
- Meyer OOSC2 (methodology)

### Non-Canonical Sources (Pending Refactoring)
- **simple_* ecosystem** - AI-generated, not human-reviewed
  - Status: SUSPECT until KB-validated refactoring occurs
  - Use: May show pattern APPLICATION but not authoritative IMPLEMENTATION
  - Goal: Elevate to canonical quality through KB-guided refactoring

### Implication for KB Construction

When creating FAQ/instructional pairs:
- ALWAYS cite canonical sources for "how it should be done"
- MAY cite simple_* as "one way it has been applied" (with caveat)
- NEVER cite simple_* as authoritative pattern implementation

### Example Citation Format





---

## Executive Summary (Updated)

### What This Plan Accomplishes

This plan creates a high-quality Eiffel knowledge base by:

1. **Studying canonical sources** (ISE, Gobo, Eiffel.org, ECMA-367, Arnout)
2. **Extracting patterns and best practices** from human-engineered code
3. **Creating FAQ/instructional pairs** linked to living code examples
4. **Establishing quality benchmarks** for Eiffel code
5. **Enabling AI-assisted refactoring** of the simple_* ecosystem

### The Quality Loop



### Source Hierarchy

| Tier | Sources | Status | Use in KB |
|------|---------|--------|-----------|
| 1 | ISE EiffelStudio stdlib | Canonical | Primary examples |
| 1 | Gobo libraries | Canonical | Primary examples |
| 1 | Eiffel.org documentation | Canonical | Reference material |
| 1 | ECMA-367 standard | Canonical | Normative definitions |
| 1 | Arnout dissertation | Canonical | Pattern classification |
| 1 | Meyer OOSC2 | Canonical | Methodology foundation |
| 2 | simple_* ecosystem | Non-canonical | Usage examples only (with caveat) |

**Critical**: simple_* is AI-generated and NOT canonical until KB-validated refactoring occurs.

### Arnout Pattern Classification (Foundation)

Karine Arnout (ETH 2004) analyzed all 23 GoF patterns for Eiffel:

**Built Into Eiffel** (no pattern needed):
- Prototype (clone/twin)
- Iterator (across, ITERABLE)
- Singleton (once features)

**Fully Componentized** (~65% of patterns):
- Observer → Event Library (ACTION_SEQUENCE)
- Visitor → Visitor Library (agents)
- Factory → Factory Library (generics + creation)
- Command → agents
- Composite → generics + inheritance
- Chain of Responsibility, Mediator, Flyweight

**Wizard Support** (skeleton generation):
- Adapter, Decorator, Bridge, Template Method

**Non-Componentizable** (too context-specific):
- Facade, Interpreter

### Study Plan (Arnout-Aligned)

| Category | Studies | Focus |
|----------|---------|-------|
| A: Built-In | 3 | What Eiffel provides natively |
| B: Componentized | 5 | Find and document the libraries |
| C: Wizard | 3 | Document skeleton patterns |
| D: Inheritance | 3 | Unique Eiffel capabilities |
| E: Contracts | 3 | DbC architecture patterns |

### Recommended Pilot Studies

| Study | Why First |
|-------|-----------|
| B1: Event/Observer | Well-documented, clear canonical source (ACTION_SEQUENCE) |
| D1: Multiple Inheritance | Unique to Eiffel, high value |
| E1: Precondition Idioms | Ubiquitous, easy to scan |
| A2: Iteration (across) | Modern Eiffel, many examples |

### Expected Outputs

| Output | Count | Quality |
|--------|-------|---------|
| Pattern studies | 17 | High (canonical sources) |
| FAQ pairs | ~400 | High (linked to living code) |
| Refactoring criteria | Per pattern | Validation rules |

### Success Criteria

1. Each FAQ links to canonical code example
2. Patterns validated against Arnout classification
3. simple_* refactoring can be measured against KB
4. RAG answers cite canonical sources

### Phases Completed

- [x] Phase 1: Content Audit (0 FAQs, identified gaps)
- [x] Phase 2: Schema Enhancement (10 new columns)
- [x] Phase 3: Study Planning (Arnout-aligned, source hierarchy defined)
- [ ] Phase 4: Pilot Studies (awaiting approval)
- [ ] Phase 5: Full Study Execution
- [ ] Phase 6: simple_* Refactoring

### References

- Arnout, K. (2004). From Patterns to Components. ETH Dissertation No. 15500.
  https://se.inf.ethz.ch/people/arnout/patterns/
- Gamma, E. et al. (1995). Design Patterns. Addison-Wesley.
- Meyer, B. (1997). Object-Oriented Software Construction, 2nd Ed.
- ECMA-367. Eiffel: Analysis, Design and Programming Language.



---

## Theoretical and Semantic Foundations

The KB enrichment must be grounded in deeper foundations, not just surface-level pattern cataloging.

### Software Theory Foundations

| Theory | Eiffel Manifestation | Impact on KB |
|--------|---------------------|--------------|
| **Type Theory** | Strong static typing, conformance rules, anchored types | FAQ must explain type semantics, not just syntax |
| **Formal Methods** | Design by Contract IS formal specification | Contracts are not "nice to have" - they are correctness proofs |
| **Hoare Logic** | {P} S {Q} = require/do/ensure | FAQ must teach precondition/postcondition as logical assertions |
| **Liskov Substitution** | Inheritance conformance rules | Explain WHY covariance/contravariance matters |
| **Information Hiding** | Export clauses, selective visibility | Not just HOW but WHY encapsulation |

### Semantic Layers

#### 1. Language Semantics (What Eiffel Constructs MEAN)

Not just syntax - operational and denotational meaning:

| Construct | Syntax | Semantics (What It MEANS) |
|-----------|--------|---------------------------|
|  | Precondition clause | "Caller MUST guarantee this before call" |
|  | Postcondition clause | "Callee GUARANTEES this after call" |
|  | Class invariant | "This is ALWAYS true for valid objects" |
|  | Previous value | "Captures state for delta assertions" |
|  | Non-void type | "This reference is GUARANTEED non-null" |
|  | SCOOP region | "This object lives in a different processor" |
|  | Memoized computation | "Computed exactly once per scope" |
|  | Abstract feature | "Subclass MUST provide implementation" |

**KB Impact**: FAQs must explain MEANING, not just usage.

#### 2. Business/Domain Semantics

How real-world concepts map to Eiffel constructs:

| Domain Concept | Eiffel Mapping | Semantic Relationship |
|----------------|----------------|----------------------|
| Entity | Class | Object identity |
| Relationship | Reference/inheritance | Association vs IS-A |
| Constraint | Contract | Business rule = precondition |
| State machine | Class with state invariant | Valid state transitions |
| Transaction | SCOOP region | Atomic operation boundary |

**KB Impact**: Show how domain modeling maps to code.

#### 3. Engineering Semantics (Design Meaning)

What design decisions MEAN for system properties:

| Design Choice | Semantic Meaning | System Property |
|---------------|------------------|-----------------|
| Multiple inheritance | Capability composition | Flexibility + complexity |
| Deferred class | Interface contract | Substitutability |
| Once feature | Singleton semantics | Global state (careful\!) |
| Agent | First-class behavior | Extensibility |
| Generic constraint | Type bound | Safety + expressiveness |

**KB Impact**: Explain trade-offs, not just mechanics.

#### 4. Scientific/Formal Semantics

The mathematical foundations:

| Concept | Mathematical Basis | Eiffel Feature |
|---------|-------------------|----------------|
| Assertion | Predicate logic | Boolean expressions in contracts |
| Loop correctness | Induction proof | Loop variant (decreasing function) |
| Partial correctness | Hoare triple | require/ensure |
| Total correctness | Termination + partial | Variant + contracts |
| Refinement | Subtype relation | Inheritance + contract strengthening |

**KB Impact**: Ground explanations in formal reasoning.

---

## Revised Study Approach: Semantic Depth

Each study should address THREE levels:

### Level 1: Syntactic (What)
- How is it written?
- What is the structure?

### Level 2: Semantic (What It Means)
- What does this construct GUARANTEE?
- What OBLIGATIONS does it impose?
- What is the INTENT?

### Level 3: Pragmatic (Why/When)
- When should you use this?
- What trade-offs exist?
- How does this relate to software theory?

### Example: Multiple Inheritance Study

**Level 1 (Syntactic)**:


**Level 2 (Semantic)**:
- C has all features of A and B
- C IS-A A and IS-A B (substitutable for both)
- Redefinition changes implementation, preserving interface
- Renaming resolves name clashes, creates new feature name

**Level 3 (Pragmatic)**:
- Use multiple inheritance for capability mixing (COMPARABLE + HASHABLE)
- Diamond problem solved via select (explicit disambiguation)
- Non-conforming inheritance for implementation reuse without IS-A
- Trade-off: Power vs complexity (C++/Java avoided this, Eiffel embraces it)

---

## Semantic Categories for FAQ Enrichment

### Category S1: Contract Semantics
- What does a precondition MEAN logically?
- How do contracts form a refinement lattice?
- What is the substitution principle for contracts?

### Category S2: Type Semantics  
- What does conformance MEAN?
- How do generics preserve type safety?
- What is the semantic difference between attached and detachable?

### Category S3: Concurrency Semantics (SCOOP)
- What does "separate" MEAN for object identity?
- How do wait conditions define synchronization?
- What is the region model semantically?

### Category S4: Inheritance Semantics
- What does IS-A mean formally?
- How does feature adaptation preserve semantics?
- What is the meaning of non-conforming inheritance?

### Category S5: Behavioral Semantics (Agents)
- What does capturing a closure MEAN?
- How do agents represent first-class behavior?
- What is the semantic relationship between agents and objects?

---

## Impact on Plan

### Before (Surface Level)
- Study: "How is multiple inheritance used?"
- FAQ: "Here is syntax for multiple inheritance"

### After (Semantic Depth)
- Study: "What does multiple inheritance MEAN for substitutability?"
- FAQ: "Multiple inheritance means C is substitutable for BOTH A and B, which requires..."

### Revised Pilot Studies

| Study | Semantic Focus |
|-------|---------------|
| B1: Observer/Event | What does publish-subscribe MEAN for coupling? |
| D1: Multiple Inheritance | What does IS-A mean when IS-A multiple things? |
| E1: Preconditions | What OBLIGATION does caller assume? What GUARANTEE does callee provide? |
| S1: Contract Semantics | How do Hoare triples map to Eiffel contracts? |

---

## References for Semantic Foundations

- Meyer, B. (1997). OOSC2 - Chapters on contracts, inheritance, genericity
- Hoare, C.A.R. (1969). An Axiomatic Basis for Computer Programming
- Liskov, B. (1987). Data Abstraction and Hierarchy
- ECMA-367 - Formal semantics sections
- Arnout (2004) - Pattern semantics in Eiffel context

