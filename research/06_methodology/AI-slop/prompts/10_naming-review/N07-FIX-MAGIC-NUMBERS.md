# N07: Fix Magic Numbers

## Context
Phase: **FIX** (5 of 5)
Purpose: Replace magic numbers/strings/characters with semantically named constants

## Mindset

**Magic numbers destroy comprehension. Named constants enable understanding.**

A "magic" value is any literal (number, string, character) that lacks semantic meaning when read in context. The reader must guess or search to understand what `3`, `100`, or `"UTF-8"` means in this specific context.

The fix: Replace with a named constant that communicates intent.

## The Three Scopes

Magic number replacement has three scope levels, chosen based on where the value is needed:

### Scope 1: System-Wide Constants

**Use when**: The same semantic value is needed across multiple classes.

**Pattern**: Create a constants class that other classes inherit from.

```eiffel
deferred class
    APPLICATION_CONSTANTS

feature -- Limits

    Maximum_retry_attempts: INTEGER = 3
            -- Maximum times to retry failed operations

    Default_timeout_seconds: INTEGER = 30
            -- Default network timeout in seconds

    Maximum_file_size_bytes: INTEGER = 10_485_760
            -- Maximum file size (10 MB)

feature -- Status codes

    Status_success: INTEGER = 0
    Status_error: INTEGER = 1
    Status_pending: INTEGER = 2

end
```

**Usage**:
```eiffel
class MY_CLASS

inherit
    APPLICATION_CONSTANTS

feature -- Implementation

    retry_operation
        local
            l_attempts: INTEGER
        do
            from l_attempts := 0 until l_attempts >= Maximum_retry_attempts loop
                -- try operation
                l_attempts := l_attempts + 1
            end
        end
```

### Scope 2: Class-Scoped Constants

**Use when**: The value is only meaningful within one class.

**Pattern**: Place constants in a feature clause at the bottom of the class.

```eiffel
class DIFF_ENGINE

feature -- Computation

    compute_diff: DIFF_RESULT
        do
            if context_size = 0 then
                context_size := Default_context_size
            end
            -- ...
        end

feature {NONE} -- Constants

    Default_context_size: INTEGER = 3
            -- Default number of context lines in diff output

    Op_match: INTEGER = 0
            -- LCS operation: lines match
    Op_insert: INTEGER = 1
            -- LCS operation: line inserted
    Op_delete: INTEGER = 2
            -- LCS operation: line deleted

end
```

### Scope 3: Feature-Scoped Constants

**Use when**: The value is only meaningful within a single feature.

**Pattern**: Use a local variable with a descriptive name.

```eiffel
parse_header (a_line: STRING): TUPLE [start: INTEGER; count: INTEGER]
    local
        Marker_source: CHARACTER  -- '-' marks source line range
        Marker_target: CHARACTER  -- '+' marks target line range
    do
        Marker_source := '-'
        Marker_target := '+'
        -- parse logic using named markers instead of '-' and '+'
    end
```

**Note**: For truly single-use values, a well-named local is often clearer than a class constant that's only used once.

## Input Required
- All source files
- Understanding of domain semantics

## Prompt

```
TASK: Identify and replace magic numbers with named constants.

INPUT:
Library: {library_path}
Source files: {list of .e files}

═══════════════════════════════════════════════════════════════
SCAN 1: NUMERIC MAGIC NUMBERS
═══════════════════════════════════════════════════════════════

For each numeric literal in code, ask:
□ Does the reader immediately understand what this number means?
□ Is this a domain-specific value (not just 0, 1, or array indices)?
□ Is this number repeated in multiple places?
□ Would changing this value require finding all occurrences?

MAGIC NUMBER CANDIDATE: {class}.{feature}:{line}
  Literal: {number}
  Context: {surrounding code}
  Semantic meaning: {what does this represent?}
  Repeated: {yes/no - how many times?}
  Scope needed: {SYSTEM | CLASS | FEATURE}
  Suggested name: {Semantic_constant_name}

═══════════════════════════════════════════════════════════════
SCAN 2: STRING MAGIC VALUES
═══════════════════════════════════════════════════════════════

For each string literal, ask:
□ Is this a configuration value that might change?
□ Is this repeated across the codebase?
□ Does the string encode a semantic concept (status, type, mode)?

MAGIC STRING CANDIDATE: {class}.{feature}:{line}
  Literal: "{string}"
  Context: {surrounding code}
  Semantic meaning: {what does this represent?}
  Scope needed: {SYSTEM | CLASS | FEATURE}
  Suggested name: {Semantic_constant_name}

═══════════════════════════════════════════════════════════════
SCAN 3: CHARACTER MAGIC VALUES
═══════════════════════════════════════════════════════════════

For each character literal, ask:
□ Does the character have domain meaning beyond its ASCII value?
□ Would a reader understand why this specific character is used?

MAGIC CHARACTER CANDIDATE: {class}.{feature}:{line}
  Literal: '{char}'
  Context: {surrounding code}
  Semantic meaning: {what does this represent?}
  Scope needed: {SYSTEM | CLASS | FEATURE}
  Suggested name: {Semantic_constant_name}

═══════════════════════════════════════════════════════════════
EXCEPTIONS: NOT MAGIC
═══════════════════════════════════════════════════════════════

Some literals are NOT magic numbers:

✓ Array/loop bounds: 0, 1 for start/end
✓ Increment/decrement: i + 1, count - 1
✓ Empty/initial: "", 0, False for initialization
✓ Boolean construction: True, False
✓ Format strings: "%N", "%T" (Eiffel newline/tab)
✓ Standard separators: ",", ":", "/" when parsing
✓ Well-known paths: "/" for root, ".." for parent

Ask: "Would ANY competent Eiffel developer understand this?"
If yes, it's not magic.

═══════════════════════════════════════════════════════════════
DETERMINE SCOPE FOR EACH FIX
═══════════════════════════════════════════════════════════════

For each magic value identified:

1. Is it used in multiple classes?
   → SYSTEM scope: Create or use constants class

2. Is it used only in one class but multiple features?
   → CLASS scope: Add to feature {NONE} -- Constants

3. Is it used only in one feature?
   → FEATURE scope: Use named local variable

═══════════════════════════════════════════════════════════════
APPLY FIXES BY SCOPE
═══════════════════════════════════════════════════════════════

SYSTEM-SCOPE FIXES:

1. Create constants class if none exists:
   File: {lib}_constants.e

   deferred class
       {LIB}_CONSTANTS

   feature -- {Category}

       {Constant_name}: {TYPE} = {value}
               -- {Description}

   end

2. Add inheritance to classes that need it:
   inherit
       {LIB}_CONSTANTS

3. Replace literals with constant references

CLASS-SCOPE FIXES:

1. Add constants section at end of class:
   feature {NONE} -- Constants

       {Constant_name}: {TYPE} = {value}
               -- {Description}

2. Replace literals with constant references

FEATURE-SCOPE FIXES:

1. Add named local:
   local
       {Constant_name}: {TYPE}
   do
       {Constant_name} := {value}

2. Replace literal with local reference

═══════════════════════════════════════════════════════════════
NAMING PATTERNS FOR CONSTANTS
═══════════════════════════════════════════════════════════════

Constants follow Initial_cap convention:

| Type | Pattern | Example |
|------|---------|---------|
| Limits | Maximum_X, Minimum_X | Maximum_retries |
| Defaults | Default_X | Default_timeout |
| Sizes | X_size, X_count | Buffer_size |
| Indices | X_index | Header_index |
| Status | Status_X | Status_success |
| Modes | Mode_X | Mode_verbose |
| Characters | X_char, X_marker | Separator_char |
| Strings | X_prefix, X_suffix | Error_prefix |

═══════════════════════════════════════════════════════════════
OUTPUT: MAGIC NUMBER FIX LOG
═══════════════════════════════════════════════════════════════

# MAGIC NUMBER FIXES: {library_name}

## Summary
| Scope | Count |
|-------|-------|
| System-wide | {n} |
| Class-scoped | {n} |
| Feature-scoped | {n} |
| **Total** | {n} |

## System-Wide Constants Added
| Constant | Value | Used In |
|----------|-------|---------|
| {name} | {value} | {classes} |

## Class-Scoped Constants Added
| Class | Constant | Value | Meaning |
|-------|----------|-------|---------|
| {class} | {name} | {value} | {meaning} |

## Feature-Scoped Locals Added
| Class.Feature | Local | Value | Meaning |
|---------------|-------|-------|---------|
| {class.feat} | {name} | {value} | {meaning} |

## Literals Replaced
| Location | Before | After |
|----------|--------|-------|
| {class}:{line} | {literal} | {constant} |
```

## Common Examples

### Before: Magic Numbers
```eiffel
set_defaults
    do
        timeout := 30
        retries := 3
        buffer_size := 4096
    end

parse_line (a_line: STRING)
    do
        if a_line.item (1) = '-' then
            -- source line
        elseif a_line.item (1) = '+' then
            -- target line
        end
    end
```

### After: Named Constants
```eiffel
set_defaults
    do
        timeout := Default_timeout_seconds
        retries := Maximum_retry_attempts
        buffer_size := Default_buffer_size
    end

parse_line (a_line: STRING)
    do
        if a_line.item (1) = Marker_deletion then
            -- source line
        elseif a_line.item (1) = Marker_addition then
            -- target line
        end
    end

feature {NONE} -- Constants

    Default_timeout_seconds: INTEGER = 30
    Maximum_retry_attempts: INTEGER = 3
    Default_buffer_size: INTEGER = 4096
    Marker_deletion: CHARACTER = '-'
    Marker_addition: CHARACTER = '+'
```

## Success Criteria
- All non-obvious literals identified
- Appropriate scope chosen for each
- Constants follow Initial_cap naming
- Each constant has header comment explaining meaning
- Library compiles after changes

## Next Step
→ N08-VERIFY-NAMING.md
