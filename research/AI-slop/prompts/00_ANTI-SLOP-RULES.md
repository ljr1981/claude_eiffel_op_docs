# ANTI-SLOP RULES: Mandatory for ALL Prompt Workflows

## What is "AI Slop"?

AI slop is documentation that LOOKS like work was done, but:
- No code was actually compiled
- No tests were actually run
- Results are "expected" or "hypothetical" rather than actual
- Claims are not backed by evidence

**AI slop is forbidden in all prompt workflows.**

## Universal Rules for ALL Workflows

### RULE 1: No Hypothetical Results

```
✗ FORBIDDEN:
  "This test WOULD pass..."
  "The expected result is..."
  "This should compile..."
  "The code would handle..."

✓ REQUIRED:
  "Test output: PASS" [from actual execution]
  "Compilation output: System Recompiled" [from actual ec.sh]
  "Result: 47 tests, 47 pass, 0 fail" [from actual run]
```

### RULE 2: Evidence Required

Every claim must have evidence:

| Claim | Required Evidence |
|-------|-------------------|
| "Code compiles" | Paste ec.sh output |
| "Tests pass" | Paste test execution output |
| "Feature works" | Show test that verifies it |
| "Bug found" | Show error message or failure |
| "Bug fixed" | Show test that now passes |

### RULE 3: Compile Before Document

For ANY step that modifies code:

```bash
# Step 1: Make change
[edit code]

# Step 2: COMPILE (mandatory)
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# Step 3: Verify compilation succeeded
[paste output showing "System Recompiled"]

# Step 4: RUN tests (mandatory)
./EIFGENs/{lib}_tests/W_code/{lib}.exe

# Step 5: Paste results
[paste actual test output]

# Step 6: THEN document
[only now write documentation]
```

### RULE 4: No Imagined Code Behavior

```
✗ FORBIDDEN:
  Reading code and claiming "this feature handles X gracefully"
  without actually testing it

✓ REQUIRED:
  Write a test for X
  Compile the test
  Run the test
  Document actual result
```

### RULE 5: Document ACTUAL Not EXPECTED

```
✗ FORBIDDEN:
  "Expected: handles empty strings gracefully"
  "Result: PASS (assumed)"

✓ REQUIRED:
  "Input: empty string"
  "Actual output: [paste what actually happened]"
  "Result: PASS" or "Result: FAIL - [actual error]"
```

## Verification Checkpoint Format

Every step that involves code MUST end with:

```markdown
## VERIFICATION CHECKPOINT

### Compilation
```
[PASTE ACTUAL ec.sh OUTPUT - not a description, the actual output]
```

### Test Execution
```
[PASTE ACTUAL test output - not a summary, the actual output]
```

### Summary (from actual counts)
- Compiled: YES (System Recompiled message shown)
- Tests run: [COUNT from actual output]
- Tests passed: [COUNT from actual output]
- Tests failed: [COUNT from actual output]
```

## Workflow-Specific Requirements

### For maintenance-xtreme:
- X01: Must compile baseline, run existing tests
- X04: Must write tests, compile, run, document actual results
- X05: Must write stress tests, compile, run, document actual results
- X10: Must run full test suite, document actual final results

### For spec-extraction:
- Must read actual source files (not imagine them)
- Specs must match actual code structure

### For design-audit:
- Must compile after any suggested changes
- Any refactoring must be verified with tests

### For project-creation:
- Must compile at each phase
- Must run tests before moving to next phase

## How to Know You're Doing Real Work

Ask yourself:
1. Did I run `ec.sh`? Can I paste the output?
2. Did I run the test executable? Can I paste the output?
3. Are my numbers from actual execution, not guessing?
4. If I claim something works, is there a test proving it?

If you answer "no" to any of these, you haven't done the step.

## Documentation Location

All documentation goes in the target library folder:

```
/d/prod/{lib}/
├── hardening/      # Maintenance-xtreme docs
├── specs/          # Spec-extraction docs
├── audit/          # Design-audit docs
├── research/       # Deep-research docs
└── testing/        # Test files (actual .e files)
```

**Documentation in the prompts folder is a template.**
**Documentation in the library folder is actual work.**

## Final Check

Before claiming any workflow step is complete:

```
[ ] I ran ec.sh and pasted the ACTUAL output
[ ] I ran tests and pasted the ACTUAL output
[ ] All numbers are from ACTUAL execution
[ ] I can point to the specific line in my output proving each claim
[ ] The documentation is in {lib}/ not in prompts/
```

If any checkbox is unchecked, the step is NOT complete.
