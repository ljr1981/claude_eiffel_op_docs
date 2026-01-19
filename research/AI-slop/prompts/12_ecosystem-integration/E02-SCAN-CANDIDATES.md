# E02: Scan Candidate Libraries

## Objective
Find existing simple_* libraries that could benefit from the new library.

## Steps

### 1. List All simple_* Libraries
```bash
ls -d /d/prod/simple_*/ | xargs -I {} basename {}
```

### 2. Check Current Dependencies
```bash
# See which libraries already use this library
for ecf in /d/prod/simple_*/*.ecf; do
  if grep -q "{lib_name}" "$ecf" 2>/dev/null; then
    basename $(dirname "$ecf")
  fi
done
```

### 3. Identify Candidate Categories

Based on the library's capabilities, identify relevant categories:

| Capability | Candidate Categories |
|------------|---------------------|
| Encoding | Serialization (json, xml, yaml), Network (http, email), File I/O |
| Factory/Pool | Database (sql, postgres), Network (http, mq), Caching |
| Reflection | Serialization, Mocking, Diffing, Validation |
| Validation | Any library accepting external input |
| Logging | All libraries (universal concern) |

### 4. Check Candidate Maturity
```bash
# For each candidate, check maturity
for lib in {candidate_list}; do
  if [ -f "/d/prod/$lib/$lib.ecf" ]; then
    src_count=$(ls /d/prod/$lib/src/*.e 2>/dev/null | wc -l)
    test_count=$(ls /d/prod/$lib/testing/*.e 2>/dev/null | wc -l)
    echo "$lib: $src_count src, $test_count tests"
  fi
done
```

### 5. Filter by Relevance

Keep candidates that:
- Have source files (not empty shells)
- Don't already use the library
- Have a clear integration use case

## Output
List of candidate libraries with their current state.

## Verification Checkpoint
- [ ] All simple_* libraries listed
- [ ] Current users identified
- [ ] Candidates categorized
- [ ] Maturity checked
- [ ] Filtered list created

## Next Step
â†’ E03-ANALYZE-INTEGRATION.md
