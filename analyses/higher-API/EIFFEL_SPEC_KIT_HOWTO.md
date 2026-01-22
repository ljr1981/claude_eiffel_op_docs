# Eiffel Spec Kit: How-To Guide

**Purpose:** Step-by-step guide for applying the Eiffel Spec Kit workflow to implement contract-driven Eiffel libraries.

**Context:** This guide uses `proven_fetch` (from the Strategic Analysis) as the running example, but the workflow applies to any library.

---

## Prerequisites

- Claude Code with eiffel-* skills installed (`~/.claude/skills/eiffel-*/`)
- Target project directory (e.g., `d:\prod\proven_fetch`)
- Access to external AIs for review cycle (Ollama, Grok, Gemini)

---

## Quick Reference

| Phase | Command | Blocks Until |
|-------|---------|--------------|
| 0 | `/eiffel.intent <path>` | Human approves intent-v2.md |
| 1 | `/eiffel.contracts <path>` | Compilation succeeds |
| 2 | `/eiffel.review <path>` | Human approves synopsis |
| 3 | `/eiffel.tasks <path>` | Human approves task breakdown |
| 4 | `/eiffel.implement <path>` | Compilation succeeds, contracts unchanged |
| 5 | `/eiffel.verify <path>` | All tests pass |
| 6 | `/eiffel.harden <path>` | All adversarial tests pass |
| 7 | `/eiffel.ship <path>` | Human approves for release |
| - | `/eiffel.status <path>` | (status check, no blocking) |

---

## Phase 0: Intent Capture

### Command
```
/eiffel.intent d:\prod\proven_fetch
```

### What You Provide
Describe what you want to build. For `proven_fetch`:

> "I want a fluent API for consuming external APIs with contract-verified responses.
> Instead of manually using simple_http + simple_json + validation, developers write:
> `fetch.from(url).expecting(Contract).verified`
> The result either satisfies the contract or documents why it failed."

### What Gets Created

```
d:\prod\proven_fetch/
└── .eiffel-workflow/
    ├── intent.md                           # Initial intent document
    ├── prompts/
    │   └── phase0-intent-review.md         # Prompt for external AI
    └── evidence/
```

### Manual Action Required

1. Open `prompts/phase0-intent-review.md`
2. Copy your `intent.md` content into the prompt where indicated
3. Submit to an external AI (Ollama, Grok, or Gemini)
4. Save the AI's response to `evidence/phase0-ai-review.md`
5. Return and say "review complete"

### What Happens Next

- Claude processes the AI's probing questions
- You answer each question
- Claude generates `intent-v2.md` incorporating your answers
- **You must explicitly approve** to proceed

### Output
```
Phase 0 COMPLETE: Intent captured and approved.
Next: Run /eiffel.contracts d:\prod\proven_fetch
```

---

## Phase 1: Contracts + Skeletal Tests

### Command
```
/eiffel.contracts d:\prod\proven_fetch
```

### What Gets Created

```
d:\prod\proven_fetch/
├── .eiffel-workflow/
│   └── evidence/
│       └── phase1-compile.txt              # Compilation evidence
├── src/
│   ├── proven_fetch.e                      # Contract skeleton
│   ├── fetch_contract.e                    # Supporting classes
│   └── fetch_result.e
├── test/
│   └── test_proven_fetch.e                 # Skeletal tests
└── proven_fetch.ecf                        # Project config
```

### Contract Skeleton Example

```eiffel
class PROVEN_FETCH

feature -- Fluent API

    from_url (a_url: READABLE_STRING_GENERAL): like Current
            -- Configure fetch from `a_url`.
        require
            url_not_empty: not a_url.is_empty
            url_valid: is_valid_url (a_url)
        do
            -- Implementation in Phase 4
        ensure
            url_set: url_model ~ a_url.to_string_32
            other_state_unchanged: response_contract = old response_contract
        end

    expecting (a_contract: FETCH_CONTRACT): like Current
            -- Specify expected response shape.
        require
            contract_attached: a_contract /= Void
        do
            -- Implementation in Phase 4
        ensure
            contract_set: response_contract = a_contract
            url_unchanged: url_model ~ old url_model
        end

    verified: FETCH_RESULT
            -- Execute fetch and verify against contract.
        require
            configured: is_configured
        do
            -- Implementation in Phase 4
        ensure
            success_means_valid: Result.is_success implies Result.satisfies (response_contract)
            failure_documented: not Result.is_success implies attached Result.failure_reason
        end

invariant
    url_implies_model: attached url implies url_model.count > 0

end
```

### Contract Completeness Checklist

Every postcondition must answer:
1. **What changed?** (direct effect)
2. **How did it change?** (relationship to `old` state)
3. **What did NOT change?** (frame conditions)

### Mandatory Gate

```bash
/d/prod/ec.sh -batch -config proven_fetch.ecf -target proven_fetch_tests -c_compile
```

**Must see "System Recompiled."** If compilation fails, fix contracts before proceeding.

### Output
```
Phase 1 COMPLETE: Contracts and skeletal tests created.
Next: Run /eiffel.review d:\prod\proven_fetch
```

---

## Phase 2: Adversarial Review Chain

### Command
```
/eiffel.review d:\prod\proven_fetch
```

### What Gets Created

```
d:\prod\proven_fetch/
└── .eiffel-workflow/
    ├── approach.md                         # Implementation sketch
    ├── prompts/
    │   ├── phase2-ollama-review.md         # Weakest AI first
    │   ├── phase2-claude-review.md         # Builds on Ollama
    │   ├── phase2-grok-review.md           # Builds on Claude
    │   └── phase2-gemini-review.md         # Final synthesis
    └── evidence/
        ├── phase2-ollama-response.md       # You save responses here
        ├── phase2-claude-response.md
        ├── phase2-grok-response.md
        ├── phase2-gemini-response.md
        └── phase2-chain.txt                # Summary evidence
```

### Manual Review Cycle

**Step 1: Ollama (local, free)**
1. Open `prompts/phase2-ollama-review.md`
2. Paste contract files where indicated
3. Submit to Ollama
4. Save response to `evidence/phase2-ollama-response.md`

**Step 2: Claude (different session)**
1. Open `prompts/phase2-claude-review.md`
2. Paste Ollama's response + contracts
3. Submit to a different Claude session
4. Save response to `evidence/phase2-claude-response.md`

**Step 3: Grok**
1. Open `prompts/phase2-grok-review.md`
2. Paste previous responses + contracts
3. Submit to Grok
4. Save response to `evidence/phase2-grok-response.md`

**Step 4: Gemini (final AI)**
1. Open `prompts/phase2-gemini-review.md`
2. Paste all previous responses + contracts
3. Submit to Gemini
4. Save response to `evidence/phase2-gemini-response.md`

**Minimum requirement:** Use at least ONE external AI. The point is multiple perspectives, not bureaucracy.

### What Happens Next

Say "reviews complete" and Claude generates `synopsis.md`:

```markdown
# Review Synopsis

## MUST FIX (blocking)
1. Missing frame condition on `verified` - doesn't specify url_model unchanged
2. No precondition for empty contract in `expecting`

## SHOULD FIX (recommended)
3. Consider timeout contract for `verified`

## Human Approval
- [ ] I have reviewed this synopsis
- [ ] MUST FIX items will be addressed before Phase 3
```

**You must explicitly approve** to proceed. If fixes needed, return to Phase 1.

### Output
```
Phase 2 COMPLETE: Adversarial review passed.
Next: Run /eiffel.tasks d:\prod\proven_fetch
```

---

## Phase 3: Task Decomposition

### Command
```
/eiffel.tasks d:\prod\proven_fetch
```

### What Gets Created

```
d:\prod\proven_fetch/
└── .eiffel-workflow/
    ├── tasks.md                            # Implementation tasks
    └── evidence/
        └── phase3-tasks.txt
```

### Example tasks.md

```markdown
# Implementation Tasks: proven_fetch

## Task 1: URL Configuration
**Files:** src/proven_fetch.e
**Features:** from_url, is_valid_url, url_model

### Acceptance Criteria
- [ ] Preconditions satisfied
- [ ] Postconditions verified by contracts
- [ ] Compiles clean

### Implementation Notes
- Use simple_http URL validation
- Store URL in internal STRING_32

### Dependencies
None

---

## Task 2: Contract Attachment
**Files:** src/proven_fetch.e, src/fetch_contract.e
**Features:** expecting, response_contract

### Acceptance Criteria
- [ ] Contract object properly attached
- [ ] Frame conditions verified

### Dependencies
None

---

## Task 3: Execution + Verification
**Files:** src/proven_fetch.e, src/fetch_result.e
**Features:** verified, execute_fetch, validate_response

### Acceptance Criteria
- [ ] HTTP request executes
- [ ] Response parsed as JSON
- [ ] Contract validation runs
- [ ] Result documents success or failure reason

### Dependencies
Task 1, Task 2
```

**You must explicitly approve** the task breakdown to proceed.

### Output
```
Phase 3 COMPLETE: Tasks defined.
Next: Run /eiffel.implement d:\prod\proven_fetch
```

---

## Phase 4: Implementation

### Command
```
/eiffel.implement d:\prod\proven_fetch
```

### Critical Rule: CONTRACTS ARE FROZEN

**DO NOT modify require/ensure/invariant clauses.**

If implementation seems impossible without changing contracts:
1. STOP
2. Document why
3. Return to Phase 1 (`/eiffel.contracts`)
4. Re-run Phase 2 review after changes

### What Happens

1. **Snapshot contracts**
   ```bash
   grep -n "require\|ensure\|invariant" src/*.e > evidence/phase4-contracts-before.txt
   ```

2. **Implement each task** - fill in `do` blocks between contracts

3. **Compile after each task**
   ```bash
   /d/prod/ec.sh -batch -config proven_fetch.ecf -target proven_fetch_tests -c_compile
   ```

4. **Detect contract changes**
   ```bash
   grep -n "require\|ensure\|invariant" src/*.e > evidence/phase4-contracts-after.txt
   diff evidence/phase4-contracts-before.txt evidence/phase4-contracts-after.txt
   ```

   **If contracts changed: STOP and notify human.**

### Output
```
Phase 4 COMPLETE: Implementation done.
- All features implemented
- Contracts unchanged
- Compilation: PASS

Next: Run /eiffel.verify d:\prod\proven_fetch
```

---

## Phase 5: Test Generation + Verification

### Command
```
/eiffel.verify d:\prod\proven_fetch
```

### What Happens

1. **Flesh out skeletal tests** from Phase 1

2. **Generate contract-derived tests** - one test per postcondition:
   ```eiffel
   test_from_url_sets_url_model
       -- Verify: url_set postcondition

   test_expecting_preserves_url
       -- Verify: url_unchanged frame condition

   test_verified_success_satisfies_contract
       -- Verify: success_means_valid postcondition

   test_verified_failure_has_reason
       -- Verify: failure_documented postcondition
   ```

3. **Generate coverage review prompt** for external AI

4. **Run all tests** - mandatory gate
   ```bash
   ./EIFGENs/proven_fetch_tests/W_code/proven_fetch.exe
   ```

**ALL TESTS MUST PASS.**

### Output
```
Phase 5 COMPLETE: Verification passed.
Tests: 12 passed

Next: Run /eiffel.harden d:\prod\proven_fetch
```

---

## Phase 6: Adversarial Testing

### Command
```
/eiffel.harden d:\prod\proven_fetch
```

### What Gets Created

Additional tests targeting:

| Category | Example Tests |
|----------|---------------|
| Boundary values | Empty URL, max-length URL |
| Invalid inputs | Malformed URLs, null contracts |
| Network failures | Timeout, connection refused, DNS failure |
| Response problems | Invalid JSON, wrong shape, empty body |
| Contract violations | Server returns unexpected structure |
| Concurrency | Multiple concurrent fetches (SCOOP) |
| Resource exhaustion | Very large responses |

### Example Adversarial Tests

```eiffel
test_empty_url_rejected
    -- Precondition must catch empty URL.
    local
        l_fetch: PROVEN_FETCH
    do
        create l_fetch
        l_fetch.from_url ("")  -- Should trigger precondition violation
    rescue
        assert ("precondition caught empty", True)
    end

test_timeout_documented
    -- Network timeout produces failure with reason.
    local
        l_fetch: PROVEN_FETCH
        l_result: FETCH_RESULT
    do
        create l_fetch
        l_result := l_fetch.from_url ("http://10.255.255.1/")  -- Non-routable
                          .expecting (any_contract)
                          .with_timeout (100)  -- 100ms
                          .verified
        assert ("not success", not l_result.is_success)
        assert ("has reason", attached l_result.failure_reason)
    end

test_wrong_shape_rejected
    -- Contract validation catches wrong response shape.
```

**ALL TESTS MUST PASS.**

### Output
```
Phase 6 COMPLETE: Hardening passed.
All tests: PASS

Next: Run /eiffel.ship d:\prod\proven_fetch
```

---

## Phase 7: Ship Checklist

### Command
```
/eiffel.ship d:\prod\proven_fetch
```

### Checklist Items

**Naming Review:**
- [ ] Class names: UPPER_SNAKE_CASE
- [ ] Feature names: lower_snake_case
- [ ] Locals: l_prefix
- [ ] Arguments: a_prefix
- [ ] No abbreviations

**Documentation:**
- [ ] README.md exists
- [ ] CHANGELOG.md exists
- [ ] All public features have header comments

**Ecosystem Integration:**
- [ ] ECF references correct simple_* dependencies
- [ ] No ISE stdlib where simple_* exists
- [ ] SCOOP compatible (concurrency=scoop)
- [ ] Void safety enabled (void_safety=all)

**Final Verification:**
```bash
/d/prod/ec.sh -batch -config proven_fetch.ecf -target proven_fetch_tests -c_compile
./EIFGENs/proven_fetch_tests/W_code/proven_fetch.exe
```

### Human Approval

**"Ship checklist complete. Ready to release?"**

**You must explicitly approve** for the library to be marked complete.

### Output
```
Phase 7 COMPLETE: Ready to ship.

All phases complete:
✓ Phase 0: Intent
✓ Phase 1: Contracts
✓ Phase 2: Review
✓ Phase 3: Tasks
✓ Phase 4: Implement
✓ Phase 5: Verify
✓ Phase 6: Harden
✓ Phase 7: Ship

Library is ready for release.
```

---

## Status Check

At any point, check progress:

```
/eiffel.status d:\prod\proven_fetch
```

### Example Output

```
╔══════════════════════════════════════════════════════════════╗
║           EIFFEL SPEC KIT STATUS                             ║
║           Project: d:\prod\proven_fetch                      ║
╠══════════════════════════════════════════════════════════════╣
║ Phase 0: Intent      [✓] intent-v2.md approved               ║
║ Phase 1: Contracts   [✓] Compiled                            ║
║ Phase 2: Review      [✓] Synopsis approved                   ║
║ Phase 3: Tasks       [✓] 5 tasks defined                     ║
║ Phase 4: Implement   [→] IN PROGRESS                         ║
║ Phase 5: Verify      [ ] Pending                             ║
║ Phase 6: Harden      [ ] Pending                             ║
║ Phase 7: Ship        [ ] Pending                             ║
╠══════════════════════════════════════════════════════════════╣
║ Current: Phase 4 - Implementation                            ║
║ Next: Complete implementation, then /eiffel.verify           ║
╚══════════════════════════════════════════════════════════════╝
```

### Drift Warnings

```
DRIFT WARNINGS:
- [!] Contracts changed since review (re-run /eiffel.review)
- [!] phase4-compile.txt missing (compile not verified)
```

---

## Applying to Strategic Analysis Libraries

To implement all 5 domain libraries from the Strategic Analysis:

### Initialize All Projects

```
/eiffel.intent d:\prod\proven_fetch
/eiffel.intent d:\prod\proven_serve
/eiffel.intent d:\prod\proven_transform
/eiffel.intent d:\prod\proven_automate
/eiffel.intent d:\prod\proven_ask
```

### Recommended Order

1. **proven_fetch** - Simplest, fewest dependencies
2. **proven_ask** - Similar pattern to fetch (request/response)
3. **proven_serve** - Inverse of fetch (expose vs consume)
4. **proven_transform** - Data pipeline patterns
5. **proven_automate** - Event-driven, most complex

### Cross-Project Contracts

When libraries depend on each other:

```eiffel
-- proven_transform using proven_fetch result
transform.from (fetch_result.data)
         .to (json_target)
         .ensuring (Shape_preserved)
```

Document these in `approach.md` during Phase 2.

---

## Enforcement Summary

| Strategic Doc Concern | Spec Kit Enforcement |
|----------------------|---------------------|
| "AI forgets" | Evidence files persist; oracle restores context |
| "AI ignores" | Mandatory compilation gates |
| "AI drops steps" | Phase sequence enforced; can't skip |
| "AI hallucinates" | Manual AI review cycle with external AIs |
| "AI seizes control" | Human approval blocks at each phase |
| "Verification bottleneck" | Contracts + adversarial testing baked in |
| "Judgment doesn't scale" | Multiple AI reviewers + human synthesis |
| "Architectural drift" | Contracts frozen after Phase 2 |

---

## Related Documents

- [EIFFEL_SPEC_KIT_PROPOSAL.md](EIFFEL_SPEC_KIT_PROPOSAL.md) - Full proposal with rationale
- [SIMPLE_EIFFEL_STRATEGIC_ANALYSIS_2026.md](SIMPLE_EIFFEL_STRATEGIC_ANALYSIS_2026.md) - Market analysis and domain definitions

---

*End of How-To Guide*
