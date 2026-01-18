# 07: Compile and Fix Errors

## Context
Working Hat: **VERIFICATION** (Phase 3 of 4)

## Input Required
- Implemented classes from 06-IMPLEMENT-FEATURES.md
- Compiler error output

## Prompt

```
TASK: Analyze compilation errors and fix WITHOUT changing contracts.

INPUT:
{compiler_errors}
{source_code}

ERROR CATEGORIES AND FIXES:

1. VUAR - Type mismatch
   Error: VUAR(1) ... Type 'X' does not conform to 'Y'
   Fix: Check assignment compatibility, add conversion or fix type

2. VEVI - Void safety violation
   Error: VEVI ... may be Void
   Fix: Add 'attached x as l_x then' guard OR fix type to attached

3. VEEN - Unknown identifier
   Error: VEEN ... unknown identifier 'x'
   Fix: Declare in 'local' block OR check spelling OR add to class

4. VKCN - Procedure used as function
   Error: VKCN(1) ... used as instruction
   Fix: Capture return value: l_temp := function_call

5. VAPE - Precondition uses non-exported feature
   Error: VAPE ... not exported
   Fix: Export the query OR remove from precondition

6. SYNTAX - Parse error
   Fix: Check for missing 'end', semicolons, keyword spelling

FIX RULES:
- NEVER modify contracts to fix errors
- If contracts are unfixable, report back (specification issue)
- Fix implementation to satisfy existing contracts
- Add local variables as needed
- Use index-based loops if 'across' causes issues

OUTPUT FORMAT:
ERROR: [exact error]
LOCATION: [file:line]
CAUSE: [brief explanation]
FIX: [code change]
```

## Success Criteria
- All compilation errors resolved
- Zero contract modifications
- Code compiles cleanly

## Next Step
→ 08-RUN-CONTRACTS.md
