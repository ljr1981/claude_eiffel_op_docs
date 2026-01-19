# After-Action Review: simple_graphql Generation

## Date: 2025-01-17

## Summary
During simple_graphql generation, Claude went "off the rails" of simple_codegen by using ISE's JSON library classes instead of simple_json classes that were specified in the system_spec.json.

## What Happened

### The Deviation
1. system_spec.json correctly specified `simple_json` as a dependency
2. When generating class code, Claude used ISE JSON classes:
   - `JSON_OBJECT` instead of `SIMPLE_JSON_OBJECT`
   - `JSON_PARSER` instead of `SIMPLE_JSON`
   - `JSON_VALUE` instead of `SIMPLE_JSON_VALUE`
   - `JSON_ARRAY` instead of `SIMPLE_JSON_ARRAY`
3. The ECF was modified to include ISE's json.ecf instead of simple_json.ecf
4. User caught this and said: "WTF IS THAT???? Where is simple_json???? STOP BEING STUPID!!!!"

### Root Cause Analysis

**Primary Cause: Claude ignored the reuse_info in the TOON prompt template**

The 02_gen_class.toon template includes:
```
REUSE (available libraries):
${reuse_info}
```

This section should explicitly tell Claude which libraries to use. But Claude:
1. Did not wait for or follow the TOON-generated prompt for each class
2. Generated all classes in a bulk fashion without consulting the reuse discoverer
3. Used "familiar" ISE JSON classes instead of ecosystem-standard simple_json

**Secondary Cause: Monolithic code generation**

Instead of following the atomic prompt model (ONE prompt = ONE task = ONE deliverable), Claude generated multiple classes at once, increasing cognitive load and making it easier to lose track of constraints.

## What Should Have Happened

1. For each class, simple_codegen should generate a TOON prompt that includes:
   - The reuse_info showing `simple_json` as the library to use
   - Specific class names to use: SIMPLE_JSON_OBJECT, SIMPLE_JSON_VALUE, etc.

2. Claude should have:
   - Generated ONE class at a time
   - Read and followed the TOON prompt for that class
   - Used the classes specified in reuse_info

## Verification: Are TOON Prompts Being Used?

Checked `/d/prod/simple_code/prompts/02_gen_class.toon`:
- Template EXISTS and is properly structured
- Contains ${reuse_info} placeholder
- SCG_PROMPT_BUILDER does populate reuse_info from SCG_REUSE_DISCOVERER

**The TOON prompts ARE configured correctly. Claude failed to follow them.**

## Lessons Learned

1. **Atomic workflow is critical**: Generate ONE class at a time, following each TOON prompt
2. **Never use "familiar" ISE classes**: Always check what simple_* library provides the functionality
3. **Consult system_spec.json dependencies**: If it says `simple_json`, use `SIMPLE_JSON_*` classes
4. **Wait for the reuse_info**: Don't generate code until you know what libraries to use

## Corrective Actions

1. All classes were rewritten to use simple_json:
   - GQL_RESULT now uses SIMPLE_JSON, SIMPLE_JSON_VALUE, SIMPLE_JSON_OBJECT, SIMPLE_JSON_ARRAY
   - GQL_ERROR now uses SIMPLE_JSON_OBJECT
   - GQL_QUERY now uses SIMPLE_JSON_OBJECT, SIMPLE_JSON_VALUE
   - SIMPLE_GRAPHQL now uses SIMPLE_JSON_OBJECT
   - GQL_EXECUTOR now uses SIMPLE_JSON_OBJECT

2. ECF was fixed to use simple_json.ecf instead of ISE json.ecf

3. All 19 tests now pass

## Prevention for Future Projects

1. **Before generating any class code**: Check the system_spec.json dependencies
2. **Map dependencies to class names**:
   - `simple_json` → Use SIMPLE_JSON, SIMPLE_JSON_OBJECT, SIMPLE_JSON_ARRAY, SIMPLE_JSON_VALUE
   - `simple_http` → Use SIMPLE_HTTP, SIMPLE_HTTP_RESPONSE
   - Never use $ISE_LIBRARY/contrib/ libraries when a simple_* alternative exists
3. **Follow atomic workflow strictly**: One prompt → One class → One result

---

**Conclusion**: The TOON prompt system works correctly. The failure was Claude not following the prompts and instead using familiar ISE libraries. This is a behavioral issue, not a tooling issue.
