# N04: Fix Feature Names

## Context
Phase: **FIX** (2 of 4)
Purpose: Rename features to follow Eiffel conventions

## Mindset

**Features are the API. Names communicate intent.**

Commands should sound like orders (verbs). Queries should sound like things (nouns). Boolean queries should ask questions.

## Input Required
- Classification report from N02
- List of features to rename

## Prompt

```
TASK: Rename features to follow Eiffel naming conventions.

INPUT:
{classification_from_N02}
Features to rename:
{list of feature renames with class.old → new names}

═══════════════════════════════════════════════════════════════
FEATURE NAMING RULES
═══════════════════════════════════════════════════════════════

1. ALL FEATURES: all_lowercase with underscores
   WRONG: yearlyPremium, SetOwner, getCount
   RIGHT: yearly_premium, set_owner, count

2. COMMANDS (procedures): Use verbs (imperative/infinitive)
   WRONG: name_setter, addition
   RIGHT: set_name, add

3. QUERIES (functions/attributes): Use nouns
   WRONG: get_count, calculate_total
   RIGHT: count, total

4. BOOLEAN QUERIES: Use is_/has_/can_ prefix OR adjective
   WRONG: check_empty, empty_check
   RIGHT: is_empty, empty, has_items, can_extend

5. CREATION PROCEDURES: Use make prefix
   WRONG: create, initialize, new
   RIGHT: make, make_from_string, make_with_capacity

6. CONVERSION FEATURES: Use to_/as_ prefix
   WRONG: string, get_as_string
   RIGHT: to_string, as_lower

═══════════════════════════════════════════════════════════════
FOR EACH FEATURE RENAME
═══════════════════════════════════════════════════════════════

FEATURE RENAME: {Class}.{oldName} → {new_name}

STEP 1: Identify feature type
  Type: {COMMAND | QUERY | BOOLEAN_QUERY | CREATION | CONVERSION}
  Current name problem: {CamelCase | wrong_prefix | verb_for_query | etc.}

STEP 2: Find all references
  grep -rn "{oldName}" --include="*.e"

  References found:
  - {file}:{line}: {context} (caller)
  - {file}:{line}: {context} (declaration)
  - ...

STEP 3: Check for overrides/redefines
  Does this feature:
  □ Inherit from parent? Parent: {parent_class}.{feature}
  □ Get redefined in children? Children: {list}

  If inherited:
  - Must rename in parent first
  - Or use rename clause: rename {old} as {new}

  If redefined:
  - Must update all children

STEP 4: Update the declaration
  File: {file}
  Line: {line}

  BEFORE:
  {oldName}: {TYPE}
      -- {comment}
    do
      ...
    end

  AFTER:
  {new_name}: {TYPE}
      -- {comment}
    do
      ...
    end

STEP 5: Update all call sites
  For each caller:

  File: {file}
  Line: {line}

  BEFORE: {context with old name}
  AFTER: {context with new name}

STEP 6: Update any rename clauses
  If class uses rename:

  inherit
      PARENT
          rename
              {oldName} as something
          end

  Update to:
          rename
              {new_name} as something
          end

STEP 7: Verify compilation
  Command: /d/prod/ec.sh -batch -config {lib}.ecf -target {lib} -c_compile
  Result: {PASS | FAIL}

═══════════════════════════════════════════════════════════════
COMMON RENAME PATTERNS
═══════════════════════════════════════════════════════════════

Pattern 1: CamelCase → snake_case
  yearlyPremium → yearly_premium
  setOwner → set_owner
  getCount → count (also remove get_)

Pattern 2: Remove get_ prefix (queries don't need it)
  get_name → name
  get_balance → balance
  get_item_at → item_at

Pattern 3: Add is_/has_ to boolean queries
  empty → is_empty
  valid → is_valid
  contains → has (or keep contains if clearer)

Pattern 4: Verb → Noun for queries
  calculate_total → total
  compute_hash → hash_code
  retrieve_item → item

Pattern 5: Noun → Verb for commands
  addition → add
  removal → remove
  update_operation → update

Pattern 6: Standardize creation procedures
  create → make
  initialize → make
  new_instance → make
  create_from_string → make_from_string

Pattern 7: Standardize conversion features
  string → to_string
  as_string → to_string
  integer → to_integer
  lower → as_lower (returns same type)
  to_lower → as_lower (returns same type)

═══════════════════════════════════════════════════════════════
HANDLING EIFFELBASE CONVENTIONS
═══════════════════════════════════════════════════════════════

Use standard names when applicable:

| Standard Name | Purpose | Use Instead Of |
|---------------|---------|----------------|
| item | Access element | get_element, value |
| count | Number of elements | size, length |
| capacity | Maximum elements | max_size |
| is_empty | No elements | empty, has_no_items |
| is_full | At capacity | full |
| has (v) | Contains value | contains, includes |
| extend (v) | Add element | add, append, push |
| put (v) | Add/replace | set, store |
| remove | Remove current | delete |
| prune (v) | Remove specific | remove_item |
| wipe_out | Remove all | clear, reset |

═══════════════════════════════════════════════════════════════
OUTPUT: FEATURE RENAME LOG
═══════════════════════════════════════════════════════════════

# FEATURE RENAMES: {library_name}

## Renames Applied
| Class | Old Name | New Name | Type | Call Sites | Status |
|-------|----------|----------|------|------------|--------|
| {cls} | {old} | {new} | {type} | {n} | {DONE} |
| ... | ... | ... | ... | ... | ... |

## Compilation Status
- After renames: {PASS/FAIL}

## Inheritance Impacts
{Any rename clauses added or modified}

## Files Modified
{List of files with change counts}
```

## Success Criteria
- All features follow snake_case
- Commands use verbs
- Queries use nouns
- Boolean queries use is_/has_ or adjectives
- All call sites updated
- Library compiles

## Next Step
→ N05-FIX-ARGUMENTS.md
