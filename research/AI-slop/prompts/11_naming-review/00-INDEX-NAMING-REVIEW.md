# Naming Review Workflow: Detecting and Fixing Eiffel Naming Violations

## CRITICAL: ANTI-SLOP ENFORCEMENT

**Before using this workflow, read: `../00_ANTI-SLOP-RULES.md`**

This workflow requires ACTUAL:
- File reading (Read tool with actual file paths)
- Pattern matching against ACTUAL code
- Renaming with compilation verification

**FORBIDDEN**:
- Claiming naming violations without reading actual code
- Suggesting renames without implementing them
- Claiming "fixed" without compile verification

---

## Overview

8 sequential prompts for identifying and fixing Eiffel naming convention violations. Critical for importing legacy code (like Eiffel Loop) into the simple_* ecosystem.

## Reference Document

**All naming rules come from:**
```
D:\prod\reference_docs\claude\NAMING_CONVENTIONS.md
```

Consult this document for complete rules and examples.

## The Problem

Naming violations cause:
- **Compile errors**: VOIT(2) conflicts, VUCR errors
- **Readability issues**: CamelCase in Eiffel looks wrong
- **Maintenance burden**: Inconsistent naming confuses developers
- **Tool failures**: EiffelStudio expects standard names

**Consistent naming is the foundation of readable Eiffel code.**

## Naming Convention Quick Reference

| Element | Convention | Example |
|---------|------------|---------|
| Class | ALL_CAPS with underscores | `LINKED_LIST`, `PRICING_MODEL` |
| Feature | all_lowercase with underscores | `set_owner`, `yearly_premium` |
| Constant | Initial_cap with underscores | `Pi`, `Welcome_message` |
| Generic parameter | Single uppercase letter | `G`, `K`, `V` |
| Local variable | lowercase, abbreviated OK | `i`, `j`, `k`, `c`, `n` (loop counters OK) |
| Argument | `a_` prefix + lowercase | `a_name`, `a_count` |
| Loop cursor | `ic_` prefix | `ic`, `ic_item` |
| Boolean query | `is_` or `has_` prefix | `is_empty`, `has_key` |
| Command | verb (imperative) | `move`, `deposit`, `set_name` |
| Query | noun | `count`, `item`, `balance` |
| Creation | `make` prefix | `make`, `make_from_string` |
| Conversion | `to_` or `as_` prefix | `to_string`, `as_lower` |

## Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    DETECTION PHASE                              │
│              (Find naming problems)                             │
├─────────────────────────────────────────────────────────────────┤
│  N01-SCAN-VIOLATIONS       Find all naming violations          │
│           ↓                                                     │
│  N02-CLASSIFY-ISSUES       Group and prioritize                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    FIX PHASE                                    │
│              (Rename systematically)                            │
├─────────────────────────────────────────────────────────────────┤
│  N03-FIX-CLASSES           Fix class names                     │
│           ↓                                                     │
│  N04-FIX-FEATURES          Fix feature names                   │
│           ↓                                                     │
│  N05-FIX-ARGUMENTS         Fix arguments and locals            │
│           ↓                                                     │
│  N06-FIX-CONTRACTS         Fix contract tag names              │
│           ↓                                                     │
│  N07-FIX-MAGIC-NUMBERS     Replace literals with constants     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    VERIFICATION PHASE                           │
├─────────────────────────────────────────────────────────────────┤
│  N08-VERIFY-NAMING         Compile and validate                │
└─────────────────────────────────────────────────────────────────┘
```

## Prompt Files

| # | File | Phase | Purpose |
|---|------|-------|---------|
| N01 | SCAN-VIOLATIONS | Detection | Find all naming violations |
| N02 | CLASSIFY-ISSUES | Detection | Group by type and severity |
| N03 | FIX-CLASSES | Fix | Rename classes to ALL_CAPS |
| N04 | FIX-FEATURES | Fix | Rename features to snake_case |
| N05 | FIX-ARGUMENTS | Fix | Fix a_ prefix, ic_ cursors |
| N06 | FIX-CONTRACTS | Fix | Standardize contract tags |
| N07 | FIX-MAGIC-NUMBERS | Fix | Replace literals with named constants |
| N08 | VERIFY-NAMING | Verification | Compile and validate |

## Violation Severity Levels

| Severity | Description | Example |
|----------|-------------|---------|
| **CRITICAL** | Causes compile errors | Loop cursor shadows feature (VOIT(2)) |
| **HIGH** | Violates core conventions | CamelCase class name |
| **MEDIUM** | Inconsistent with ecosystem | Missing `a_` on arguments |
| **LOW** | Style preference | Abbreviated local variable |

## Common Violation Patterns

### CamelCase (Java/C# style)
```eiffel
-- WRONG
LinkedList
setOwner
yearlyPremium

-- RIGHT
LINKED_LIST
set_owner
yearly_premium
```

### Missing Argument Prefix
```eiffel
-- WRONG
set_name (name: STRING)

-- RIGHT
set_name (a_name: STRING)
```

### Loop Cursor Conflicts (VOIT(2))
```eiffel
-- WRONG: 'item' may conflict with a feature
across list as item loop ...

-- RIGHT
across list as ic loop ...
```

**Note**: Short names like `i`, `j`, `k`, `c` are acceptable for from-loop counters:
```eiffel
-- OK: Conventional from-loop counter
from i := 1 until i > count loop
    process (i)
    i := i + 1
end

-- NEEDS FIX: Across-loop cursor should use ic_ prefix
across list as ic_item loop  -- not 'item' or 'x'
    process (ic_item)
end
```

### Wrong Query/Command Naming
```eiffel
-- WRONG: Query should be noun, not verb
get_count: INTEGER

-- RIGHT
count: INTEGER
```

### Magic Numbers
```eiffel
-- WRONG: What does 3 mean? What is 4096?
context_size := 3
create buffer.make (4096)

-- RIGHT: Named constants explain meaning
context_size := Default_context_lines
create buffer.make (Default_buffer_size)

feature {NONE} -- Constants

    Default_context_lines: INTEGER = 3
    Default_buffer_size: INTEGER = 4096
```

## Output Format

The workflow produces:

```
naming-review/
├── VIOLATION-SCAN.md         # All detected violations
├── CLASSIFICATION.md         # Grouped and prioritized
├── CLASS-RENAMES.md          # Class name changes
├── FEATURE-RENAMES.md        # Feature name changes
├── ARGUMENT-FIXES.md         # Argument/local fixes
├── CONTRACT-FIXES.md         # Contract tag fixes
├── MAGIC-NUMBER-FIXES.md     # Literal → constant replacements
└── VERIFICATION.md           # Compile results
```

## When to Use This Workflow

1. **Importing legacy code** - Eiffel Loop, old projects
2. **After code generation** - AI-generated code may have bad naming
3. **Before release** - Final polish
4. **Code review** - Systematic naming audit
5. **Onboarding** - Understand naming decisions

## Integration with Other Workflows

- Run **AFTER** void-safety fixes (code must compile first)
- Run **BEFORE** design-audit (clean naming aids analysis)
- Run **BEFORE** github_prep (public code should be clean)

## Metrics Tracked

- Total violations found
- Violations by category
- Critical violations fixed
- Compile success after fixes
