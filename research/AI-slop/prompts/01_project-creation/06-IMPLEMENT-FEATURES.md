# 06: Implement Feature Bodies

## Context
Working Hat: **IMPLEMENTATION** (Phase 2 of 4)

## Input Required
- Complete specification from 05-WRITE-INVARIANTS.md

## Prompt

```
TASK: Implement feature bodies that satisfy all contracts.

INPUT:
{complete_specification}

IMPLEMENTATION RULES:
1. SATISFY POSTCONDITIONS
   - Each ensure clause MUST be true when feature exits
   - Work backward: what makes the postcondition true?

2. ASSUME PRECONDITIONS
   - Don't re-check require clauses in the body
   - Preconditions are GUARANTEED true on entry

3. MAINTAIN INVARIANTS
   - Class invariant must hold when feature exits
   - Temporary violations OK mid-feature

4. VOID SAFETY
   - Use 'attached x as l_x' for detachable types
   - Use 'check attached x as l_x end' if compiler can't prove

5. ITERATION PATTERN
   from i := 1 until i > collection.count loop
       -- use collection[i]
       i := i + 1
   end

6. CREATION PATTERN
   create result.make (args)
   -- NOT: result := create {TYPE}.make (args)

OUTPUT FORMAT:
feature_name (arg: TYPE): RESULT_TYPE
    require
        -- unchanged
    local
        l_var: TYPE
    do
        -- implementation here
        Result := computed_value
    ensure
        -- unchanged
    end

FORBIDDEN:
- Adding/removing/changing contracts
- Defensive checks that duplicate preconditions
- Comments explaining obvious code
- Multiple return points (use Result :=)
```

## Success Criteria
- All postconditions satisfiable by implementation
- No contract modifications
- Void-safe patterns used
- Single assignment to Result per path

## Next Step
→ 07-COMPILE-CHECK.md (HAT CHANGE: Verification)
