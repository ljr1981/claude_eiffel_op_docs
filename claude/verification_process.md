# Verification Process: From Probable to Provable

**Purpose:** Document the iterative process for AI-assisted development with formal verification, based on Bertrand Meyer's framework from "AI for software engineering: from probable to provable" (CACM, 2025).

---

## The Core Problem

Meyer identifies the fundamental issue with AI-generated code:

> "Unlike Traditional-AI, even very good Modern-AI can only produce a probably correct answer; if we multiply probabilities, we cannot come close to guaranteeing the correctness of a realistic system."

The **Modular Probability Problem** (Dijkstra, 1970):
- N modules, each with probability p of being correct
- System correctness = p^N (assuming independence)
- Even p = 99.9% with 1000 modules → 37% system correctness
- With 5000 modules → less than 1%

**Implication:** We cannot rely on AI producing "mostly correct" code. We need verification.

---

## The Solution: Vibe-Contracting + Vibe-Coding

Meyer's prescription:

> "Use the most advanced mechanisms of generative AI, combined with human insight and common sense (the need for which will never go away) to develop specification and implementation hand in hand. At each step, check their consistency with proof tools."

This is **exactly** our approach with Eiffel's Design by Contract.

---

## The Iterative Verification Process

### Meyer's Description:

> "Specify a little, implement a little (in either order); attempt to verify; hit a property that does not verify; attempt to correct the specification, the implementation or both; repeat."

### Our Practical Workflow:

```
┌─────────────────────────────────────────────────────────────┐
│  1. SPECIFICATION PHASE (Specification Hat)                 │
│     - AI proposes contracts (preconditions, postconditions) │
│     - Human reviews for completeness                        │
│     - Focus: WHAT, not HOW                                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  2. IMPLEMENTATION PHASE (Feature Hat)                      │
│     - AI proposes implementation to satisfy contracts       │
│     - Human reviews for correctness                         │
│     - Focus: HOW to achieve WHAT                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  3. STATIC VERIFICATION                                     │
│     - Compiler checks syntax and types                      │
│     - [Future] AutoProof checks contract satisfaction       │
│     - Catches errors WITHOUT execution                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  4. DYNAMIC VERIFICATION                                    │
│     - Runtime contract checking (current approach)          │
│     - Tests exercise code paths                             │
│     - Catches errors DURING execution                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  5. ITERATION                                               │
│     - Contract violation? Fix implementation OR contract    │
│     - Test failure? Fix code OR test OR specification       │
│     - Repeat until verified                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Verification Levels

### Level 1: Compilation (Always)
- Type safety
- Void safety
- Syntax correctness

### Level 2: Runtime Contracts (Current Practice)
- Preconditions checked on entry
- Postconditions checked on exit
- Class invariants checked after creation and public calls
- Loop variants checked for termination

### Level 3: Static Proofs (Future Goal)
- AutoProof (https://autoproof.org) for Eiffel
- Proves contracts hold WITHOUT running code
- Catches errors that tests might miss
- Requires additional annotations (loop invariants especially)

### Level 4: Tests (Complementary)
- Exercise specific scenarios
- Verify behavior matches expectations
- Cannot prove absence of bugs, only presence

---

## The Contract Completeness Problem

Meyer warns:

> "The weakness — and it's real — is that I might get contracts that are *true* but *incomplete*. They pass but don't capture full correctness."

### Example of Incomplete Contract:

```eiffel
add_item (a_item: STRING)
    -- Add item to list.
  require
    item_not_void: a_item /= Void
  do
    items.extend (a_item)
  ensure
    item_added: items.has (a_item)  -- TRUE but INCOMPLETE
  end
```

**What's missing?**
- `count_increased: items.count = old items.count + 1`
- `others_unchanged: across 1 |..| old items.count as i all items[i] ~ (old items.twin)[i] end`

The contract `items.has (a_item)` is *true* after the call, but doesn't capture *everything* the feature guarantees.

### Defense Against Incompleteness:

1. **Systematic postcondition patterns** (see `contract_patterns.md`)
2. **Review contracts separately from code** (Specification Hat)
3. **Ask "what ELSE is guaranteed?"** after writing each postcondition
4. **Class invariants catch cross-feature properties**

---

## AI's Role in Verification

### What AI Does Well:
- Generate boilerplate contracts
- Apply known patterns consistently
- Produce initial specifications from natural language
- Suggest loop invariants (with review)

### What AI Does Poorly:
- Guarantee completeness of contracts
- Identify subtle invariants
- Know when a contract is "too weak"
- Reason about system-wide properties

### The Human Role (Never Goes Away):
- Architectural decisions
- Completeness review
- Domain knowledge
- Common sense checks
- Final accountability

---

## AutoProof Integration (Future)

AutoProof is the Eiffel static verifier mentioned by Meyer. Key points:

**What AutoProof Needs:**
- Loop invariants (we already require these)
- Loop variants (we already require these)
- Class invariants
- Intermediate assertions (`check` statements)
- Sometimes: ghost variables for specification state

**AutoProof-Compatible Contract Style:**
```eiffel
feature -- Example with full annotations

  binary_search (a_array: ARRAY [INTEGER]; a_target: INTEGER): INTEGER
      -- Index of `a_target` in sorted `a_array`, or -1 if not found.
    require
      array_sorted: across 1 |..| (a_array.count - 1) as i all
                      a_array[i] <= a_array[i + 1]
                    end
    local
      low, high, mid: INTEGER
    do
      from
        low := a_array.lower
        high := a_array.upper
        Result := -1
      invariant
        result_valid: Result = -1 or else a_array.valid_index (Result)
        target_in_range: Result = -1 implies
          (across a_array.lower |..| (low - 1) as i all a_array[i] /= a_target end) and
          (across (high + 1) |..| a_array.upper as i all a_array[i] /= a_target end)
      until
        low > high or Result /= -1
      loop
        mid := (low + high) // 2
        if a_array[mid] = a_target then
          Result := mid
        elseif a_array[mid] < a_target then
          low := mid + 1
        else
          high := mid - 1
        end
      variant
        high - low + 1
      end
    ensure
      found_correct: Result /= -1 implies a_array[Result] = a_target
      not_found_correct: Result = -1 implies
        across a_array as i all i /= a_target end
    end
```

---

## Practical Guidelines for Claude

### When Writing Contracts:

1. **Start with contracts, not code** (Specification Hat first)
2. **Ask: What must be true BEFORE?** → Preconditions
3. **Ask: What will be true AFTER?** → Postconditions
4. **Ask: What is ALWAYS true?** → Class invariants
5. **Ask: What ELSE is guaranteed?** → Completeness check

### When Writing Implementation:

1. **Implementation must SATISFY contracts, not define them**
2. **If implementation is hard, maybe contract is wrong**
3. **If contract seems too strict, discuss with human**
4. **Every loop needs invariant AND variant**

### When Verification Fails:

1. **Contract violation in test?**
   - Is the contract wrong? (specification bug)
   - Is the implementation wrong? (code bug)
   - Is the test wrong? (test bug)

2. **Don't just "fix the test to pass"** — understand WHY it failed

---

## References

- Meyer, Bertrand. "AI for software engineering: from probable to provable." Communications of the ACM, 2025.
- Meyer, Bertrand. "Handbook of Requirements and Business Analysis." Springer, 2022.
- AutoProof: https://autoproof.org
- Dijkstra, E.W. "Notes on Structured Programming." 1970.

---

## Update Log

| Date | Change |
|------|--------|
| 2025-12-03 | Initial creation based on Meyer's CACM 2025 paper |
