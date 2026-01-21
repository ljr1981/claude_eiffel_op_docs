# M07: Compile and Validate Changes

## Context
Phase: **VALIDATE** (3 of 3)
Scenario: Pre-existing codebase, contracts added

## Input Required
- Modified source code with new contracts
- Compiler output

## Prompt

```
TASK: Compile modified code and resolve any errors from contract additions.

INPUT:
{modified_source_code}
{compiler_output}

EXPECTED ERROR TYPES FROM CONTRACT ADDITIONS:

1. VAPE - Contract references non-exported feature
   Cause: Precondition uses private query
   Fix: Export the query to {ANY} or appropriate clients

2. SYNTAX - Malformed contract
   Cause: Typo in contract clause
   Fix: Correct syntax

3. VEEN - Unknown identifier in contract
   Cause: Referenced non-existent feature
   Fix: Add the query or fix name

4. TYPE MISMATCH - Contract expression not BOOLEAN
   Cause: Expression doesn't evaluate to boolean
   Fix: Correct expression

VALIDATION RULES:
- DO NOT remove contracts to fix errors
- Export queries if needed for preconditions
- Add helper queries if complex expressions needed
- If contract truly unfixable, flag for human review

OUTPUT FORMAT:
COMPILATION STATUS: [SUCCESS | FAILED]

If FAILED:
ERROR: {exact error message}
LOCATION: {file:line}
CAUSE: {why contract addition caused this}
FIX: {specific change}

CHANGES MADE:
- {file}: {description of fix}

CONTRACTS PRESERVED: [YES | NO - explain why not]

If SUCCESS:
All contract additions compile cleanly.
Ready for runtime validation.
```

## Success Criteria
- Code compiles with new contracts
- No contracts removed to achieve compilation
- All fixes maintain contract intent

## Next Step
→ M08-TEST-NEW-CONTRACTS.md
