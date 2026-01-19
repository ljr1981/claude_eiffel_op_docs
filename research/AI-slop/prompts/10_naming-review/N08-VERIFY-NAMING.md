# N08: Verify Naming

## Context
Phase: **VERIFICATION** (Final)
Purpose: Compile, test, and validate all naming fixes

## Mindset

**Trust but verify. Compilation is the truth.**

All the renaming in the world means nothing if the code doesn't compile. This step ensures all fixes are correct and complete.

## Input Required
- All fix logs from N03-N07
- Library source code

## Prompt

```
TASK: Verify all naming fixes compile and tests pass.

INPUT:
{fix_logs_from_N03_N04_N05_N06}
Library: {library_path}

═══════════════════════════════════════════════════════════════
STEP 1: FULL COMPILATION
═══════════════════════════════════════════════════════════════

Command:
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

Expected: Compilation successful
Actual: {PASTE ACTUAL OUTPUT}

If compilation fails:

COMPILATION ERROR:
  Error code: {VOIT | VUCR | VEEN | etc.}
  Message: {full error message}
  File: {file}
  Line: {line}

  Analysis:
  - Cause: {what naming issue caused this}
  - Fix: {what rename was missed or incorrect}

  Action taken:
  - {describe fix applied}

  Recompile result: {PASS | FAIL}

Repeat until: COMPILATION SUCCESSFUL

═══════════════════════════════════════════════════════════════
STEP 2: RUN TESTS
═══════════════════════════════════════════════════════════════

Command:
./EIFGENs/{lib}_tests/W_code/{lib}.exe

Expected: All tests pass
Actual: {PASTE ACTUAL OUTPUT}

Test Results:
  Total: {n}
  Passed: {n}
  Failed: {n}

If tests fail:

TEST FAILURE:
  Test: {test_name}
  Error: {error message}

  Analysis:
  - Related to naming change: {yes/no}
  - If yes, which rename: {describe}
  - Fix needed: {describe}

  Action taken:
  - {describe fix}

  Retest result: {PASS | FAIL}

Repeat until: ALL TESTS PASS

═══════════════════════════════════════════════════════════════
STEP 3: VERIFY NO REMAINING VIOLATIONS
═══════════════════════════════════════════════════════════════

Re-run quick scans to ensure no violations remain:

SCAN 1: CamelCase classes
  grep -rE "^class[[:space:]]+[A-Z][a-z]" --include="*.e" .
  Expected: No matches
  Actual: {results}

SCAN 2: CamelCase features
  grep -rE "[[:space:]][a-z]+[A-Z]" --include="*.e" .
  Expected: Minimal matches (may catch strings)
  Review: {any real violations}

SCAN 3: Arguments without a_ prefix
  grep -rE "\([[:space:]]*[^a][a-z_]+:" --include="*.e" .
  Expected: Minimal matches
  Review: {any real violations}

SCAN 4: Cursors without ic prefix
  grep -rE "as [^i][a-z_]+ loop" --include="*.e" .
  Expected: No matches
  Actual: {results}

SCAN 5: TUPLE label conflicts
  grep -rE "TUPLE \[.*count:" --include="*.e" .
  Expected: No matches
  Actual: {results}

REMAINING VIOLATIONS: {count}
{List any violations that were missed}

═══════════════════════════════════════════════════════════════
STEP 4: VERIFY ECF CONSISTENCY
═══════════════════════════════════════════════════════════════

Check ECF file for any stale class references:

File: {lib}.ecf

Check for:
□ All renamed classes updated in <class> elements
□ All renamed classes updated in <class_option> elements
□ No references to old class names

Issues found: {none | list issues}

═══════════════════════════════════════════════════════════════
STEP 5: VERIFY DOCUMENTATION CONSISTENCY
═══════════════════════════════════════════════════════════════

Check documentation for stale names:

Files to check:
□ README.md
□ docs/index.html
□ docs/*.html
□ Any inline documentation

grep for old class/feature names:
{list old names that might appear in docs}

Documentation updates needed:
- {file}: Update {old_name} to {new_name}
- ...

═══════════════════════════════════════════════════════════════
STEP 6: GENERATE SUMMARY REPORT
═══════════════════════════════════════════════════════════════

# NAMING REVIEW COMPLETE: {library_name}

## Final Status
| Check | Status |
|-------|--------|
| Compilation | {PASS/FAIL} |
| Tests | {n}/{n} passing |
| Remaining violations | {count} |
| ECF consistency | {PASS/FAIL} |
| Documentation | {updated/needs update} |

## Changes Summary
| Category | Items Fixed |
|----------|-------------|
| Class renames | {n} |
| Feature renames | {n} |
| Argument fixes | {n} |
| Local/cursor fixes | {n} |
| Contract tag fixes | {n} |
| Magic number fixes | {n} |
| **TOTAL** | {n} |

## Files Modified
{count} files modified

{List files with change counts}

## Breaking Changes
{List any public API changes that affect external clients}

Recommended changelog entry:
```
## [version] - YYYY-MM-DD

### Changed
- Renamed classes: {list}
- Renamed features: {list}
- Standardized argument naming with a_ prefix
- Standardized loop cursors with ic prefix
```

## Remaining Work
{Any follow-up items}

═══════════════════════════════════════════════════════════════
STEP 7: COMMIT CHANGES
═══════════════════════════════════════════════════════════════

If all verification passes:

git add -A
git status

Review changes:
{show staged files}

Commit:
git commit -m "$(cat <<'EOF'
refactor: standardize Eiffel naming conventions

- Classes: ALL_CAPS with underscores
- Features: snake_case (verbs for commands, nouns for queries)
- Arguments: a_ prefix
- Loop cursors: ic prefix
- Contract tags: descriptive patterns

Changes:
- {n} class renames
- {n} feature renames
- {n} argument fixes
- {n} cursor fixes

All tests passing.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

Result: {commit hash}
```

## Success Criteria
- Library compiles without errors
- All tests pass
- No remaining violations in critical categories
- ECF updated if needed
- Documentation updated if needed
- Changes committed

## Workflow Complete

The library now follows Eiffel naming conventions as defined in:
`D:\prod\reference_docs\claude\NAMING_CONVENTIONS.md`
