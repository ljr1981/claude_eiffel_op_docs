# X04: Adversarial Tests - EXECUTABLE

## THIS IS AN EXECUTABLE STEP

**You MUST write actual test code, compile it, run it, and document actual results.**

## Actions Required

1. **CREATE** `testing/adversarial_tests.e` in the target library
2. **COMPILE** using `ec.sh`
3. **RUN** the tests
4. **DOCUMENT** actual output in `hardening/X04-TESTS-LOG.md`

## Step-by-Step Execution

### Step 1: Create Test File

Create `{lib}/testing/adversarial_tests.e`:

```eiffel
note
    description: "Adversarial tests for {LIB_NAME}"
    date: "{DATE}"

class
    ADVERSARIAL_TESTS

create
    make

feature {NONE} -- Initialization

    make
        do
            passed := 0
            failed := 0
            risk := 0
        end

feature -- Counters

    passed, failed, risk: INTEGER

feature -- Input Attack Tests

    test_empty_string_input
        local
            l_obj: {MAIN_CLASS}
            l_retried: BOOLEAN
        do
            if not l_retried then
                create l_obj.make
                -- Call feature with empty string
                -- Document what happens
                passed := passed + 1
                print ("  PASS: test_empty_string_input%N")
            end
        rescue
            failed := failed + 1
            print ("  FAIL: test_empty_string_input - " + (create {EXCEPTION_MANAGER}).last_exception.description + "%N")
            l_retried := True
            retry
        end

    -- ADD MORE TESTS FOR:
    -- test_null_byte_content
    -- test_control_characters
    -- test_very_long_input
    -- test_boundary_values
    -- test_special_characters

feature -- Run All

    run_all
        do
            print ("%N=== Adversarial Tests ===%N")
            test_empty_string_input
            -- Call all other tests
            print ("%N=== Summary: " + passed.out + " pass, " + failed.out + " fail, " + risk.out + " risk ===%N")
        end

end
```

### Step 2: Integrate Into Test Runner

Edit `{lib}/testing/test_app.e` to call adversarial tests:

```eiffel
-- Add to make:
    run_adversarial_tests

-- Add feature:
    run_adversarial_tests
        do
            create adversarial_tests.make
            adversarial_tests.run_all
        end

-- Add attribute:
    adversarial_tests: ADVERSARIAL_TESTS
```

### Step 3: COMPILE (MANDATORY)

```bash
cd /d/prod/{lib}
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile
```

**PASTE THE ACTUAL OUTPUT INTO YOUR LOG.**

If compilation fails:
- Fix the errors
- Recompile
- Document the fix

### Step 4: RUN TESTS (MANDATORY)

```bash
./EIFGENs/{lib}_tests/W_code/{lib}.exe
```

**PASTE THE ACTUAL OUTPUT INTO YOUR LOG.**

### Step 5: Document in `hardening/X04-TESTS-LOG.md`

```markdown
# X04: Adversarial Tests Log - {LIB_NAME}

## Date: {ACTUAL DATE}

## Tests Written

| Test Name | Category | Input |
|-----------|----------|-------|
| test_empty_string_input | Empty Input | "" |
| test_null_byte_content | Special Chars | "%U" |
| ... | ... | ... |

## Compilation Output

```
[PASTE ACTUAL ec.sh OUTPUT HERE]
```

## Test Execution Output

```
[PASTE ACTUAL TEST RUN OUTPUT HERE]
```

## Results

| Category | Tests | Pass | Fail | Risk |
|----------|-------|------|------|------|
| Empty Input | X | X | X | X |
| Special Chars | X | X | X | X |
| Boundaries | X | X | X | X |
| **Total** | **X** | **X** | **X** | **X** |

## Bugs Found

### BUG-001: {Title}
- **Test**: test_X
- **Input**: {actual input}
- **Expected**: {what should happen}
- **Actual**: {PASTE ACTUAL ERROR/OUTPUT}
- **Status**: Open

## Files Modified

- `testing/adversarial_tests.e` - Created with X tests
- `testing/test_app.e` - Added adversarial test runner
```

## Test Categories to Implement

For each category, write at least 2 tests:

### 1. Empty Input (2+ tests)
- Empty strings
- Empty collections
- Zero values

### 2. Special Characters (2+ tests)
- Null bytes (%U)
- Control characters (%N, %R, %T)
- Unicode

### 3. Boundary Values (2+ tests)
- Maximum integer
- Very long strings (10K+ chars)
- Single element collections

### 4. State Attacks (2+ tests)
- Reuse after error
- Multiple operations same instance

### 5. Error Paths (2+ tests)
- Missing files
- Invalid input

## Minimum Requirements

- [ ] At least 10 adversarial tests written
- [ ] Test file compiles
- [ ] Tests actually run
- [ ] Results documented with ACTUAL output
- [ ] Any failures documented as bugs

## VERIFICATION CHECKPOINT

Before proceeding to X05, confirm:

```
Compilation: [SUCCESS/FAILED]
Tests Run: [NUMBER]
Tests Passed: [NUMBER]
Tests Failed: [NUMBER]
Bugs Found: [NUMBER]
```

## Next Step

→ X05-STRESS-ATTACK.md
