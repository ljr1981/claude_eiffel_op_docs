# N02: Classify Naming Issues

## Context
Phase: **DETECTION** (2 of 2)
Purpose: Group violations by type, prioritize by impact

## Mindset

**Fix critical issues first. Group related changes together.**

Renaming is invasive. Plan the changes to minimize churn and maximize compilation success at each step.

## Input Required
- Violation scan from N01

## Prompt

```
TASK: Classify and prioritize naming violations for systematic fixing.

INPUT:
{violation_scan_from_N01}

═══════════════════════════════════════════════════════════════
STEP 1: IDENTIFY CRITICAL BLOCKERS
These MUST be fixed first - they cause compile errors
═══════════════════════════════════════════════════════════════

CRITICAL BLOCKERS:

1. VOIT(2) CONFLICTS (loop cursors shadowing features)
   {list each cursor that shadows a feature}

   CURSOR SHADOW: {class}.{feature}
     Cursor: {cursor_name}
     Shadows: {feature_name}
     Fix: Rename cursor to ic_{suggestion}

2. TUPLE LABEL CONFLICTS
   {list each tuple label that shadows TUPLE features}

   TUPLE SHADOW: {class}.{feature}
     Label: {label_name}
     Shadows: TUPLE.{feature}
     Fix: Rename label to {suggestion}

3. LOCAL/ATTRIBUTE CONFLICTS
   {list each local that shadows an attribute}

   LOCAL SHADOW: {class}.{feature}
     Local: {local_name}
     Shadows: {attribute_name}
     Fix: Rename local to l_{suggestion}

CRITICAL COUNT: {n}
ESTIMATED FIX TIME: {estimate based on count}

═══════════════════════════════════════════════════════════════
STEP 2: GROUP BY RENAME SCOPE
Determine ripple effect of each rename
═══════════════════════════════════════════════════════════════

SCOPE ANALYSIS:

1. CLASS RENAMES (affect entire codebase)

   CLASS RENAME: {old_name} → {new_name}
     Files to update: {count}
     ECF updates needed: {yes/no}
     External references: {list any external dependencies}
     Risk: {HIGH - class renames are invasive}

2. PUBLIC FEATURE RENAMES (affect clients)

   FEATURE RENAME: {class}.{old_name} → {new_name}
     Callers in this library: {count}
     External clients possible: {yes/no}
     Risk: {MEDIUM - may break external code}

3. PRIVATE FEATURE RENAMES (local impact only)

   PRIVATE RENAME: {class}.{old_name} → {new_name}
     Internal callers: {count}
     Risk: LOW

4. LOCAL/ARGUMENT RENAMES (single feature impact)

   LOCAL RENAME: {class}.{feature}.{old_name} → {new_name}
     Risk: MINIMAL

═══════════════════════════════════════════════════════════════
STEP 3: IDENTIFY RENAME CHAINS
Some renames require coordinated changes
═══════════════════════════════════════════════════════════════

RENAME CHAINS:

CHAIN 1: {description}
  1. Rename {A} to {A'}
  2. Rename {B} to {B'} (depends on A rename)
  3. ...

  Order matters: {yes/no}
  Reason: {why order matters}

═══════════════════════════════════════════════════════════════
STEP 4: CALCULATE FIX ORDER
Prioritize to maintain compilability
═══════════════════════════════════════════════════════════════

RECOMMENDED FIX ORDER:

Phase 1: CRITICAL (must compile after)
  □ {list critical fixes in order}
  CHECKPOINT: Compile to verify

Phase 2: CLASS NAMES (high impact, do early)
  □ {list class renames}
  CHECKPOINT: Compile to verify

Phase 3: PUBLIC FEATURES (medium impact)
  □ {list public feature renames}
  CHECKPOINT: Compile to verify

Phase 4: PRIVATE FEATURES (low impact)
  □ {list private feature renames}

Phase 5: ARGUMENTS (minimal impact)
  □ {list argument renames}

Phase 6: LOCALS/CURSORS (minimal impact)
  □ {list local/cursor renames}

Phase 7: CONTRACT TAGS (no compile impact)
  □ {list contract tag fixes}

Phase 8: CLAUSE LABELS (no compile impact)
  □ {list clause label fixes}

═══════════════════════════════════════════════════════════════
STEP 5: IDENTIFY SAFE BATCH OPERATIONS
Some fixes can be done with find/replace
═══════════════════════════════════════════════════════════════

BATCH OPERATIONS:

BATCH 1: Add a_ prefix to all arguments
  Pattern: Find arguments without a_ prefix
  Files affected: {count}
  Safe for batch: {yes/no}
  Reason: {why safe or not}

BATCH 2: Add ic_ prefix to all cursors
  Pattern: across X as {cursor} → across X as ic_{cursor}
  Files affected: {count}
  Safe for batch: {yes - cursor scope is local}

BATCH 3: {other batch operations}

═══════════════════════════════════════════════════════════════
OUTPUT: CLASSIFICATION REPORT
═══════════════════════════════════════════════════════════════

# NAMING VIOLATION CLASSIFICATION: {library_name}

## Priority Summary
| Priority | Count | Compile Impact |
|----------|-------|----------------|
| Critical | {n} | Blocks compilation |
| Class Renames | {n} | Requires ECF update |
| Public Features | {n} | May break clients |
| Private Features | {n} | Internal only |
| Arguments | {n} | None |
| Locals/Cursors | {n} | None |
| Contract Tags | {n} | None |
| Clause Labels | {n} | None |

## Fix Phases
{Ordered list of fix phases with checkpoints}

## Batch Operations Available
{List of safe batch operations}

## Estimated Effort
- Critical fixes: {estimate}
- Full cleanup: {estimate}
- Recommended approach: {phased vs all-at-once}
```

## Success Criteria
- All violations classified by scope
- Fix order determined
- Compile checkpoints identified
- Batch operations identified

## Next Step
→ N03-FIX-CLASSES.md
