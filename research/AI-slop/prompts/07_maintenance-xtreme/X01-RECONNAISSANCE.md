# X01: Reconnaissance - EXECUTABLE

## THIS IS AN EXECUTABLE STEP

**You MUST read actual source files, compile, and run existing tests to establish baseline.**

## Actions Required

1. **LIST** all source files in `{lib}/src/`
2. **READ** each source file using the Read tool
3. **COMPILE** using `ec.sh` to verify baseline
4. **RUN** existing tests to establish baseline
5. **DOCUMENT** actual findings in `hardening/X01-RECON-ACTUAL.md`

## Step-by-Step Execution

### Step 1: List Source Files

```bash
ls -la /d/prod/{lib}/src/
```

**Record the actual file list.**

### Step 2: Read Each Source File

For each `.e` file in `src/`, use the Read tool:
- Read the entire file
- Note the class name
- Count features
- Count contracts (require/ensure/invariant blocks)

**You must ACTUALLY read the files, not imagine their contents.**

### Step 3: COMPILE Baseline (MANDATORY)

```bash
cd /d/prod/{lib}
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile
```

**PASTE THE ACTUAL COMPILATION OUTPUT.**

This establishes:
- The code compiles before any changes
- Any existing warnings

### Step 4: RUN Baseline Tests (MANDATORY)

```bash
./EIFGENs/{lib}_tests/W_code/{lib}.exe
```

**PASTE THE ACTUAL TEST OUTPUT.**

This establishes:
- How many tests exist
- How many pass/fail before any changes

### Step 5: Create Hardening Directory

```bash
mkdir -p /d/prod/{lib}/hardening
```

### Step 6: Document in `hardening/X01-RECON-ACTUAL.md`

Create this file with ACTUAL data:

```markdown
# X01: Reconnaissance - {LIB_NAME}

## Date: {YYYY-MM-DD}

## Baseline Verification

### Compilation
```
[PASTE ACTUAL ec.sh OUTPUT]
```

### Test Run
```
[PASTE ACTUAL TEST OUTPUT]
```

### Baseline Status
- Compiles: YES
- Tests: [X] pass, [Y] fail
- Warnings: [COUNT]

## Source Files

| File | Class | Lines | Features | Contracts |
|------|-------|-------|----------|-----------|
| {file}.e | {CLASS} | {n} | {n} | {n} pre, {n} post, {n} inv |
| ... | ... | ... | ... | ... |

## Public API Analysis

### {MAIN_CLASS}

| Feature | Type | Params | Pre | Post | Risk |
|---------|------|--------|-----|------|------|
| {name} | creation | {params} | {n} | {n} | [H/M/L] |
| {name} | command | {params} | {n} | {n} | [H/M/L] |
| {name} | query | {params} | {n} | {n} | [H/M/L] |

## Contract Coverage Summary

| Metric | Count | Percentage |
|--------|-------|------------|
| Total features | {n} | 100% |
| With preconditions | {n} | {n}% |
| With postconditions | {n} | {n}% |
| Classes with invariants | {n}/{total} | {n}% |

## Attack Surface Priority

### High (Unprotected public features)
1. `{CLASS}.{feature}` - No preconditions, accepts STRING
2. ...

### Medium (Partial protection)
1. `{CLASS}.{feature}` - Has precondition but no postcondition
2. ...

### Low (Protected)
1. `{CLASS}.{feature}` - Full contracts
2. ...
```

## What You Must Extract

### From Each Class
- Class name
- Feature count
- Contract count (preconditions, postconditions)
- Class invariant (yes/no)

### Entry Points (Attack Vectors)
- All public features
- Their parameter types
- What inputs they accept

### State
- Mutable attributes
- What modifies them
- What protects them

## VERIFICATION CHECKPOINT

Before proceeding to X02, you must have:

```
Compilation output: [PASTED]
Test output: [PASTED]
Source files read: [NUMBER]
Attack surfaces listed: [NUMBER]
hardening/X01-RECON-ACTUAL.md: [CREATED]
```

## Next Step

→ X02-VULNERABILITY-SCAN.md
