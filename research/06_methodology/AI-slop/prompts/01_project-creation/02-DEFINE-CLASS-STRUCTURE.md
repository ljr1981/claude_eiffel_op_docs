# 02: Define Class Structure

## Context
Working Hat: **SPECIFICATION** (Phase 1 of 4)

## Input Required
- Output from 01-ANALYZE-REQUIREMENTS.md

## Prompt

```
TASK: Define Eiffel class skeleton with feature signatures only.

INPUT:
{analyzed_requirements}

OUTPUT FORMAT (per class):
class CLASS_NAME

inherit
    PARENT_IF_ANY

create
    creation_procedures_list

feature -- Access (Queries)
    query_name: RETURN_TYPE
        -- Brief description

feature -- Status (Boolean Queries)
    is_condition: BOOLEAN
        -- Brief description

feature -- Measurement (Integer/Numeric Queries)
    count_of_something: INTEGER
        -- Brief description

feature -- Element Change (Commands)
    command_name (args: TYPES)
        -- Brief description

end

RULES:
- NO feature bodies (no 'do' blocks)
- NO contracts yet (no require/ensure/invariant)
- ONLY signatures and one-line comments
- Group features by category (Access, Status, Element Change, etc.)
- Use Eiffel naming: is_* for booleans, *_count for integers
- Commands return nothing; Queries have return types
```

## Success Criteria
- All classes from analysis represented
- All features have signatures
- Command-Query Separation enforced
- Proper feature categorization

## Next Step
→ 03-WRITE-PRECONDITIONS.md
