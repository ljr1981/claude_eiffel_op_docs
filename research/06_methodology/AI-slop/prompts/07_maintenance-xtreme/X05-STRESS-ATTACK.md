# X05: Stress Attack - EXECUTABLE

## THIS IS AN EXECUTABLE STEP

**You MUST write actual stress tests, compile them, run them, and document actual results.**

## Actions Required

1. **CREATE** `testing/stress_tests.e` in the target library
2. **UPDATE** `testing/test_app.e` to run stress tests
3. **COMPILE** using `ec.sh`
4. **RUN** the tests
5. **DOCUMENT** actual output in `hardening/X05-STRESS-LOG.md`

## Step-by-Step Execution

### Step 1: Create Stress Test File

Create `{lib}/testing/stress_tests.e`:

```eiffel
note
    description: "Stress tests for {LIB_NAME}"
    date: "{DATE}"

class
    STRESS_TESTS

create
    make

feature {NONE} -- Initialization

    make
        do
            print ("Running STRESS tests for {lib}...%N%N")
            run_all_stress_tests
        end

feature -- Volume Tests

    test_100_items
        local
            l_obj: {MAIN_CLASS}
            l_input: STRING
            i: INTEGER
            l_retried: BOOLEAN
        do
            if not l_retried then
                print ("Testing 100 items... ")
                create l_obj.make
                create l_input.make (1000)
                from i := 1 until i > 100 loop
                    l_input.append ("item" + i.out + "%N")
                    i := i + 1
                end
                -- Call the feature under test
                -- l_obj.process (l_input)
                print ("PASS%N")
            end
        rescue
            print ("CRASHED%N")
            l_retried := True
            retry
        end

    test_1000_items
        -- Same pattern with 1000 items
        local
            l_retried: BOOLEAN
        do
            if not l_retried then
                print ("Testing 1000 items... ")
                -- Implementation
                print ("PASS%N")
            end
        rescue
            print ("CRASHED%N")
            l_retried := True
            retry
        end

    test_5000_items
        -- Same pattern with 5000 items
        local
            l_retried: BOOLEAN
        do
            if not l_retried then
                print ("Testing 5000 items... ")
                -- Implementation
                print ("PASS%N")
            end
        rescue
            print ("CRASHED%N")
            l_retried := True
            retry
        end

feature -- Worst Case Tests

    test_worst_case
            -- Test with pathological input (e.g., all different for diff)
        local
            l_retried: BOOLEAN
        do
            if not l_retried then
                print ("Testing worst case... ")
                -- Implementation specific to library
                print ("PASS%N")
            end
        rescue
            print ("CRASHED%N")
            l_retried := True
            retry
        end

feature -- Run All

    run_all_stress_tests
        do
            print ("=== Volume Tests ===%N")
            test_100_items
            test_1000_items
            test_5000_items

            print ("%N=== Worst Case Tests ===%N")
            test_worst_case

            print ("%N=== Stress Testing Complete ===%N")
        end

end
```

### Step 2: Integrate Into Test Runner

Edit `{lib}/testing/test_app.e`:

```eiffel
-- Add to make:
    run_stress_tests

-- Add feature:
    run_stress_tests
        do
            create stress_tests.make
        end

-- Add attribute:
    stress_tests: STRESS_TESTS
```

### Step 3: COMPILE (MANDATORY)

```bash
cd /d/prod/{lib}
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile
```

**PASTE THE ACTUAL OUTPUT.**

### Step 4: RUN TESTS (MANDATORY)

```bash
./EIFGENs/{lib}_tests/W_code/{lib}.exe
```

**PASTE THE ACTUAL OUTPUT.**

### Step 5: Document in `hardening/X05-STRESS-LOG.md`

```markdown
# X05: Stress Tests Log - {LIB_NAME}

## Date: {ACTUAL DATE}

## Tests Written

| Test Name | Size | Type |
|-----------|------|------|
| test_100_items | 100 | Volume |
| test_1000_items | 1000 | Volume |
| test_5000_items | 5000 | Volume |
| test_worst_case | N/A | Pathological |

## Compilation Output

```
[PASTE ACTUAL ec.sh OUTPUT HERE]
```

## Test Execution Output

```
[PASTE ACTUAL TEST RUN OUTPUT HERE]
```

## Results

| Test | Input Size | Result | Notes |
|------|------------|--------|-------|
| test_100_items | 100 | PASS | Fast |
| test_1000_items | 1000 | PASS | OK |
| test_5000_items | 5000 | PASS/SLOW/CRASH | {notes} |
| test_worst_case | varies | PASS/FAIL | {notes} |

## Performance Scaling

| Size | Observed Behavior |
|------|-------------------|
| 100 | Fast |
| 1000 | Acceptable |
| 5000 | [ACTUAL OBSERVATION] |

## Limits Found

| Feature | Limit | Type | Evidence |
|---------|-------|------|----------|
| {feature} | {size} | {type} | {what happened} |

## Files Modified

- `testing/stress_tests.e` - Created
- `testing/test_app.e` - Added stress test runner
```

## Stress Test Categories

### Volume Tests (REQUIRED)
Test with increasing sizes:
- 100 items (baseline)
- 1000 items
- 5000 items
- 10000 items (if feasible)

### Worst Case Tests (REQUIRED)
Find pathological inputs:
- For diff: completely different files
- For search: element not found
- For sort: already sorted / reverse sorted

### Frequency Tests (OPTIONAL)
Rapid repeated calls:
- 1000 calls in a loop
- Check for memory growth

## Minimum Requirements

- [ ] At least 4 stress tests written
- [ ] Volume tests at multiple sizes
- [ ] At least one worst-case test
- [ ] Tests compile
- [ ] Tests run
- [ ] Results documented with ACTUAL output

## VERIFICATION CHECKPOINT

Before proceeding to X06, confirm:

```
Compilation: [SUCCESS/FAILED]
Stress Tests Run: [NUMBER]
Tests Passed: [NUMBER]
Tests Crashed: [NUMBER]
Limits Found: [NUMBER]
```

## Next Step

→ X06-MUTATION-WARFARE.md
