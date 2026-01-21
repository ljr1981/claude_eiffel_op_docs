# N03: Fix Class Names

## Context
Phase: **FIX** (1 of 4)
Purpose: Rename classes to follow ALL_CAPS convention

## Mindset

**Class renames are the most invasive. Do them first, do them right.**

A class rename affects:
- The class file itself
- The filename
- All files that reference the class
- ECF configuration
- Any external documentation

## Input Required
- Classification report from N02
- List of class names to fix

## Prompt

```
TASK: Rename classes to follow Eiffel naming conventions.

INPUT:
{classification_from_N02}
Classes to rename:
{list of class renames with old → new names}

═══════════════════════════════════════════════════════════════
PRE-FLIGHT CHECKS
═══════════════════════════════════════════════════════════════

Before renaming, verify:

□ Library compiles in current state
  Command: /d/prod/ec.sh -batch -config {lib}.ecf -target {lib} -c_compile
  Result: {PASS | FAIL - fix first}

□ No uncommitted changes
  Command: git status
  Result: {clean | dirty - commit first}

□ New names don't conflict with existing classes
  For each new name, search: grep -r "class {NEW_NAME}" .
  Conflicts: {none | list conflicts}

═══════════════════════════════════════════════════════════════
FOR EACH CLASS RENAME
═══════════════════════════════════════════════════════════════

CLASS RENAME: {OldName} → {NEW_NAME}

STEP 1: Find all references
  grep -rn "{OldName}" --include="*.e" --include="*.ecf"

  References found:
  - {file}:{line}: {context}
  - ...

STEP 2: Rename the class declaration
  File: {class_file}

  BEFORE:
  class
      {OldName}

  AFTER:
  class
      {NEW_NAME}

STEP 3: Update all references in .e files
  For each file with references:

  File: {file}
  Lines: {line numbers}

  Changes:
  - Line {n}: {old context} → {new context}
  - ...

STEP 4: Update ECF if class is explicitly listed
  File: {lib}.ecf

  Check for:
  - <class name="{OldName}"/>
  - <class_option class="{OldName}">

  Changes needed: {yes/no}

STEP 5: Rename the file (if filename doesn't match)
  Current: {old_filename}.e
  New: {new_filename}.e

  Command: git mv {old_filename}.e {new_filename}.e

STEP 6: Verify compilation
  Command: /d/prod/ec.sh -batch -config {lib}.ecf -target {lib} -c_compile
  Result: {PASS | FAIL}

  If FAIL:
  - Error: {error message}
  - Fix: {what to fix}

═══════════════════════════════════════════════════════════════
CLASS RENAME PATTERNS
═══════════════════════════════════════════════════════════════

Pattern 1: CamelCase → ALL_CAPS

  LinkedList → LINKED_LIST
  HttpRequest → HTTP_REQUEST
  JsonParser → JSON_PARSER
  MyClass → MY_CLASS

Pattern 2: lowercase → ALL_CAPS

  account → ACCOUNT
  order_item → ORDER_ITEM

Pattern 3: Abbreviation Expansion

  ACCT → ACCOUNT (if not standard abbreviation)
  HTTP → HTTP (keep - standard abbreviation)
  JSON → JSON (keep - standard abbreviation)

Pattern 4: Remove Redundant Prefix

  EL_STRING_HANDLER → STRING_HANDLER (if EL_ is library prefix)

  Note: Only remove if prefix adds no semantic value
  Keep if: Distinguishes from system class (EL_STRING vs STRING)

═══════════════════════════════════════════════════════════════
HANDLING EXTERNAL DEPENDENCIES
═══════════════════════════════════════════════════════════════

If this library is used by other libraries:

1. Document the rename
   Add to CHANGELOG.md:

   ## Breaking Changes
   - Renamed `{OldName}` to `{NEW_NAME}`

2. Consider migration period
   Option A: Clean break (recommended for internal code)
   Option B: Add rename clause (for external clients)

   class NEW_NAME
   obsolete "Renamed from OldName"
   ...

═══════════════════════════════════════════════════════════════
OUTPUT: CLASS RENAME LOG
═══════════════════════════════════════════════════════════════

# CLASS RENAMES: {library_name}

## Renames Applied
| Old Name | New Name | Files Changed | Status |
|----------|----------|---------------|--------|
| {old} | {new} | {n} | {DONE | FAILED} |
| ... | ... | ... | ... |

## Compilation Status
- Before renames: {PASS/FAIL}
- After renames: {PASS/FAIL}

## Files Modified
{List of all files changed with line counts}

## ECF Changes
{Any ECF modifications made}

## Next Steps
{Any follow-up needed}
```

## Success Criteria
- All class names follow ALL_CAPS convention
- All references updated
- Files renamed to match class names
- ECF updated if needed
- Library compiles after changes

## Next Step
→ N04-FIX-FEATURES.md
