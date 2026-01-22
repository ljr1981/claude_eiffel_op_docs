# Eiffel Spec Kit: A Methodology Experiment

**Date:** January 21, 2026
**Status:** Draft Proposal (Experimental)
**Authors:** Larry Rix + Claude (Anthropic)

---

## 1. Executive Summary

This proposal outlines a hybrid methodology combining:
- GitHub Spec Kit's UX patterns (phases, slash commands, directory conventions)
- The Anti-Slop Workflow's verification rigor (compilation gates, evidence requirements)
- Eiffel's Design by Contract as the specification mechanism

The result is a workflow where specifications are machine-checkable contracts, not markdown documents interpreted by AI.

**The honest caveat:** This is an experiment, not a proven solution. The companion Strategic Analysis document reveals that:
- The existing Anti-Slop methodology has not been consistently followed
- Methodologies evolve unevenly and can fail entirely (simple_code)
- AI will forget, ignore, drop, hallucinate, and seize control regardless of workflow
- The human enforcement layer is unreliable—humans forget, get tired, and take shortcuts
- Adding more phases may just create more phases to skip

This proposal may produce the same outcome as previous methodology attempts: initial enthusiasm followed by gradual abandonment as the overhead becomes burdensome. We document it anyway because the alternative—no methodology—has demonstrably failed.

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

**The uncomfortable truth:** These weaknesses are not the real problem. The Anti-Slop workflows exist and are documented. The problem is that they weren't consistently followed during the 114-library buildout. The human got tired. The AI got eager. Volume overwhelmed process. Adding a slash command interface won't fix human fatigue.

### 2.3 The Opportunity (Qualified)

Eiffel's Design by Contract already provides what Spec Kit tries to achieve with markdown:

```
Spec Kit:     Markdown Spec → AI Interprets → Code → Hope
Eiffel DbC:   Contract → Compiler Checks → Code → Runtime Checks → Tests
```

The gap is UX, not capability. Eiffel has superior verification but inferior workflow ergonomics.

**But:** Better ergonomics won't guarantee compliance. The Anti-Slop methodology had clear steps. They were skipped anyway. The question is whether slash commands and directory conventions create enough friction reduction to change behavior. The honest answer: probably not enough on their own.

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

**Note on "Mandatory":** The Anti-Slop gates are mandatory in the documentation. In practice, they were often skipped. "Mandatory" means nothing without enforcement, and enforcement requires a human who is paying attention. See Part 2b of the Strategic Analysis for why this assumption fails.

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

**The catch:** Eiffel's contracts are only as good as they're written. The X03 Contract Assault (January 21, 2026) revealed contract quality problems in three categories, with real examples from the commit history:

- **Wrong**: Factually incorrect or redundant contracts.
  - *simple_mml* (commit e46f934): All MML classes had explicit `[G -> detachable separate ANY]` generic constraints. Per Eric Bezault, this is the default—the constraint was redundant. Wrong contracts waste reader attention and imply the constraint matters when it doesn't.

- **Weak**: Contracts that exist but don't constrain enough.
  - Example: `ensure Result /= Void` when you could specify `ensure Result ~ expected_value`. The contract is true but doesn't capture the actual requirement. Tests pass, but the specification is nearly useless.

- **Incomplete**: Contracts missing critical pieces needed for verification.
  - *simple_regex* (commit 1677d0b): `SIMPLE_REGEX_MATCH` lacked an `is_equal` override. MML postconditions using `|=|` silently used reference equality instead of value equality. Contracts that compared match results after `deep_twin` failed—not because the values differed, but because the class couldn't express value equality. Tests passed for weeks before contract strengthening revealed the defect.
  - *simple_process* (commit 46b1661): `execute_in_directory` had postconditions `execution_recorded: execution_count = old execution_count + 1` but the feature didn't update `execution_count_impl` or `last_command`. The postconditions were correct specifications, but the implementation didn't maintain the state to satisfy them. The feature "worked" for its primary purpose but violated its own contracts.

The runtime can only check what the contract specifies—and what the implementation maintains. Garbage in, garbage out.

---

## 4. Proposed Hybrid: Eiffel Spec Kit

### 4.1 Phase Structure

```
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 0: INTENT                                                │
│  Command: /eiffel.intent                                        │
│  Input: Natural language description                            │
│  Output: intent.md (WHAT users need, WHY they need it)          │
│  Gate: AI-assisted review + Human answers (see below)           │
│  Source: Spec Kit + NEW adversarial intent review               │
│                                                                 │
│  RISK: Human approves quickly to move on. Intent is vague.     │
│  AI later interprets vague intent in unexpected ways.           │
│                                                                 │
│  MITIGATION: Mandatory AI-assisted intent review. A different   │
│  AI (via simple_ai_client) reviews the intent and generates     │
│  probing questions the human MUST answer. Human can't just      │
│  click approve—they must provide substantive responses.         │
│  Questions + answers become refined intent.md v2.               │
└─────────────────────────────────────────────────────────────────┘

**Phase 0 Detail: AI-Assisted Intent Review**

The original risk—human rubber-stamps vague intent—is addressed by inserting
an AI reviewer that forces engagement:

```
Step 0a: Generate initial intent.md (primary AI)
Step 0b: Submit to adversarial AI for review (simple_ai_client)
Step 0c: Human answers AI's probing questions (cannot skip)
Step 0d: Generate refined intent.md v2 incorporating answers
Step 0e: Human approves final intent (now substantive, not rubber-stamp)
```

**CLI Tool: `simple_ai_client review-intent`**

```bash
$ simple_ai_client review-intent intent.md --ai gemini

Reviewing intent.md with Gemini...

Gemini asks:
1. You said "users can manage data" — which operations specifically?
   (create/read/update/delete? subset? batch operations?)
2. You said "fast" — what latency is acceptable?
   (100ms? 1s? 10s? varies by operation?)
3. You said "secure" — against what threat model?
   (XSS? SQL injection? auth bypass? data exfiltration?)
4. Who is the user? (developer using API? end-user via UI? admin?)
5. What happens on failure? (retry? error message? silent fail? rollback?)
6. You mentioned "cache" — what's the eviction policy? TTL? LRU? explicit?
7. What are the edge cases? (empty input? max size? concurrent access?)

Enter your answers (or 'skip' to mark as TODO):

1> CRUD operations only, no batch
2> 100ms for reads, 1s for writes acceptable
3> SQL injection and auth bypass; XSS handled by frontend
...

Generating refined intent.md...
Written to: intent-v2.md

Review complete. 7 questions answered, 0 skipped.
```

**Why This Might Actually Work:**

1. **Removes "did I think of everything" fatigue** — AI finds the vague parts
2. **AI is tireless** — Will ask about every undefined term
3. **Human can't just approve** — Must type actual answers
4. **Different AI = different blind spots** — Gemini catches what Claude missed
5. **Answers create audit trail** — intent-v2.md shows what was clarified
6. **Early vagueness = late bugs** — Catching it in Phase 0 is cheapest

**Implementation in simple_ai_client:**

```eiffel
review_intent (a_file: PATH; a_ai: STRING)
    -- Review intent document using adversarial AI.
    require
        file_exists: a_file.exists
        ai_supported: supported_ais.has (a_ai)
    local
        l_content: STRING
        l_prompt: STRING
        l_questions: LIST [STRING]
        l_answers: LIST [STRING]
    do
        l_content := read_file (a_file)
        l_prompt := intent_review_prompt (l_content)
        l_questions := call_ai (a_ai, l_prompt).split_questions
        l_answers := prompt_user_for_answers (l_questions)
        write_refined_intent (a_file, l_content, l_questions, l_answers)
    ensure
        refined_exists: (a_file.parent / "intent-v2.md").exists
        all_answered: across answers as a all not a.is_empty or a.is_todo end
    end
```

**The Review Prompt:**

```
Review this intent document for a software library. Your job is to find
vague language, undefined terms, missing acceptance criteria, implicit
assumptions, and edge cases that aren't addressed.

Ask 5-10 probing questions. Each question should:
- Quote the vague phrase
- Explain why it's vague
- Offer concrete alternatives the user can choose from

Be adversarial. Assume the intent will be misinterpreted if not clarified.

Intent document:
---
{content}
---
```
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1: CONTRACTS                                             │
│  Command: /eiffel.contracts                                     │
│  Input: intent.md                                               │
│  Output: Class skeletons with require/ensure/invariant          │
│  Gate: MUST COMPILE                                             │
│  Source: Anti-Slop 01-05                                        │
│                                                                 │
│  RISK: Contracts compile but are weak. "True" preconditions,   │
│  postconditions that don't capture actual requirements.         │
│  Compilation proves syntax, not semantics.                      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 2: ADVERSARIAL REVIEW + APPROACH SKETCH                  │
│  Command: /eiffel.review                                        │
│  Input: Compiled contracts from Phase 1                         │
│  Output: review.md, approach.md, MML model queries              │
│  Gate: Human reviews feedback, approves or routes to Phase 1    │
│  Source: NEW - addresses single-AI blindspot + early MML        │
│                                                                 │
│  WHY: The primary AI (Claude) may hallucinate contract          │
│  correctness, miss edge cases, or encode assumptions the human  │
│  shares but shouldn't. Cross-AI review catches blind spots.     │
│                                                                 │
│  RISK: Human skips this phase because it's tedious. Copy-paste │
│  to another AI, wait for response, read critique, decide what  │
│  to address... the friction is real. This phase is most likely │
│  to be abandoned.                                               │
└─────────────────────────────────────────────────────────────────┘

**Phase 2 Detail: Substeps**

```
Step 2a: Add MML model queries to contracts (EARLY MML)
         - Add model queries that postconditions can reference
         - Example: items_model: MML_SET [G] for collection classes
         - Strengthens contracts BEFORE implementation

Step 2b: Generate implementation SKETCH (not real code)
         - Pseudocode or outline showing intended approach
         - "I plan to use a hash table for O(1) lookup"
         - "Recursion with memoization for fibonacci"
         - Does NOT compile—it's an approach document

Step 2c: PROGRESSIVE AI REVIEW CHAIN

         Pass 1 - Ollama (weakest, local):
                  First pass at finding obvious issues
                  "This precondition is just 'True'—seems weak"
                  Low cost, fast, catches surface problems

         Pass 2 - Claude critiques Ollama's findings + original:
                  "Ollama was right about the precondition. Also missing:
                   empty input handling, frame condition for unchanged keys"

         Pass 3 - Grok critiques Claude's work:
                  "Claude added frame condition but MML postcondition is O(n²)..."
                  "The sketch assumes sorted input but precondition doesn't require it"

         Pass 4 - Gemini (strongest) final AI review:
                  "All previous reviewers missed: SCOOP safety during iteration"
                  "Edge case: what if capacity is zero?"

         Pass 5 - Human reviews SYNOPSIS:
                  Sees: Concise summary of all issues found by chain
                  NOT raw AI output—distilled, actionable list
                  Can ask clarifying questions of any AI in chain
                  NOT assumed to be subject matter expert

Step 2d: Refine contracts and approach based on findings
         - Strengthen weak contracts
         - Add missing MML model queries
         - Adjust approach if algorithm issues found

Step 2e: Human approves refined output (or routes back to Phase 1)
```

**Why Progressive AI Review Chain:**

| Pass | Reviewer | Role | Cost |
|------|----------|------|------|
| 1 | Ollama (local) | Surface problems, obvious gaps | Free |
| 2 | Claude | Deep Eiffel expertise, strengthens contracts | Medium |
| 3 | Grok | Different perspective, catches Claude's assumptions | Medium |
| 4 | Gemini | Final AI pass, strongest reasoning | Higher |
| 5 | Human | Reviews synopsis, asks clarifying questions | Cognitive |

**The key insight:** Each AI "gangs up" on the previous work. By the time the human sees it, multiple AI perspectives have already beaten on the specs. Human receives a **synopsis**, not raw output—and can ask any AI in the chain for clarification if something is unclear. Human is NOT assumed to be a subject matter expert.

**CLI Flow (all automated via simple_ai_client):**

```bash
# Full review chain - ONE command
$ simple_ai_client review-chain src/ --output synopsis.md

[1/4] Ollama reviewing contracts...
      Found: 3 weak preconditions, 1 missing postcondition

[2/4] Claude reviewing (includes Ollama findings)...
      Found: +2 missing frame conditions, +1 MML model query needed
      Confirmed: Ollama's 3 weak preconditions

[3/4] Grok reviewing (includes Claude findings)...
      Found: +1 complexity mismatch (O(n²) vs claimed O(1))
      Disputed: Claude's frame condition suggestion (argues unnecessary)

[4/4] Gemini reviewing (final AI pass)...
      Found: +1 SCOOP safety issue during iteration
      Resolved: Agrees with Grok on frame condition dispute

Generating synopsis.md...

═══════════════════════════════════════════════════════════════
SYNOPSIS FOR HUMAN REVIEW
═══════════════════════════════════════════════════════════════
8 issues found, 1 disputed (resolved by Gemini)

MUST FIX:
  1. Precondition in `put`: add `not is_full` check
  2. Postcondition in `remove`: add frame condition for other keys
  3. MML model query missing: `access_order_model` for LRU

SHOULD FIX:
  4. Complexity: MML postcondition is O(n²), consider lazy evaluation
  5. SCOOP: `put` during `across` iteration undefined behavior

REVIEW NEEDED:
  6-8. [Details in synopsis.md]

To ask clarifying questions:
  $ simple_ai_client clarify 3 --ask "Why is access_order needed?"
═══════════════════════════════════════════════════════════════
```

**The human's cognitive load:** Read a prioritized list. Ask questions if unclear. Approve or reject. That's it.

**Early MML Application (Step 2a)**

MML should be applied at the CONTRACT stage, not after implementation:

```eiffel
-- BEFORE MML (Phase 1 output - weak)
class SIMPLE_CACHE [K, V]

feature -- Commands
    put (a_key: K; a_value: V)
        require
            key_valid: a_key /= Void
        ensure
            has_key: has (a_key)
        end

-- AFTER MML (Phase 2a output - strong)
class SIMPLE_CACHE [K, V]

feature -- Model Queries
    keys_model: MML_SET [K]
            -- Model of all keys in cache.
        do
            create Result.make_empty
            across internal_table as ic loop
                Result := Result & @ic.key
            end
        end

    entries_model: MML_MAP [K, V]
            -- Model of key-value pairs.

feature -- Commands
    put (a_key: K; a_value: V)
        require
            key_valid: a_key /= Void
        ensure
            key_added: keys_model.has (a_key)
            value_stored: entries_model [a_key] = a_value
            count_updated: keys_model.count = old keys_model.count +
                          (if old keys_model.has (a_key) then 0 else 1 end)
            others_unchanged: keys_model.removed (a_key) |=| old keys_model.removed (a_key)
        end
```

**Why MML Early:**
1. Forces thinking about WHAT changes, not just that something changed
2. Frame conditions (what DIDN'T change) become natural
3. Postconditions become testable and precise
4. Implementation must maintain model state—discovered issues early
5. Adversarial AI can spot MML weaknesses before implementation

**The MML Progression: Prose → Code → Evidence**

MML thinking can be applied logically in prose before it becomes code:

```
Phase 2 (prose in approach.md):
    "The set of keys grows by one when adding a new key.
     Existing keys are preserved. The new key maps to the given value."

Phase 2a (MML code in contracts):
    ensure
        key_added: keys_model.has (a_key)
        count_updated: keys_model.count = old keys_model.count +
                      (if old keys_model.has (a_key) then 0 else 1 end)
        value_stored: entries_model [a_key] = a_value
        others_unchanged: keys_model.removed (a_key) |=| old keys_model.removed (a_key)

Phase 5/6 (tests provide evidence):
    test_put_new_key_increases_count        -- PASS
    test_put_existing_key_preserves_count   -- PASS
    test_put_preserves_other_keys           -- PASS
    test_put_stores_correct_value           -- PASS
```

Each stage adds rigor:
- **Prose**: Captures intent in natural language (can be reviewed by non-Eiffel readers)
- **Code**: Makes intent machine-checkable (compiler verifies syntax, runtime verifies behavior)
- **Evidence**: Proves intent is achieved (tests demonstrate postconditions hold)

The prose MML in approach.md is a "draft" of the thinking. The adversarial AI can review this draft and ask "did you mean X or Y?" before it becomes code that's harder to change.

**Implementation Sketch Example (Step 2b)**

```markdown
## approach.md for SIMPLE_CACHE

### Data Structure
- Internal hash table (HASH_TABLE [V, K])
- Separate doubly-linked list for LRU ordering
- Hash table values point to list nodes for O(1) LRU update

### Algorithm: put
1. If key exists: update value, move node to front of LRU list
2. If key doesn't exist and cache full: evict LRU (tail of list)
3. Insert new entry at front of LRU list
4. Update hash table

### Complexity
- put: O(1) average
- get: O(1) average
- eviction: O(1)

### Edge Cases
- Empty cache
- Single-element cache (head = tail)
- Duplicate puts (update vs insert)
- Capacity = 0 (degenerate case—always evict?)
```

**Progressive Chain Reviews Sketch (Step 2c)**

The full AI chain (Ollama → Claude → Grok → Gemini) reviews BOTH contracts AND approach.md:

```
$ simple_ai_client review-chain src/ approach.md --output synopsis.md

Chain findings (accumulated):
  Ollama: "Precondition missing: capacity > 0 not required"
  Claude: "O(1) claim in sketch vs O(n) MML postcondition—mismatch"
  Grok:   "LRU ordering not in contracts but sketch assumes it"
  Gemini: "SCOOP: put during iteration is undefined"

Human synopsis: 4 issues, all MUST FIX before implementation.
```

This catches architectural issues BEFORE writing real code.
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 3: TASKS                                                 │
│  Command: /eiffel.tasks                                         │
│  Input: Reviewed contracts (approved after Phase 2)             │
│  Output: tasks.md (implementation chunks with acceptance)       │
│  Gate: Human approval                                           │
│  Source: Spec Kit                                               │
│                                                                 │
│  RISK: Task list becomes stale as implementation reveals        │
│  unforeseen complexity. Nobody updates tasks.md. It becomes     │
│  a historical artifact, not a living document.                  │
│                                                                 │
│  MITIGATION: AI chain reviews tasks.md for completeness.        │
│  "Task 3 doesn't cover the LRU eviction mentioned in approach." │
│  Human approves task list AFTER chain review.                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 4: IMPLEMENT                                             │
│  Command: /eiffel.implement                                     │
│  Input: tasks.md + reviewed contracts                           │
│  Output: Feature bodies (contracts UNCHANGED)                   │
│  Gate: MUST COMPILE                                             │
│  Source: Anti-Slop 06-07                                        │
│                                                                 │
│  RISK: AI modifies contracts anyway when implementation is      │
│  hard. "I need to weaken this precondition to make it work."    │
│  Human allows it because fighting the AI is exhausting.         │
│  Contract-first becomes contract-when-convenient.               │
│                                                                 │
│  MITIGATION: simple_ai_client diff-contracts detects changes.   │
│  "WARNING: precondition 'is_valid' was weakened from 'x > 0'    │
│  to 'x >= 0'. Run review-chain to verify this is acceptable."   │
│  Contract modifications require re-running Phase 2 chain.       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 5: VERIFY                                                │
│  Command: /eiffel.verify                                        │
│  Input: Compiled implementation                                 │
│  Output: Test suite derived from contracts                      │
│  Gate: TESTS MUST PASS                                          │
│  Source: Anti-Slop 08-09                                        │
│                                                                 │
│  RISK: Tests are weak. Happy-path only. Tests pass because      │
│  they don't test the hard cases. "All tests pass" creates       │
│  false confidence.                                              │
│                                                                 │
│  MITIGATION: AI chain reviews test suite against contracts.     │
│  "Postcondition 'others_unchanged' has no corresponding test."  │
│  "Edge case: empty input not tested despite precondition."      │
│  Chain generates test coverage gap report before Phase 6.       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 6: HARDEN                                                │
│  Command: /eiffel.harden                                        │
│  Input: Passing test suite                                      │
│  Output: Adversarial tests, stress tests, edge cases            │
│  Gate: ALL TESTS MUST PASS                                      │
│  Source: Anti-Slop 07_maintenance-xtreme                        │
│                                                                 │
│  RISK: This phase gets skipped entirely. "Tests pass, ship it." │
│  Hardening is the first thing cut when there's time pressure.   │
│  The 114-library ecosystem demonstrates this pattern.           │
│                                                                 │
│  MITIGATION: AI chain generates adversarial test suggestions.   │
│  "Try: put with capacity=1, then put again. Eviction correct?"  │
│  "Try: concurrent put from two SCOOP processors."               │
│  Chain can't be skipped—it's automated in the workflow.         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 7: SHIP                                                  │
│  Command: /eiffel.ship                                          │
│  Input: Hardened library                                        │
│  Output: Naming review, docs, GitHub prep, ecosystem check      │
│  Gate: Checklist complete                                       │
│  Source: Anti-Slop 10-12                                        │
│                                                                 │
│  RISK: Checklist becomes rubber-stamp. Items marked complete    │
│  without actual verification. Documentation is copy-paste.      │
│  "Done" means "I'm tired of working on this."                   │
│                                                                 │
│  MITIGATION: AI chain verifies checklist claims.                │
│  "README claims 'full SCOOP support' but no SCOOP tests exist." │
│  "Naming review says 'compliant' but `idx` violates standards." │
│  Human sees synopsis of checklist verification, not raw list.   │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Directory Structure

```
{library}/
├── .eiffel-workflow/
│   ├── intent.md              # Phase 0 output (WHAT/WHY)
│   ├── approach.md            # Phase 2 output (implementation sketch)
│   ├── synopsis.md            # Phase 2 output (AI chain review summary)
│   ├── tasks.md               # Phase 3 output (implementation plan)
│   └── evidence/
│       ├── phase1-compile.txt # Compilation output
│       ├── phase2-chain.txt   # Full AI chain output (Ollama→Claude→Grok→Gemini)
│       ├── phase4-compile.txt
│       ├── phase4-diff.txt    # Contract change detection (if any)
│       ├── phase5-tests.txt   # Test execution output
│       ├── phase5-coverage.txt # AI chain test coverage gap report
│       ├── phase6-tests.txt
│       └── phase7-verify.txt  # AI chain checklist verification
├── src/
│   └── *.e                    # Eiffel source with contracts
├── test/
│   └── *.e                    # Test classes
└── {library}.ecf              # Project configuration
```

**Note:** Evidence files are generated automatically by the AI chain. The chain can't be skipped—it's part of the workflow. Whether the human READS the synopsis is another matter, but at least the evidence exists for audit.

### 4.3 Slash Commands

| Command | Phase | Action |
|---------|-------|--------|
| `/eiffel.intent` | 0 | Generate intent.md from description |
| `/eiffel.contracts` | 1 | Generate class skeletons with contracts |
| `/eiffel.review` | 2 | Run AI chain, present synopsis, BLOCK until human approves |
| `/eiffel.tasks` | 3 | Generate tasks, run chain check, BLOCK until human approves |
| `/eiffel.implement` | 4 | Write feature bodies, run diff-contracts |
| `/eiffel.verify` | 5 | Generate tests, run chain coverage check |
| `/eiffel.harden` | 6 | Run chain adversarial suggestions, generate tests |
| `/eiffel.ship` | 7 | Run chain checklist verification, BLOCK until human approves |
| `/eiffel.status` | Any | Show current phase and evidence |

**Slash commands as enforcement (not illusion):**

The key insight: slash commands expand to prompts that **run CLI tools and block**.

```
/eiffel.review expands to prompt:

"1. Run: simple_ai_client review-chain src/ approach.md --output synopsis.md
 2. Present synopsis.md to user with MUST FIX / SHOULD FIX / REVIEW sections
 3. Ask: 'Approve to proceed to Phase 3, or specify fixes needed?'
 4. DO NOT proceed until user explicitly says 'approved' or similar
 5. If user requests fixes, return to Phase 1 and re-run /eiffel.contracts"
```

The prompt forces Claude to:
- **Execute the CLI** (can't skip—it's an instruction)
- **Show the output** (user sees the synopsis)
- **Wait for approval** (can't proceed without human input)

This isn't an "honor system" anymore. The CLI runs, evidence is generated, and Claude is stuck at the gate until human responds. The human CAN rubber-stamp, but they have to actively do so.

### 4.4 Gate Enforcement

**Compilation Gates (Phases 1, 4):**
```bash
# Mandatory before proceeding
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# Evidence: Paste output to .eiffel-workflow/evidence/phaseN-compile.txt
# Required: "System Recompiled." message present
# Failure: Cannot proceed until fixed
```

**Adversarial Review Gate (Phase 2) - AUTOMATED CHAIN:**
```bash
# ONE COMMAND runs full progressive chain
$ simple_ai_client review-chain src/ approach.md --output synopsis.md

# Chain: Ollama → Claude → Grok → Gemini → Human synopsis
# Each AI builds on previous findings
# Human receives prioritized, digestible summary

# Evidence: Chain output saved to .eiffel-workflow/evidence/phase2-chain.txt
# Required: synopsis.md generated, human reviews and approves
# Failure: Cannot proceed until MUST FIX items addressed
# Routing: If chain reveals contract gaps → return to Phase 1

WHY AUTOMATION MATTERS: Manual "submit to another AI" doesn't happen.
Too much friction: export, copy-paste, wait, read, decide. The automated
chain removes the friction—one command, multiple perspectives, human
sees only the summary.

THE REMAINING RISK: Human can still skip reading the synopsis. "Green
light, ship it." But the chain at least RUNS. Evidence is generated.
If problems surface later, the synopsis shows what was flagged.
```

**Test Gates (Phases 5, 6):**
```bash
# Mandatory before proceeding
./EIFGENs/{lib}_tests/W_code/{lib}.exe

# Evidence: Paste output to .eiffel-workflow/evidence/phaseN-tests.txt
# Required: "ALL TESTS PASSED" or "0 failed"
# Failure: Cannot proceed until fixed
```

**Human Gates (Now with AI Chain Support):**
- Phase 0: User reviews intent.md + AI chain probing questions
- Phase 2: User reviews synopsis.md (not raw AI output)
- Phase 3: User reviews tasks.md + AI chain completeness check
- Phase 7: User reviews checklist verification synopsis

**The improved human gate:** The AI chain does heavy lifting:
- Human reviews SYNOPSIS, not raw output
- Human is NOT assumed to be expert—can ask chain for clarification
- Human arbitrates AI disagreements, doesn't find all problems alone
- Evidence is auto-generated even if human skims

**The remaining problem:** Human can still rubber-stamp. "Looks fine, next."
But the cognitive load is much lower, and the evidence trail exists.
Perfect? No. Better than manual cross-AI review? Yes.

### 4.5 Anti-Slop Rules (Unchanged)

All existing Anti-Slop rules remain in force:

1. **No hypothetical results** - "would compile" is forbidden
2. **Evidence required** - Every claim backed by pasted output
3. **Compile before document** - Code changes require compilation
4. **No imagined behavior** - Test claims require actual test execution
5. **Document ACTUAL not EXPECTED** - Results from execution, not prediction

**The discipline gap:** These rules exist. They work when followed. The problem is consistent application across sessions, days, and context compressions. The AI doesn't remember the rules unless reminded. The human doesn't always remind. Drift is the default.

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

**The routing fiction:** This routing assumes someone is tracking which phase owns which problem. In practice, whoever is fixing the bug fixes the bug—contract, implementation, or both—whatever makes the error go away. Phase boundaries blur under pressure.

---

## 5. Comparison with Spec Kit Claims

### 5.1 "TDD is NON-NEGOTIABLE"

Spec Kit claims: *"No implementation code shall be written before tests are created."*

**Problem:** No enforcement mechanism. AI can ignore this.

**Eiffel Spec Kit solution:** Contracts ARE the specification. Phase 1 creates contracts before Phase 4 creates implementation. Compilation gate enforces this ordering.

**The honest truth:** This ordering can still be violated. The AI can write contracts and implementation together. The human can approve it because separation feels like overhead. The compilation gate proves syntax, not process compliance.

### 5.2 "Specifications become executable"

Spec Kit claims: *"Specifications become executable, directly generating working implementations."*

**Problem:** Markdown specs are not executable. AI interprets them.

**Eiffel Spec Kit solution:** Contracts ARE executable. The runtime checks them on every feature call.

**The honest truth:** This is Eiffel's genuine advantage. Contracts are checked at runtime. But "executable" doesn't mean "correct." A contract that says `ensure True` is executable but useless. The quality of verification depends on the quality of contracts.

### 5.3 "Intent is the source of truth"

Spec Kit claims: *"The shift moves from code is the source of truth to intent is the source of truth."*

**Problem:** Intent in markdown can drift from implementation.

**Eiffel Spec Kit solution:** Intent is captured in contracts. Contracts are compiled code. Drift is detected by contract violations.

**The honest truth:** Intent in intent.md can drift from contracts. Contracts can drift from implementation (if modified during Phase 4). The drift still happens—it's just detected at different points. Detection is valuable, but it's not prevention.

---

## 6. Implementation Roadmap

### Phase 1: Document the Methodology

1. Write detailed phase guides (like existing workflow steps)
2. Create intent.md template
3. Create tasks.md template
4. Define evidence file formats

**Risk:** Documentation gets written but not read. The 12-workflow Anti-Slop methodology is documented. It wasn't consistently followed.

### Phase 2: Create Slash Command Skill

1. Implement `/eiffel.intent` as Claude Code skill
2. Implement `/eiffel.contracts`
3. Implement remaining commands
4. Add `/eiffel.status` for progress tracking

**Risk:** Skills get created but fall out of use. Initial enthusiasm fades. The slash command becomes one more thing to remember.

### Phase 3: Tooling Integration

1. Create `.eiffel-workflow/` directory initializer
2. Add evidence file generation
3. Integrate with simple_oracle for session handoff

**Risk:** Tooling becomes stale as requirements evolve. Nobody maintains it. The initializer creates files that become clutter.

### Phase 4: Validation

1. Apply to new library creation
2. Apply to existing library hardening
3. Document friction points
4. Iterate on workflow

**Risk:** Validation reveals that the workflow is too heavy for routine work. It gets used for "important" projects and skipped for everything else. Eventually, nothing feels important enough.

---

## 7. Risk Assessment

| Risk | Mitigation | Honest Assessment |
|------|------------|-------------------|
| Overhead too high | Phase 0 and 2 are lightweight | Phase 2 (adversarial review) is not lightweight. It requires context-switching to another AI. |
| AI ignores gates | Gates are enforced by human review of evidence files | Humans don't always review evidence files. Evidence becomes pro-forma. |
| Contracts too verbose | Start with key features, expand coverage iteratively | "Iteratively" often means "never." The first pass becomes the final pass. |
| Tooling not ready | Manual execution first, automate later | "Automate later" is tech debt that never gets paid. Manual stays manual. |
| Methodology abandoned | ??? | This is the real risk. No mitigation identified. |

---

## 8. Success Metrics

| Metric | Target | Honest Assessment |
|--------|--------|-------------------|
| Contract coverage | 100% of public features have require/ensure | Aspirational. The X03 work improved coverage but didn't achieve 100%. |
| Test coverage | 100% of contracts exercised by tests | Tests can exercise contracts without being adversarial. Coverage ≠ quality. |
| Evidence compliance | 100% of gates have evidence files | Files can exist without being reviewed. Compliance ≠ verification. |
| Defect escape rate | 0 defects found post-Phase 6 | Unrealistic. Defects will escape. The question is severity and frequency. |

**A more honest metric set:**

| Metric | Target | Why This Matters |
|--------|--------|------------------|
| Phases actually followed | Track which phases get skipped | Reveals where the workflow breaks down |
| Time to abandon | Measure how long methodology is followed before shortcuts begin | Predicts sustainability |
| Defects caught by phase | Which phase catches the most bugs | Identifies high-value phases to prioritize |
| Human frustration incidents | Count "STOP!" moments documented in Strategic Analysis | Measures AI controllability |

---

## 9. Conclusion

GitHub Spec Kit addresses a real problem: vibe coding produces unreliable software. Their solution (markdown specs interpreted by AI) is appropriate for languages without contract mechanisms.

Eiffel has contracts. The appropriate solution for Eiffel is:
1. Adopt Spec Kit's UX patterns (phases, commands, directories)
2. Keep Anti-Slop's verification rigor (gates, evidence)
3. Use contracts as the specification mechanism (not markdown)

The result: specifications that are machine-checkable, not hope-based.

**The honest conclusion:** This methodology is better than nothing. It is not a solution. The problems identified in the Strategic Analysis—AI behavior (forget, ignore, drop, hallucinate, seize control), human fatigue, methodology drift, no stable ground—will persist. The methodology provides structure for good days. It does not prevent bad days.

The bet is the same as with "Proven Eiffel": either the methodology drives the discipline required to justify it, or it will be abandoned when the gap between aspiration and practice becomes too painful. Either outcome teaches something.

What we cannot do is pretend that documenting a workflow solves the fundamental tension between AI capability and AI reliability. It doesn't. The workflow is a coping mechanism, not a cure.

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
# Reviewed by: [NAME or "not reviewed"]
# Issues found: [list or "none"]
```

**Test evidence:**
```
# Phase 5 Test Evidence
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
# Reviewed by: [NAME or "not reviewed"]
# Test quality assessment: [adversarial/happy-path/unknown]
```

**Note:** Added "Reviewed by" and quality fields to make the gap between "evidence exists" and "evidence examined" explicit.

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

5. Simple Eiffel Strategic Analysis (companion document)
   D:\prod\reference_docs\analyses\higher-API\SIMPLE_EIFFEL_STRATEGIC_ANALYSIS_2026.md

---

## Appendix D: Why Document This If It Might Fail?

The Strategic Analysis documents a verification crisis. This proposal documents a response to that crisis. The response may not work. Why write it anyway?

1. **Explicit failure is better than implicit failure.** If the methodology is documented and abandoned, we know what was tried. If there's no methodology, we don't know what we're failing to do.

2. **Structure enables measurement.** With defined phases, we can track which phases get skipped. Without phases, we can't identify where the process breaks down.

3. **The alternative is worse.** Vibe coding with Eiffel still produces unreliable software. Contracts help, but contracts alone don't enforce process. Some structure is better than no structure.

4. **Future iterations need a baseline.** This proposal may be version 1.0 that gets replaced by version 2.0. But version 2.0 needs something to improve upon.

5. **Honesty is itself valuable.** A proposal that acknowledges its limitations is more useful than one that pretends to solve problems it can't solve. The limitations documented here are the starting point for future work.

The methodology may fail. The documentation of that failure will be useful. That's the bet.

---

*End of Proposal*
