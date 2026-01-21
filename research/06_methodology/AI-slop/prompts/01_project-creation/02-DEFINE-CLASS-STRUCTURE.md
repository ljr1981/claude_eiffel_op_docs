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

feature -- Model (MML specification)
    model: MML_{TYPE} [{PARAMS}]
        -- Abstract mathematical model of this class's state
        -- TYPE: SEQUENCE (ordered), SET (unique), BAG (counted), MAP (key-value)

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

MODEL QUERY DECISION:
For classes that manage collections/state, define a model query:
| Class Type | Model Type | Example |
|------------|------------|---------|
| Ordered list | MML_SEQUENCE [G] | Stack, Queue, List |
| Unique items | MML_SET [G] | Tags, IDs |
| Counted items | MML_BAG [G] | Inventory, Word counts |
| Key-value | MML_MAP [K, V] | Dictionary, Cache |
| Multi-valued | MML_RELATION [K, V] | Graph edges |

RULES:
- NO feature bodies (no 'do' blocks)
- NO contracts yet (no require/ensure/invariant)
- ONLY signatures and one-line comments
- Group features by category (Model, Access, Status, Element Change, etc.)
- Use Eiffel naming: is_* for booleans, *_count for integers
- Commands return nothing; Queries have return types
- **ADD model query for any class managing state/collections**
```

## Success Criteria
- All classes from analysis represented
- All features have signatures
- Command-Query Separation enforced
- Proper feature categorization

## Next Step
→ 03-WRITE-PRECONDITIONS.md
