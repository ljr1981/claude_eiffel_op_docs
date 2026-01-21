# Maintenance-Xtreme Workflow: EXECUTABLE Adversarial Code Hardening

## CRITICAL: This is an EXECUTABLE Workflow

**YOU MUST COMPILE AND RUN CODE AT EVERY STEP.**

This is NOT a documentation exercise. Every prompt in this workflow requires:
1. Actual code modifications
2. Actual compilation using `ec.sh`
3. Actual test execution
4. Documentation of ACTUAL results (not hypothetical)

**If you haven't run `ec.sh` and executed tests, you haven't done the step.**

## Anti-Slop Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                    MANDATORY VERIFICATION                        │
│                                                                  │
│  ✗ FORBIDDEN: "This test WOULD pass/fail..."                    │
│  ✗ FORBIDDEN: "Expected behavior: ..."                          │
│  ✗ FORBIDDEN: Documenting without compiling                     │
│  ✗ FORBIDDEN: Test results without running tests                │
│                                                                  │
│  ✓ REQUIRED: Actual ec.sh compilation output                    │
│  ✓ REQUIRED: Actual test execution output                       │
│  ✓ REQUIRED: Actual error messages (if any)                     │
│  ✓ REQUIRED: Code diffs showing what changed                    │
└─────────────────────────────────────────────────────────────────┘
```

## Workflow Steps

| Step | File | MANDATORY Actions |
|------|------|-------------------|
| X01 | RECONNAISSANCE | Read source files, list features to attack |
| X02 | VULNERABILITY-SCAN | Identify attack vectors from actual code |
| X03 | CONTRACT-ASSAULT | **WRITE** contracts, **COMPILE**, record failures |
| X04 | ADVERSARIAL-TESTS | **WRITE** tests, **COMPILE**, **RUN**, record results |
| X05 | STRESS-ATTACK | **WRITE** stress tests, **COMPILE**, **RUN**, record results |
| X06 | MUTATION-WARFARE | **MODIFY** code, **COMPILE**, see what breaks |
| X07 | TRIAGE-FINDINGS | Categorize ACTUAL findings from previous steps |
| X08 | SURGICAL-FIXES | **FIX** code, **COMPILE**, **RUN** tests to verify |
| X09 | HARDEN-DEFENSES | **ADD** defensive code, **COMPILE**, **RUN** tests |
| X10 | VERIFY-HARDENING | **RUN** full test suite, document ACTUAL results |

## Required Commands

Every step involving code changes MUST use:

```bash
# Compile
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# Run tests
./EIFGENs/{lib}_tests/W_code/{lib}.exe
```

## Output Location

All documentation goes in the target library folder:

```
{simple_lib}/
├── hardening/
│   ├── X01-RECON-ACTUAL.md      # Actual findings
│   ├── X02-VULNS-ACTUAL.md      # Actual vulnerabilities
│   ├── X03-CONTRACTS-LOG.md     # Contracts added + compile results
│   ├── X04-TESTS-LOG.md         # Tests written + run results
│   ├── X05-STRESS-LOG.md        # Stress tests + run results
│   ├── X06-MUTATION-LOG.md      # Mutations + what broke
│   ├── X07-TRIAGE.md            # Prioritized actual bugs
│   ├── X08-FIXES-LOG.md         # Fixes applied + verification
│   ├── X09-HARDENING-LOG.md     # Defensive code + verification
│   └── X10-FINAL-VERIFIED.md    # Final test results (ACTUAL)
└── testing/
    ├── adversarial_tests.e      # Actual test file (committed)
    └── stress_tests.e           # Actual stress test file
```

## Verification Checkpoint Format

Every step MUST end with a verification block:

```markdown
## VERIFICATION CHECKPOINT

### Compilation
```
[PASTE ACTUAL ec.sh OUTPUT HERE]
```

### Test Execution
```
[PASTE ACTUAL TEST OUTPUT HERE]
```

### Results
- Tests run: [ACTUAL NUMBER]
- Tests passed: [ACTUAL NUMBER]
- Tests failed: [ACTUAL NUMBER]
- Errors: [ACTUAL ERROR MESSAGES OR "None"]
```

## Workflow Execution

To run this workflow on a library:

```
Apply maintenance-xtreme to simple_{name}
```

Claude Code will then:
1. Execute X01 through X10 in order
2. **ACTUALLY** compile and run tests at each step
3. Document **ACTUAL** results in {simple_lib}/hardening/
4. Commit test files to {simple_lib}/testing/

**NO STEP IS COMPLETE WITHOUT COMPILATION AND TEST EXECUTION.**
