# N01: Scan Naming Violations

## Context
Phase: **DETECTION** (1 of 2)
Purpose: Find all naming convention violations in the codebase

## Reference
```
D:\prod\reference_docs\claude\NAMING_CONVENTIONS.md
```

## Mindset

**Naming is not cosmetic. Bad naming causes real bugs.**

VOIT(2) errors from loop cursor conflicts. VUCR errors from shadowing. Confusion from inconsistent style. Find them all.

## Input Required
- Library path
- All `.e` source files

## Prompt

```
TASK: Scan all Eiffel source files for naming convention violations.

INPUT:
Library: {library_path}
Source files: {list of .e files}

Reference: D:\prod\reference_docs\claude\NAMING_CONVENTIONS.md

═══════════════════════════════════════════════════════════════
SCAN 1: CLASS NAMES
Convention: ALL_CAPS with underscores
═══════════════════════════════════════════════════════════════

For each class, check:
□ Is it ALL_CAPS? (not CamelCase, not lowercase)
□ Are words separated by underscores?
□ Are abbreviations acceptable? (HTTP, JSON, SQL = OK)
□ Does filename match class name?

VIOLATIONS:

CLASS NAME VIOLATION: {class_name}
  File: {file_path}
  Problem: {CamelCase | lowercase | missing_underscores | abbreviation}
  Current: {current_name}
  Should be: {correct_name}
  Severity: HIGH

═══════════════════════════════════════════════════════════════
SCAN 2: FEATURE NAMES
Convention: all_lowercase with underscores
═══════════════════════════════════════════════════════════════

For each feature, check:
□ Is it all_lowercase?
□ Are words separated by underscores?
□ Commands use verbs (imperative)?
□ Queries use nouns?
□ Boolean queries use is_/has_ prefix?
□ Creation procedures use make prefix?

VIOLATIONS:

FEATURE NAME VIOLATION: {class}.{feature}
  File: {file_path}:{line}
  Problem: {CamelCase | wrong_prefix | verb_for_query | noun_for_command}
  Current: {current_name}
  Should be: {correct_name}
  Severity: {HIGH | MEDIUM}

═══════════════════════════════════════════════════════════════
SCAN 3: CONSTANT NAMES
Convention: Initial_cap with underscores
═══════════════════════════════════════════════════════════════

For each constant (manifest or once), check:
□ Does it start with capital letter?
□ Are remaining letters lowercase (except after underscore)?

VIOLATIONS:

CONSTANT NAME VIOLATION: {class}.{constant}
  File: {file_path}:{line}
  Problem: {ALL_CAPS | all_lowercase | CamelCase}
  Current: {current_name}
  Should be: {correct_name}
  Severity: MEDIUM

═══════════════════════════════════════════════════════════════
SCAN 4: ARGUMENT NAMES
Convention: a_ prefix + lowercase
═══════════════════════════════════════════════════════════════

For each argument, check:
□ Does it have a_ prefix?
□ Is the rest lowercase with underscores?

VIOLATIONS:

ARGUMENT NAME VIOLATION: {class}.{feature}({argument})
  File: {file_path}:{line}
  Problem: {missing_a_prefix | CamelCase | shadows_attribute}
  Current: {current_name}
  Should be: {correct_name}
  Severity: MEDIUM

═══════════════════════════════════════════════════════════════
SCAN 5: LOCAL VARIABLE NAMES
Convention: lowercase, abbreviations OK
═══════════════════════════════════════════════════════════════

For each local variable, check:
□ Is it lowercase?
□ Does it shadow a feature? (potential VUCR)

ACCEPTABLE (do NOT flag as violations):
- Short loop counters: i, j, k, n, c, etc. (conventional in from-loops)
- Abbreviated locals: s (STRING), l (LIST), etc.
- These are standard practice and do not need l_ prefix

VIOLATIONS:

LOCAL NAME VIOLATION: {class}.{feature} local {variable}
  File: {file_path}:{line}
  Problem: {CamelCase | shadows_feature}
  Current: {current_name}
  Should be: {correct_name}
  Severity: {CRITICAL if shadows | LOW otherwise}

═══════════════════════════════════════════════════════════════
SCAN 6: LOOP CURSOR NAMES
Convention: ic or ic_ prefix
═══════════════════════════════════════════════════════════════

For each `across ... as X` cursor, check:
□ Does it use ic or ic_ prefix?
□ Does it shadow a feature? (VOIT(2) error)

VIOLATIONS:

CURSOR NAME VIOLATION: {class}.{feature} cursor {cursor}
  File: {file_path}:{line}
  Problem: {no_ic_prefix | shadows_feature}
  Current: across {structure} as {cursor}
  Should be: across {structure} as ic_{suggested}
  Shadows: {feature_name} (if applicable)
  Severity: CRITICAL (if shadows) | MEDIUM (if no prefix)

═══════════════════════════════════════════════════════════════
SCAN 7: GENERIC PARAMETER NAMES
Convention: Single uppercase letter
═══════════════════════════════════════════════════════════════

For each generic parameter, check:
□ Is it a single uppercase letter?
□ Common: G, K, V, T

VIOLATIONS:

GENERIC NAME VIOLATION: {class} [{param}]
  File: {file_path}
  Problem: {multi_letter | lowercase}
  Current: {current_param}
  Should be: {G | K | V | T}
  Severity: LOW

═══════════════════════════════════════════════════════════════
SCAN 8: CONTRACT TAG NAMES
Convention: descriptive_tag: expression
═══════════════════════════════════════════════════════════════

For each contract clause, check:
□ Preconditions: X_valid, X_not_empty, X_positive, has_X, not_X
□ Postconditions: X_set, X_increased, result_X, X_unchanged
□ Invariants: X_valid, X_consistent

VIOLATIONS:

CONTRACT TAG VIOLATION: {class}.{feature} {require|ensure|invariant}
  File: {file_path}:{line}
  Problem: {missing_tag | non_descriptive | wrong_pattern}
  Current: {current_tag}: {expression}
  Should be: {suggested_tag}: {expression}
  Severity: LOW

═══════════════════════════════════════════════════════════════
SCAN 9: TUPLE LABEL NAMES
Convention: Avoid TUPLE feature names (count, is_equal, etc.)
═══════════════════════════════════════════════════════════════

For each TUPLE type with labels, check:
□ No label named: count, is_equal, default, generating_type, generator

VIOLATIONS:

TUPLE LABEL VIOLATION: {class}.{feature}
  File: {file_path}:{line}
  Problem: shadows TUPLE.{label}
  Current: TUPLE [{label}: {type}; ...]
  Should be: TUPLE [{alternative}: {type}; ...]
  Severity: CRITICAL

═══════════════════════════════════════════════════════════════
SCAN 10: FEATURE CLAUSE LABELS
Convention: Standard EiffelBase labels
═══════════════════════════════════════════════════════════════

For each feature clause, check:
□ Uses standard labels: Initialization, Access, Measurement,
  Status report, Element change, Removal, Conversion, Implementation

VIOLATIONS:

CLAUSE LABEL VIOLATION: {class}
  File: {file_path}:{line}
  Problem: {non_standard_label}
  Current: feature -- {current_label}
  Should be: feature -- {standard_label}
  Severity: LOW

═══════════════════════════════════════════════════════════════
OUTPUT: VIOLATION SUMMARY
═══════════════════════════════════════════════════════════════

# NAMING VIOLATION SCAN: {library_name}

## Summary
| Category | Critical | High | Medium | Low | Total |
|----------|----------|------|--------|-----|-------|
| Class names | {n} | {n} | {n} | {n} | {n} |
| Feature names | {n} | {n} | {n} | {n} | {n} |
| Constant names | {n} | {n} | {n} | {n} | {n} |
| Argument names | {n} | {n} | {n} | {n} | {n} |
| Local names | {n} | {n} | {n} | {n} | {n} |
| Cursor names | {n} | {n} | {n} | {n} | {n} |
| Generic params | {n} | {n} | {n} | {n} | {n} |
| Contract tags | {n} | {n} | {n} | {n} | {n} |
| Tuple labels | {n} | {n} | {n} | {n} | {n} |
| Clause labels | {n} | {n} | {n} | {n} | {n} |
| **TOTAL** | {n} | {n} | {n} | {n} | {n} |

## Critical Violations (Must Fix)
{List all CRITICAL violations - these cause compile errors}

## High Violations
{List all HIGH violations}

## All Violations by File
{Group violations by file for easier fixing}
```

## Success Criteria
- All 10 naming categories scanned
- Every violation has severity assigned
- Violations grouped by file for fixing
- Critical violations highlighted

## Next Step
→ N02-CLASSIFY-ISSUES.md
