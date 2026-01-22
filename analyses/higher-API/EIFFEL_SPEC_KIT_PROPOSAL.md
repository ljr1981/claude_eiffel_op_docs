# Eiffel Spec Kit: Hybrid Methodology Proposal

**Date:** January 21, 2026
**Status:** Draft Proposal
**Authors:** Larry Rix + Claude (Anthropic)

---

## 1. Executive Summary

This proposal outlines a hybrid methodology combining:
- GitHub Spec Kit's UX patterns (phases, slash commands, directory conventions)
- The Anti-Slop Workflow's verification rigor (compilation gates, evidence requirements)
- Eiffel's Design by Contract as the specification mechanism

The result is a workflow where specifications are machine-checkable contracts, not markdown documents interpreted by AI.

---

## 2. Background

### 2.1 GitHub Spec Kit

GitHub released Spec Kit as an open-source toolkit for specification-driven development with AI agents. Key characteristics:

| Aspect | Spec Kit Approach |
|--------|-------------------|
| Spec format | Markdown files |
| Verification | AI interprets specs |
| Enforcement | Honor system |
| Phases | 4 (specify, plan, tasks, implement) |
| Agents | 16+ supported (Claude, Copilot, Cursor, etc.) |

**Strengths:**
- Clear separation of WHAT (specify) from HOW (plan)
- Task decomposition before implementation
- Slash command interface for easy invocation
- Standard directory convention (`.specify/`)

**Weaknesses:**
- Specs are markdown, not code (no compiler verification)
- No compilation gates
- No evidence requirements
- Verification relies on AI following instructions

### 2.2 Anti-Slop Workflow (Current Methodology)

The existing Simple Eiffel methodology consists of 12 workflows with explicit verification gates.

| Aspect | Anti-Slop Approach |
|--------|-------------------|
| Spec format | Eiffel contracts (compiled code) |
| Verification | Runtime contract checking |
| Enforcement | Mandatory gates + evidence |
| Phases | 12 workflows, numbered steps |
| Layer model | Spec → Contract → Code → Test |

**Strengths:**
- Contracts are machine-checkable
- Compilation gates enforce real work
- Evidence requirements prevent AI hallucination
- Runtime verification catches logic errors
- Four-layer alignment model

**Weaknesses:**
- Workflow navigation requires reading INDEX files
- No explicit WHAT/WHY phase before contracts
- Task decomposition embedded in steps, not explicit
- No slash command interface

### 2.3 The Opportunity

Eiffel's Design by Contract already provides what Spec Kit tries to achieve with markdown:

```
Spec Kit:     Markdown Spec → AI Interprets → Code → Hope
Eiffel DbC:   Contract → Compiler Checks → Code → Runtime Checks → Tests
```

The gap is UX, not capability. Eiffel has superior verification but inferior workflow ergonomics.

---

## 3. Comparative Analysis

### 3.1 Phase Mapping

| Spec Kit Phase | Current Anti-Slop | Gap |
|----------------|-------------------|-----|
| `/speckit.specify` (WHAT) | None explicit | Missing |
| `/speckit.plan` (HOW) | Embedded in 01 | Implicit |
| `/speckit.tasks` | None explicit | Missing |
| `/speckit.implement` | 06-IMPLEMENT | Present |
| N/A | 07-COMPILE-CHECK | Spec Kit lacks |
| N/A | 08-RUN-CONTRACTS | Spec Kit lacks |
| N/A | 09-GENERATE-TESTS | Spec Kit lacks |

### 3.2 Verification Comparison

| Verification Type | Spec Kit | Anti-Slop |
|-------------------|----------|-----------|
| Spec consistency | AI interpretation | Compiler |
| Type safety | Language-dependent | Eiffel compiler |
| Null safety | Language-dependent | Void safety |
| Behavior correctness | Hope | Runtime contracts |
| Compilation gate | None | Mandatory |
| Test gate | "NON-NEGOTIABLE" (unenforced) | Mandatory |
| Evidence required | No | Yes |

### 3.3 Spec Format Comparison

**Spec Kit (markdown):**
```markdown
## User Stories
- As a user, I want to cache frequently accessed data
- Cache should evict least-recently-used items when full

## Acceptance Criteria
- [ ] Cache stores key-value pairs
- [ ] LRU eviction when capacity exceeded
- [ ] O(1) get and put operations
```

**Eiffel (contracts):**
```eiffel
put (a_key: K; a_value: V)
    require
        key_not_void: a_key /= Void
        not_full_or_can_evict: count < capacity or else count > 0
    ensure
        stored: item (a_key) = a_value
        count_bounded: count <= capacity
        lru_evicted: old count = capacity implies count = capacity
    end
```

**Difference:** Spec Kit's criteria are checked by humans reading AI output. Eiffel's contracts are checked by the runtime on every call.

---

## 4. Proposed Hybrid: Eiffel Spec Kit

### 4.1 Phase Structure

```
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 0: INTENT                                                │
│  Command: /eiffel.intent                                        │
│  Input: Natural language description                            │
│  Output: intent.md (WHAT users need, WHY they need it)          │
│  Gate: Human approval                                           │
│  Source: Spec Kit                                               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1: CONTRACTS                                             │
│  Command: /eiffel.contracts                                     │
│  Input: intent.md                                               │
│  Output: Class skeletons with require/ensure/invariant          │
│  Gate: MUST COMPILE                                             │
│  Source: Anti-Slop 01-05                                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 2: ADVERSARIAL REVIEW                                    │
│  Command: /eiffel.review                                        │
│  Input: Compiled contracts from Phase 1                         │
│  Output: review.md (critiques, gaps, suggestions from other AI) │
│  Gate: Human reviews feedback, approves or routes to Phase 1    │
│  Source: NEW - addresses single-AI blindspot problem            │
│                                                                 │
│  WHY: The primary AI (Claude) may hallucinate contract          │
│  correctness, miss edge cases, or encode assumptions the human  │
│  shares but shouldn't. Submitting contracts to Grok, Gemini,    │
│  or other AIs for adversarial review catches blind spots.       │
│  This phase makes cross-AI review MANDATORY, not optional.      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 3: TASKS                                                 │
│  Command: /eiffel.tasks                                         │
│  Input: Reviewed contracts (approved after Phase 2)             │
│  Output: tasks.md (implementation chunks with acceptance)       │
│  Gate: Human approval                                           │
│  Source: Spec Kit                                               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 4: IMPLEMENT                                             │
│  Command: /eiffel.implement                                     │
│  Input: tasks.md + reviewed contracts                           │
│  Output: Feature bodies (contracts UNCHANGED)                   │
│  Gate: MUST COMPILE                                             │
│  Source: Anti-Slop 06-07                                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 5: VERIFY                                                │
│  Command: /eiffel.verify                                        │
│  Input: Compiled implementation                                 │
│  Output: Test suite derived from contracts                      │
│  Gate: TESTS MUST PASS                                          │
│  Source: Anti-Slop 08-09                                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 6: HARDEN                                                │
│  Command: /eiffel.harden                                        │
│  Input: Passing test suite                                      │
│  Output: Adversarial tests, stress tests, edge cases            │
│  Gate: ALL TESTS MUST PASS                                      │
│  Source: Anti-Slop 07_maintenance-xtreme                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 7: SHIP                                                  │
│  Command: /eiffel.ship                                          │
│  Input: Hardened library                                        │
│  Output: Naming review, docs, GitHub prep, ecosystem check      │
│  Gate: Checklist complete                                       │
│  Source: Anti-Slop 10-12                                        │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Directory Structure

```
{library}/
├── .eiffel-workflow/
│   ├── intent.md              # Phase 0 output (WHAT/WHY)
│   ├── review.md              # Phase 2 output (adversarial AI feedback)
│   ├── tasks.md               # Phase 3 output (implementation plan)
│   └── evidence/
│       ├── phase1-compile.txt # Compilation output
│       ├── phase2-review.txt  # Adversarial review output (which AI, key findings)
│       ├── phase4-compile.txt
│       ├── phase5-tests.txt   # Test execution output
│       └── phase6-tests.txt
├── src/
│   └── *.e                    # Eiffel source with contracts
├── test/
│   └── *.e                    # Test classes
└── {library}.ecf              # Project configuration
```

### 4.3 Slash Commands

| Command | Phase | Action |
|---------|-------|--------|
| `/eiffel.intent` | 0 | Generate intent.md from description |
| `/eiffel.contracts` | 1 | Generate class skeletons with contracts |
| `/eiffel.review` | 2 | Submit contracts to other AIs for adversarial review |
| `/eiffel.tasks` | 3 | Break contracts into implementation tasks |
| `/eiffel.implement` | 4 | Write feature bodies (contracts frozen) |
| `/eiffel.verify` | 5 | Generate and run contract-based tests |
| `/eiffel.harden` | 6 | Adversarial and stress testing |
| `/eiffel.ship` | 7 | Naming, docs, GitHub, ecosystem |
| `/eiffel.status` | Any | Show current phase and evidence |

### 4.4 Gate Enforcement

**Compilation Gates (Phases 1, 4):**
```bash
# Mandatory before proceeding
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# Evidence: Paste output to .eiffel-workflow/evidence/phaseN-compile.txt
# Required: "System Recompiled." message present
# Failure: Cannot proceed until fixed
```

**Adversarial Review Gate (Phase 2):**
```
# Mandatory before proceeding to implementation
1. Export contracts to portable format (Eiffel source or summary)
2. Submit to at least ONE other AI (Grok, Gemini, GPT-4, etc.)
3. Prompt: "Review these Eiffel contracts for: missing preconditions,
   weak postconditions, edge cases not covered, implicit assumptions,
   potential runtime failures. Be adversarial."
4. Record findings in .eiffel-workflow/review.md
5. Record which AI and key findings in evidence/phase2-review.txt

# Required: At least one external AI review completed
# Failure: Cannot proceed until review done and findings addressed
# Routing: If review reveals contract gaps → return to Phase 1

WHY THIS MATTERS: The primary AI and human often share blind spots.
The human depends on AI expertise but AI hallucinates. Cross-AI review
is a sanity check that catches errors neither party would catch alone.
Without a mandated phase, this review becomes a "remember to do it"
chore that humans forget. The tool sits unused in the toolbox.
```

**Test Gates (Phases 5, 6):**
```bash
# Mandatory before proceeding
./EIFGENs/{lib}_tests/W_code/{lib}.exe

# Evidence: Paste output to .eiffel-workflow/evidence/phaseN-tests.txt
# Required: "ALL TESTS PASSED" or "0 failed"
# Failure: Cannot proceed until fixed
```

**Human Gates (Phases 0, 3):**
- Phase 0: User reviews intent.md, confirms it captures requirements
- Phase 3: User reviews tasks.md, confirms decomposition is appropriate

### 4.5 Anti-Slop Rules (Unchanged)

All existing Anti-Slop rules remain in force:

1. **No hypothetical results** - "would compile" is forbidden
2. **Evidence required** - Every claim backed by pasted output
3. **Compile before document** - Code changes require compilation
4. **No imagined behavior** - Test claims require actual test execution
5. **Document ACTUAL not EXPECTED** - Results from execution, not prediction

### 4.6 Contract Rules

Contracts serve as the specification. Rules:

1. **Contracts written before implementation** (Phase 1 before Phase 4)
2. **Contracts reviewed adversarially** (Phase 2 before Phase 3)
3. **Contracts frozen during implementation** (Phase 4 cannot modify require/ensure/invariant)
4. **Contract violations route to contract phase** (not implementation phase)
5. **Implementation violations route to implementation phase** (not contract phase)

**Failure Routing:**
```
If precondition fails → Caller is wrong OR precondition is wrong
  → Route to Phase 1 (contract review)
  → Consider Phase 2 re-review if fundamental

If postcondition fails → Implementation is wrong OR postcondition is wrong
  → Route to Phase 4 if code bug
  → Route to Phase 1 if spec bug

If invariant fails → Class state is corrupted
  → Route to Phase 4 (implementation fix)

If adversarial review finds gaps → Contract is incomplete
  → Route to Phase 1 (contract strengthening)
  → Re-run Phase 2 after changes
```

---

## 5. Comparison with Spec Kit Claims

### 5.1 "TDD is NON-NEGOTIABLE"

Spec Kit claims: *"No implementation code shall be written before tests are created."*

**Problem:** No enforcement mechanism. AI can ignore this.

**Eiffel Spec Kit solution:** Contracts ARE the specification. Phase 1 creates contracts before Phase 3 creates implementation. Compilation gate enforces this ordering.

### 5.2 "Specifications become executable"

Spec Kit claims: *"Specifications become executable, directly generating working implementations."*

**Problem:** Markdown specs are not executable. AI interprets them.

**Eiffel Spec Kit solution:** Contracts ARE executable. The runtime checks them on every feature call.

### 5.3 "Intent is the source of truth"

Spec Kit claims: *"The shift moves from code is the source of truth to intent is the source of truth."*

**Problem:** Intent in markdown can drift from implementation.

**Eiffel Spec Kit solution:** Intent is captured in contracts. Contracts are compiled code. Drift is detected by contract violations.

---

## 6. Implementation Roadmap

### Phase 1: Document the Methodology

1. Write detailed phase guides (like existing workflow steps)
2. Create intent.md template
3. Create tasks.md template
4. Define evidence file formats

### Phase 2: Create Slash Command Skill

1. Implement `/eiffel.intent` as Claude Code skill
2. Implement `/eiffel.contracts`
3. Implement remaining commands
4. Add `/eiffel.status` for progress tracking

### Phase 3: Tooling Integration

1. Create `.eiffel-workflow/` directory initializer
2. Add evidence file generation
3. Integrate with simple_oracle for session handoff

### Phase 4: Validation

1. Apply to new library creation
2. Apply to existing library hardening
3. Document friction points
4. Iterate on workflow

---

## 7. Risk Assessment

| Risk | Mitigation |
|------|------------|
| Overhead too high | Phase 0 and 2 are lightweight; gates already exist |
| AI ignores gates | Gates are enforced by human review of evidence files |
| Contracts too verbose | Start with key features, expand coverage iteratively |
| Tooling not ready | Manual execution first, automate later |

---

## 8. Success Metrics

| Metric | Target |
|--------|--------|
| Contract coverage | 100% of public features have require/ensure |
| Test coverage | 100% of contracts exercised by tests |
| Evidence compliance | 100% of gates have evidence files |
| Defect escape rate | 0 defects found post-Phase 5 |

---

## 9. Conclusion

GitHub Spec Kit addresses a real problem: vibe coding produces unreliable software. Their solution (markdown specs interpreted by AI) is appropriate for languages without contract mechanisms.

Eiffel has contracts. The appropriate solution for Eiffel is:
1. Adopt Spec Kit's UX patterns (phases, commands, directories)
2. Keep Anti-Slop's verification rigor (gates, evidence)
3. Use contracts as the specification mechanism (not markdown)

The result: specifications that are machine-checkable, not hope-based.

---

## Appendix A: Spec Kit Command Mapping

| Spec Kit | Eiffel Spec Kit | Difference |
|----------|-----------------|------------|
| `/speckit.constitution` | N/A | Eiffel has OOSC2 + DbC principles |
| `/speckit.specify` | `/eiffel.intent` | Output is intent.md, not spec.md |
| `/speckit.plan` | `/eiffel.contracts` | Output is compiled contracts, not plan.md |
| `/speckit.tasks` | `/eiffel.tasks` | Same concept |
| `/speckit.implement` | `/eiffel.implement` | Same concept + compilation gate |
| `/speckit.clarify` | N/A | Contracts clarify by definition |
| `/speckit.analyze` | `/eiffel.status` | Shows evidence, not just status |
| `/speckit.checklist` | N/A | Gates are the checklist |

---

## Appendix B: Evidence File Format

**Compilation evidence:**
```
# Phase 1 Compilation Evidence
# Date: 2026-01-21 18:30:00
# Command: /d/prod/ec.sh -batch -config simple_cache.ecf -target simple_cache_tests -c_compile

Eiffel Compilation Manager
Version 25.02.9.8732 - win64

Degree 6: Examining System
Degree 5: Parsing Classes
Degree 4: Analyzing Inheritance
Degree 3: Checking Types
Degree 2: Generating Byte Code
Degree 1: Generating Metadata
Melting System Changes
System Recompiled.

# Status: PASS
```

**Test evidence:**
```
# Phase 4 Test Evidence
# Date: 2026-01-21 18:35:00
# Command: ./EIFGENs/simple_cache_tests/W_code/simple_cache.exe

Running SIMPLE_CACHE tests...

  PASS: test_put_get
  PASS: test_lru_eviction
  PASS: test_capacity_limit
  ...

========================
Results: 47 passed, 0 failed
ALL TESTS PASSED

# Status: PASS
```

---

## Appendix C: References

1. GitHub Spec Kit Repository
   https://github.com/github/spec-kit

2. GitHub Blog: Spec-driven development with AI
   https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/

3. Anti-Slop Workflow Documentation
   D:\prod\reference_docs\research\06_methodology\AI-slop\prompts\

4. OOSC2 (Object-Oriented Software Construction, 2nd Edition)
   Bertrand Meyer, 1997

5. Simple Eiffel Strategic Analysis
   D:\prod\reference_docs\analyses\SIMPLE_EIFFEL_STRATEGIC_ANALYSIS_2026.md
