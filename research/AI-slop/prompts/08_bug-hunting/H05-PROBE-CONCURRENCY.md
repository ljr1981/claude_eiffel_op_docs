# H05: Probe Concurrency

## Context
Phase: **PROBE** (2 of 3)
Scenario: Bug hunting, SCOOP/threading issues

## Input Required
- Source code
- ECF configuration (concurrency setting)
- Problem domain (concurrent use cases)

## Prompt

```
TASK: Identify concurrency bugs including domain-specific concurrent scenarios.

INPUT:
{eiffel_source_code}
{ecf_config}
{problem_domain}

FIRST: Check concurrency mode in ECF
- concurrency="scoop" → Full SCOOP analysis
- concurrency="thread" → Thread analysis
- concurrency="none" → Limited analysis (may still have async issues)

═══════════════════════════════════════════════════════════════
SCOOP TECHNICAL PATTERNS
═══════════════════════════════════════════════════════════════

1. MISSING SEPARATE
   - Object crosses processor boundary without separate

2. SEPARATE ARGUMENT TRAITOR
   - Separate stored in non-separate attribute

3. LOCK ORDERING / DEADLOCK
   - Inconsistent multiple separate argument order

4. WAIT CONDITION ISSUES
   - Precondition can't be evaluated on separate

═══════════════════════════════════════════════════════════════
DOMAIN CONCURRENCY SCENARIOS
What concurrent access does the domain require/forbid?
═══════════════════════════════════════════════════════════════

1. IDENTIFY DOMAIN CONCURRENT USE CASES
   - Multiple users accessing same resource?
   - Background processing while user interacts?
   - Timed events concurrent with user actions?

2. IDENTIFY DOMAIN CONSISTENCY REQUIREMENTS
   - What invariants must hold across concurrent access?
   - What operations must be atomic?
   - What sequences must be serialized?

3. PROBE DOMAIN RACE CONDITIONS
   - Two operations that could interleave harmfully
   - Read-modify-write without atomicity
   - Check-then-act with stale check

═══════════════════════════════════════════════════════════════
CONTRACT CONCURRENCY ALIGNMENT
Do contracts account for concurrent access?
═══════════════════════════════════════════════════════════════

MALFORMED CONTRACT (Concurrency):
  - Precondition assumes exclusive access but doesn't have it
  - Postcondition references state that could change
  - Invariant assumes atomic operations that aren't

MALFORMED CODE (Concurrency):
  - Feature modifies shared state without synchronization
  - Feature reads state that could be mid-modification
  - Feature assumes sequence that concurrency breaks

OUTPUT FORMAT:
CONCURRENCY MODE: {scoop | thread | none}

DOMAIN CONCURRENT USE CASES:
  1. {use case description}
  2. {use case description}

TECHNICAL FINDINGS:
  Finding 1:
    Pattern: {SCOOP/thread pattern}
    Location: {class.feature}
    Risk: {what could happen}

DOMAIN CONCURRENCY FINDINGS:
  Finding 1:
    Scenario: {domain concurrent scenario}
    Conflict: {what could go wrong}
    Contract gap: {missing synchronization contract}
    Code gap: {missing synchronization code}

CONTRACT-CONCURRENCY MALFORMATION:
  - {contract} assumes {condition} that concurrency violates

CODE-CONCURRENCY MALFORMATION:
  - {code} does {action} unsafely under concurrency

CONCURRENCY RISK SCORE: [LOW | MEDIUM | HIGH | CRITICAL]
```

## Success Criteria
- Technical SCOOP/thread issues found
- Domain concurrency scenarios identified
- Contract-concurrency alignment checked
- Both contract and code malformation detected

## Next Step
→ H06-CONSTRUCT-EXPLOIT.md (PHASE CHANGE: Verify)
