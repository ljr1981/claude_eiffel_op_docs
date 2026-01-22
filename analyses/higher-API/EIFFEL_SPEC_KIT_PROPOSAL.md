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

Step 2c: THREE-PASS REVIEW (layered AI + human)
         Pass 1 - Self-critique (Claude reviews own output):
                  "Wait, this precondition doesn't handle empty input..."
                  "I should add a frame condition for unchanged keys..."

         Pass 2 - Cross-AI critique (Grok/Gemini via simple_ai_client):
                  "Claude missed that the MML postcondition is O(n²)..."
                  "The sketch assumes sorted input but precondition doesn't require it"

         Pass 3 - Human reviews (final approval):
                  Sees: original + Claude's self-critique + Grok's critique
                  Much less cognitive load—AIs did the heavy lifting
                  Human focuses on: "Do I agree with the critiques? What's unresolved?"

Step 2d: Refine contracts and approach based on findings
         - Strengthen weak contracts
         - Add missing MML model queries
         - Adjust approach if algorithm issues found

Step 2e: Human approves refined output (or routes back to Phase 1)
```

**Why Three-Pass Review:**

| Pass | Reviewer | Catches | Blind Spots |
|------|----------|---------|-------------|
| 1 | Claude (self) | Obvious errors, forgotten edge cases | Same assumptions as original |
| 2 | Grok/Gemini | Different model's perspective, Claude's blind spots | May not know Eiffel well |
| 3 | Human | Domain knowledge, business requirements | Fatigue, expertise gaps |

**The key insight:** Human reviews *AI-reviewed* output, not *raw* output. The human's job shifts from "find all problems" to "arbitrate AI disagreements and catch what both missed." This is sustainable; reviewing everything alone is not.

**CLI Flow:**

```bash
# Step 2c automated via simple_ai_client
$ simple_ai_client review-contracts src/ --self-critique --cross-ai gemini

Pass 1: Claude self-critique...
  - precondition missing: empty input handling
  - frame condition missing: put should specify unchanged keys
  - model query needed: access_order for LRU tracking

Pass 2: Gemini cross-AI critique...
  - MML postcondition keys_model is O(n), sketch claims O(1)—mismatch
  - Edge case: capacity=0 not addressed in preconditions
  - SCOOP safety: what if put called during iteration?

Generating review.md...

Human review required: 6 issues identified, 0 resolved automatically.
```

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

**Adversarial Review of Sketch (Step 2c)**

The adversarial AI sees BOTH the contracts AND the approach:

```
Adversarial AI asks:
1. Your sketch says "O(1) average" but MML postcondition iterates
   all keys for keys_model—that's O(n). Is this acceptable for
   contract checking, or do you need a faster model?

2. Edge case "Capacity = 0" isn't addressed in preconditions.
   Should `require capacity > 0` be added to make?

3. The sketch mentions "doubly-linked list" but contracts don't
   specify LRU ordering preservation. Add postcondition?

4. What happens if put is called during iteration? SCOOP-safe?
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
│  hard. "I need to weaken this precondition to make it work."   │
│  Human allows it because fighting the AI is exhausting.         │
│  Contract-first becomes contract-when-convenient.               │
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
│  RISK: Tests are weak. Happy-path only. Tests pass because     │
│  they don't test the hard cases. "All tests pass" creates      │
│  false confidence.                                              │
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
│  RISK: This phase gets skipped entirely. "Tests pass, ship it."│
│  Hardening is the first thing cut when there's time pressure.  │
│  The 114-library ecosystem demonstrates this pattern.           │
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
│  RISK: Checklist becomes rubber-stamp. Items marked complete   │
│  without actual verification. Documentation is copy-paste.      │
│  "Done" means "I'm tired of working on this."                   │
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

**Note:** The evidence files are supposed to prove work was done. In practice, they often become pro-forma—pasted output that nobody reads. The existence of an evidence file does not guarantee the evidence was examined.

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

**The slash command illusion:** These commands make the workflow feel more structured. But a slash command is just a prompt. The AI can still forget, ignore, drop, hallucinate, and seize control after the command is invoked. The command doesn't enforce compliance—it just provides a starting point.

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

THE REALITY: Even with a mandated phase, the human may skip it when
pressed for time. "I'll do the adversarial review on the next library."
Mandates only work when someone is checking compliance. There is no
one checking.
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

**The human gate problem:** These gates depend on a human who is:
- Paying attention (not fatigued from hours of AI interaction)
- Willing to push back (not eager to see progress)
- Knowledgeable enough to spot problems (not dependent on AI expertise)

All three assumptions are frequently false.

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
