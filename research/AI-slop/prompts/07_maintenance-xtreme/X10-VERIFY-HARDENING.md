# X10: Verify Hardening - EXECUTABLE

## THIS IS THE FINAL VERIFICATION STEP

**You MUST compile and run ALL tests, documenting ACTUAL results.**

## Actions Required

1. **COMPILE** final codebase using `ec.sh`
2. **RUN** complete test suite
3. **VERIFY** all tests pass
4. **DOCUMENT** actual results in `hardening/X10-FINAL-VERIFIED.md`

## Step-by-Step Execution

### Step 1: COMPILE Final Codebase (MANDATORY)

```bash
cd /d/prod/{lib}
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile
```

**PASTE THE ACTUAL COMPILATION OUTPUT.**

Verify:
- Compiles with no errors
- Note any warnings

### Step 2: RUN Complete Test Suite (MANDATORY)

```bash
./EIFGENs/{lib}_tests/W_code/{lib}.exe
```

**PASTE THE ENTIRE ACTUAL TEST OUTPUT.**

This should include:
- Original library tests
- Adversarial tests (from X04)
- Stress tests (from X05)

### Step 3: Count Results

From the actual test output, count:
- Total tests run
- Tests passed
- Tests failed
- Tests with risk flags

### Step 4: Create `hardening/X10-FINAL-VERIFIED.md`

```markdown
# X10: Final Verification - {LIB_NAME}

## Date: {ACTUAL DATE}

## VERIFIED TEST RESULTS

### Compilation Output
```
[PASTE ACTUAL ec.sh OUTPUT]
```

### Test Execution Output
```
[PASTE ENTIRE ACTUAL TEST OUTPUT]
```

## Summary

| Category | Tests | Passed | Failed |
|----------|-------|--------|--------|
| Original | [N] | [N] | [N] |
| Adversarial | [N] | [N] | [N] |
| Stress | [N] | [N] | [N] |
| **TOTAL** | **[N]** | **[N]** | **[N]** |

**Pass Rate**: [N]% ([pass]/[total])
**Crash Rate**: [N]%

## Files Added During Hardening

- `testing/adversarial_tests.e` - [N] adversarial tests
- `testing/stress_tests.e` - [N] stress tests
- `hardening/X01-RECON-ACTUAL.md` - Reconnaissance findings
- `hardening/X04-TESTS-LOG.md` - Adversarial test log
- `hardening/X05-STRESS-LOG.md` - Stress test log
- `hardening/X10-FINAL-VERIFIED.md` - This file

## Verified Findings

### From Adversarial Testing
[SUMMARIZE ACTUAL FINDINGS FROM X04]

### From Stress Testing
[SUMMARIZE ACTUAL FINDINGS FROM X05]

## Verification Status

- [ ] Code compiles without errors
- [ ] All original tests pass
- [ ] All adversarial tests pass (or risks documented)
- [ ] All stress tests pass (or limits documented)
- [ ] Test files committed to repository

## Conclusion

{LIB_NAME} has been hardened through the Maintenance-Xtreme workflow.

**Final Test Results**: [N] tests, [N] pass, [N] fail

*Verified by actual compilation and execution on {DATE}*
*Compiler: EiffelStudio {VERSION}*
```

## CRITICAL: What Constitutes Completion

### REQUIRED for X10 to be complete:
1. Final compilation output pasted (ACTUAL)
2. Final test output pasted (ACTUAL)
3. All test counts are from ACTUAL execution
4. Summary table has REAL numbers

### FORBIDDEN:
- "Expected results..." (must be ACTUAL results)
- "Would pass..." (must ACTUALLY run)
- Hypothetical summaries
- Imagined test output

## Verification Checklist

Before declaring hardening complete:

```
[ ] Compilation succeeded (actual output shown)
[ ] Tests ran (actual output shown)
[ ] Original tests: [ACTUAL NUMBER] pass
[ ] Adversarial tests: [ACTUAL NUMBER] pass
[ ] Stress tests: [ACTUAL NUMBER] pass
[ ] All test files exist in testing/
[ ] All log files exist in hardening/
```

## Workflow Complete

Once this step is done with ACTUAL verification:

1. All hardening documentation is in `{lib}/hardening/`
2. All new tests are in `{lib}/testing/`
3. Final test results are VERIFIED by actual execution

**The maintenance-xtreme workflow is complete only when X10-FINAL-VERIFIED.md contains ACTUAL test output.**
