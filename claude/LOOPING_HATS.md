# Looping Hat Prompts

These prompts use the **iterative loop pattern**. Claude continues working autonomously until it signals completion with `<promise>KEYWORD</promise>`.

## How to Use

1. Copy the prompt block for your desired hat
2. Replace the `Target:` path with your actual file/module
3. Paste to Claude
4. Claude iterates until done, signaling with the promise tag

**Or simply say**: "Put on your [HAT NAME] for [target path]"

---

## Contractor Hat (Contract Strengthening)

### Full Module Audit

```
Put on your CONTRACTOR HAT.

Target: [YOUR_MODULE_PATH]

Your mission: Strengthen Design by Contract coverage across all classes in this module.

For each class:
1. Read the class thoroughly
2. Examine every feature (method)
3. For each feature, evaluate:
   - Are preconditions complete? (What MUST be true before calling?)
   - Are postconditions complete? (What is GUARANTEED after?)
   - Are there implicit assumptions that should be explicit contracts?
4. Add or strengthen contracts where semantically appropriate
5. Do NOT add trivial contracts (e.g., `Result /= Void` when the type is already attached)
6. DO add domain-meaningful contracts (valid ranges, state requirements, relationship invariants)

After each class, move to the next. Keep iterating through the module.

When you have reviewed ALL classes and can identify no more weak or missing contracts, output:

<promise>CONTRACTS_COMPLETE</promise>

If you find issues, fix them and continue. Do not signal completion until you're confident coverage is comprehensive.
```

### Single Class Deep Dive

```
Put on your CONTRACTOR HAT.

Target: [YOUR_CLASS_PATH]

Deep audit this single class for contract completeness.

Iteration approach:
1. First pass: Read entire class, note all features
2. Second pass: Evaluate preconditions on each feature
3. Third pass: Evaluate postconditions on each feature
4. Fourth pass: Evaluate class invariants
5. Fifth pass: Look for cross-feature relationships that should be contracted

For each issue found, fix it immediately.

After each pass, re-examine for anything missed. Continue iterating until you are confident that:
- Every feature has semantically appropriate preconditions
- Every feature has meaningful postconditions where applicable
- Class invariants capture stable state rules
- No implicit assumptions remain uncontracted

When complete, output:

<promise>CLASS_CONTRACTS_COMPLETE</promise>
```

---

## Specification Hat (Vibe-Contracting)

### Write Contracts Before Implementation

```
Put on your SPECIFICATION HAT.

Target: [YOUR_CLASS_PATH]

Your mission: Write formal contracts BEFORE implementation for new/stub features.

For each feature that needs implementation:
1. Write feature signature (name, parameters, return type)
2. Ask: "What MUST be true BEFORE this can be called?" → Write `require` clause
3. Ask: "What will be true AFTER this completes?" → Write `ensure` clause
4. Ask: "Is this TRUE but INCOMPLETE?" → Add missing postconditions
5. Check for `old` expressions needed for state change guarantees
6. Update class invariants if new state is introduced

Key questions for completeness:
- Does postcondition say item exists AND count changed?
- Does postcondition capture ALL side effects?
- Are all caller obligations documented in preconditions?

When ALL features have complete contracts (implementation can come later), output:

<promise>SPECS_COMPLETE</promise>
```

---

## Testing Hat (Test Coverage)

### Edge Case Hunter

```
Put on your TESTING HAT.

Target: [YOUR_TEST_PATH]

Your mission: Expand test coverage beyond happy paths.

For each test class:
1. Read the corresponding source class
2. Read existing tests
3. Identify gaps:
   - Boundary conditions not tested
   - Error paths not exercised
   - Contract preconditions not deliberately violated
   - Edge cases in loops, empty collections, max values
4. Write new test cases that stress these edges
5. Run tests to confirm they pass (or expose real bugs)

Keep iterating through test classes. For each gap found, add a test.

When you have reviewed ALL test classes and can identify no more coverage gaps, output:

<promise>TESTS_COMPREHENSIVE</promise>

If tests fail and reveal bugs, fix the bugs first, then continue testing.
```

### Contract Exercise Tests

```
Put on your TESTING HAT.

Target: [YOUR_CLASS_PATH] and corresponding tests

Your mission: Ensure every contract is exercised by at least one test.

Approach:
1. List all preconditions in the class
2. List all postconditions in the class
3. For each contract, find or create a test that exercises it
4. For preconditions: Create tests that SHOULD trigger the precondition (valid calls)
5. Optionally: Create tests that deliberately violate preconditions to confirm they fire

Iterate until every contract has test coverage.

When complete, output:

<promise>CONTRACTS_TESTED</promise>
```

---

## Security Hat (Security Audit)

### Injection & Input Validation Audit

```
Put on your SECURITY HAT.

Target: [YOUR_MODULE_PATH]

Your mission: Audit for security vulnerabilities, especially around external input.

Focus areas:
1. **Path traversal**: Any file paths from user input? Check for ../ attacks
2. **Command injection**: Any shell commands built from user input?
3. **JSON/XSS injection**: User input embedded without escaping?
4. **SQL injection**: Any database queries with user data?
5. **Buffer issues**: Any C external calls with user-controlled sizes?
6. **Unvalidated callbacks**: Handlers accepting arbitrary data?

For each potential vulnerability:
1. Assess severity (Critical/High/Medium/Low)
2. Implement a fix using contracts and/or validation
3. Add a contract that makes the vulnerability impossible
4. Move to next issue

Keep iterating through all source files.

When you have audited ALL files and addressed ALL identified vulnerabilities, output:

<promise>SECURITY_HARDENED</promise>
```

### FFI Boundary Audit

```
Put on your SECURITY HAT.

Target: All C external calls in [YOUR_PROJECT]

Your mission: Audit the boundary between Eiffel and C code.

For each C external:
1. What data crosses the boundary?
2. Are pointers validated before passing to C?
3. Are returned pointers checked for null?
4. Are buffer sizes validated?
5. Is memory ownership clear (who frees what)?
6. Could malformed data from C corrupt Eiffel state?

Add contracts at the Eiffel side of each boundary to enforce safety.

When all C externals have been audited and hardened, output:

<promise>FFI_SECURE</promise>
```

---

## Build Hat (Compile Until Clean)

### Clean Compile Loop

```
Put on your BUILD HAT.

Target: [YOUR_ECF_PATH]

Your mission: Achieve a clean compile.

Loop:
1. Run: /d/prod/ec.sh -batch -config [ecf] -target [target] -c_compile
2. If errors, analyze each error
3. Fix the error
4. Repeat from step 1

Do NOT stop until compilation succeeds with zero errors.

When compile is clean, output:

<promise>BUILD_CLEAN</promise>
```

### Build + Test Loop

```
Put on your BUILD HAT, then your TESTING HAT.

Target: [YOUR_PROJECT]

Your mission: Clean compile AND passing tests.

Loop:
1. Compile the project
2. If compile fails, fix errors and restart loop
3. If compile succeeds, run tests
4. If tests fail, fix failures and restart loop
5. If tests pass, done

Continue until BOTH compile AND tests succeed.

When complete, output:

<promise>BUILD_AND_TESTS_GREEN</promise>
```

---

## Refactoring Hat

### Code Smell Elimination

```
Put on your REFACTORING HAT.

Target: [YOUR_CLASS_PATH]

Your mission: Improve code structure without changing behavior.

Focus areas:
1. Long features (>50 lines) - extract into smaller features
2. Duplicate code - consolidate into shared features
3. God classes - split into focused classes
4. Long parameter lists - introduce parameter objects
5. Feature envy - move features to where they belong
6. Poor naming - improve clarity

For each refactoring:
1. Ensure tests exist for affected code
2. Make minimal change
3. Verify tests still pass
4. Move to next smell

Keep iterating until code is clean and maintainable.

When complete, output:

<promise>REFACTORING_COMPLETE</promise>
```

---

## Documentation Hat

### API Documentation Pass

```
Put on your DOCUMENTATION HAT.

Target: [YOUR_MODULE_PATH]

Your mission: Ensure all public APIs are documented.

For each class:
1. Check class-level `note` clause has description
2. For each public feature, verify:
   - Comment explains purpose (why, not what)
   - Parameters are described if not obvious
   - Return value is described
   - Complex logic has inline comments
3. Add examples in `note` clauses where helpful
4. Update any stale documentation

Keep iterating through all classes.

When all public APIs are documented, output:

<promise>DOCS_COMPLETE</promise>
```

---

## Logging Hat (Diagnostics)

### Add Strategic Logging

```
Put on your LOGGING HAT.

Target: [YOUR_CLASS_PATH]

Your mission: Add diagnostic logging that tells the "story" of execution.

Guidelines:
1. Log at feature entry with key parameter values
2. Log at decision points (which branch taken and why)
3. Log before external calls (AI providers, file I/O, FFI)
4. Log after external calls (success/failure, key results)
5. Use appropriate log levels (debug, info, warning, error)
6. Do NOT log sensitive data (API keys, personal info)

The goal: If a user hits a bug, the log should tell us exactly how the code got to the failure point.

When logging coverage is comprehensive, output:

<promise>LOGGING_COMPLETE</promise>
```

---

## Code Review Hat

### Deep Review Pass

```
Put on your CODE REVIEW HAT.

Target: [YOUR_CLASS_PATH]

Your mission: Comprehensive code review with actionable findings.

Review checklist:
1. **Correctness**: Logic correct? Edge cases handled?
2. **Contracts**: Preconditions/postconditions appropriate?
3. **Security**: Input validated? Injection prevented?
4. **Performance**: Any obvious inefficiencies?
5. **Maintainability**: Code readable? Well-named?
6. **Eiffel-specific**: Void safety? STRING_8 vs STRING_32?

For each issue found:
1. Note location (file:line)
2. Assess severity (Critical/High/Medium/Low)
3. Provide specific recommendation
4. Fix Critical/High issues immediately

When review is complete and all Critical/High issues fixed, output:

<promise>REVIEW_COMPLETE</promise>
```

---

## Performance Hat

### Optimization Pass

```
Put on your PERFORMANCE HAT.

Target: [YOUR_CLASS_PATH]

Your mission: Identify and fix performance bottlenecks.

Approach:
1. Identify potential hot spots (loops, repeated operations, I/O)
2. Look for:
   - Unnecessary object creation in loops
   - Repeated calculations that could be cached
   - Inefficient algorithms (O(n²) when O(n) possible)
   - Redundant I/O operations
3. For each bottleneck found, implement optimization
4. Document performance-critical code with comments

Keep iterating until no more obvious optimizations remain.

When complete, output:

<promise>OPTIMIZED</promise>
```

---

## Cleanup Hat

### Code Hygiene Pass

```
Put on your CLEANUP HAT.

Target: [YOUR_MODULE_PATH]

Your mission: General code hygiene and maintenance.

Focus:
1. Remove dead code (unused features, commented-out code)
2. Fix compiler warnings
3. Update obsolete patterns
4. Remove debugging code left in production
5. Ensure consistent formatting

For each issue found, fix it immediately.

When codebase is clean and warning-free, output:

<promise>CLEANUP_COMPLETE</promise>
```

---

## Code Smell Detector Hat

### Smell Identification Pass

```
Put on your CODE SMELL DETECTOR HAT.

Target: [YOUR_MODULE_PATH]

Your mission: Identify code smells and design problems (report only, don't fix).

Look for:
| Smell | Signs |
|-------|-------|
| God Class | >1000 lines, many unrelated features |
| Long Feature | >50 lines, multiple nesting levels |
| Feature Envy | Feature uses another class more than its own |
| Duplicate Code | Copy-pasted code with variations |
| Long Parameter List | >4 parameters |
| Dead Code | Unreachable/unused features |

For each smell found:
1. Note location
2. Assess severity (High/Medium/Low impact)
3. Suggest refactoring approach

Output a smell report, then signal:

<promise>SMELLS_CATALOGED</promise>
```

---

## SCOOP Hat (Concurrency)

### Concurrency Opportunity Analysis

```
Put on your SCOOP HAT.

Target: [YOUR_MODULE_PATH]

Your mission: Identify opportunities for SCOOP concurrency.

Good candidates for `separate`:
- Independent batch operations
- I/O operations that can overlap
- Background tasks (logging, cleanup)
- Producer-consumer patterns

Poor candidates:
- Tightly coupled operations
- Very short operations (overhead exceeds benefit)
- Sequential algorithms with dependencies

For each opportunity found:
1. Assess benefit vs complexity
2. Design locking strategy (routine arguments)
3. Implement if benefit is clear
4. Test for correctness under concurrency

When analysis complete and high-value opportunities implemented, output:

<promise>CONCURRENCY_ANALYZED</promise>
```

---

## Combined Hat Sequences

### Full Quality Pass

```
This is a multi-hat sequence. Complete each phase before moving to the next.

Target: [YOUR_MODULE_PATH]

PHASE 1 - CONTRACTOR HAT:
Strengthen contracts. Signal with <promise>PHASE1_CONTRACTS</promise>

PHASE 2 - TESTING HAT:
Expand test coverage. Signal with <promise>PHASE2_TESTS</promise>

PHASE 3 - SECURITY HAT:
Audit for vulnerabilities. Signal with <promise>PHASE3_SECURITY</promise>

PHASE 4 - BUILD HAT:
Ensure clean compile and passing tests. Signal with <promise>PHASE4_BUILD</promise>

When ALL phases complete, output:

<promise>QUALITY_PASS_COMPLETE</promise>
```

### Pre-Release Checklist

```
Multi-hat sequence for release preparation.

Target: [YOUR_PROJECT]

PHASE 1 - CLEANUP HAT:
Remove dead code, fix warnings. Signal: <promise>PHASE1_CLEANUP</promise>

PHASE 2 - DOCUMENTATION HAT:
Ensure all public APIs documented. Signal: <promise>PHASE2_DOCS</promise>

PHASE 3 - SECURITY HAT:
Final security audit. Signal: <promise>PHASE3_SECURITY</promise>

PHASE 4 - BUILD HAT:
Clean release build. Signal: <promise>PHASE4_BUILD</promise>

When ready for release, output:

<promise>RELEASE_READY</promise>
```

---

## Usage Notes

1. **Max iterations**: Set a mental limit. If Claude loops >15 times without progress, intervene.

2. **Token awareness**: Long loops consume tokens. Start with single-class targets before module-wide sweeps.

3. **Review promises**: When Claude signals completion, review what was done. Trust but verify.

4. **Compound loops**: The "Combined Hat Sequences" are ambitious. Consider running hats separately first.

5. **Escape hatch**: If Claude gets stuck, interrupt and refine the prompt.

6. **Quick invocation**: Just say "Put on your [HAT] for [path]" - no need to paste the full prompt.

---

## Update Log

| Date | Change |
|------|--------|
| 2026-01-08 | Created comprehensive looping hats from HATS.md patterns |
