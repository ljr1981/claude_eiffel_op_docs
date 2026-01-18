# 05: Write Class Invariants

## Context
Working Hat: **SPECIFICATION** (Phase 1 of 4)

## Input Required
- Complete class with pre/postconditions from 04-WRITE-POSTCONDITIONS.md

## Prompt

```
TASK: Add class invariants that must hold after any public feature.

INPUT:
{class_with_contracts}

OUTPUT FORMAT:
class CLASS_NAME
    -- features...

invariant
    tag_name: boolean_expression
    another_tag: another_expression

end

INVARIANT CATEGORIES - check each:
1. ATTRIBUTE VALIDITY
   - count_non_negative: count >= 0
   - capacity_sufficient: count <= capacity
   - name_not_empty: not name.is_empty

2. ATTRIBUTE RELATIONSHIPS
   - consistency: start_date <= end_date
   - derived_correct: full_name.same_string (first_name + " " + last_name)

3. COLLECTION INTEGRITY
   - no_void_items: across items as ic all ic /= Void end
   - sorted: across 1 |..| (count - 1) as i all items[i] <= items[i + 1] end

4. STATE MACHINE RULES
   - valid_state: state = Open or state = Closed
   - closed_final: is_closed implies count = old count

5. VOID SAFETY
   - attached_required: internal_data /= Void

RULES:
- Invariants checked after EVERY public feature call
- Only reference queries (no side effects)
- If invariant references 'old', it's wrong (use postconditions)
- Start minimal, add as bugs reveal missing invariants
- Tag names should read as assertions: "count_non_negative" NOT "check_count"
```

## Success Criteria
- Core data integrity rules captured
- No references to 'old' or 'Result'
- Only queries used in expressions
- Relationships between attributes validated

## Next Step
→ 06-IMPLEMENT-FEATURES.md (HAT CHANGE: Implementation)
