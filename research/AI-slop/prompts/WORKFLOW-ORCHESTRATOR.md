# WORKFLOW ORCHESTRATOR: Execute Anti-Slop Prompts

## STOP. READ THIS FIRST.

You are about to execute a structured workflow. **DO NOT IMPROVISE.**

This orchestrator ensures you:
1. Read and apply the anti-slop rules
2. Execute workflows in correct numerical order
3. Follow each step within each workflow
4. Actually compile and run code (not describe what would happen)

---

## PHASE 0: LOAD ANTI-SLOP RULES (MANDATORY)

**BEFORE DOING ANYTHING ELSE:**

```
READ: D:\prod\reference_docs\research\AI-slop\prompts\00_ANTI-SLOP-RULES.md
```

After reading, acknowledge by stating:
```
ANTI-SLOP RULES LOADED:
- No hypothetical results
- Evidence required for every claim
- Compile before document
- No imagined code behavior
- Document ACTUAL not EXPECTED
```

**If you cannot recite these rules, you have not loaded them. Read again.**

---

## PHASE 1: DETERMINE APPLICABLE WORKFLOWS

The workflows are numbered 01-12. Determine which apply:

| # | Workflow | Applies When |
|---|----------|--------------|
| 01 | project-creation | Creating a NEW library from scratch |
| 02 | spec-extraction | Library EXISTS, need to extract/document its spec |
| 03 | deep-research | Need domain research before design |
| 04 | spec-from-research | Have research, need to convert to spec |
| 05 | design-audit | Library EXISTS, need to audit/improve design |
| 06 | maintenance | Library EXISTS, need standard maintenance |
| 07 | maintenance-xtreme | Library EXISTS, need hardening/stress testing |
| 08 | bug-hunting | Library EXISTS, need to find bugs |
| 09 | bug-fixing | Bugs FOUND, need to fix them |
| 10 | naming-review | Library EXISTS, need naming convention fixes |
| 11 | github-prep | Library READY, need GitHub preparation |
| 12 | ecosystem-integration | Library PUBLISHED, find integration opportunities |

**Ask the user OR determine from context:**
- Is this a new project or existing?
- Which workflows should be applied?

**DEFAULT for existing libraries:** 02 → 05 → 06 → 07 → 08 → 09 → 10 → 11
**DEFAULT for new projects:** 01 → (then treat as existing)
**DEFAULT for publishing:** 10 → 11 → 12

---

## PHASE 2: EXECUTE WORKFLOWS IN ORDER

For each applicable workflow, follow this process:

### Step 2.1: Read the INDEX file

```
READ: D:\prod\reference_docs\research\AI-slop\prompts\{NN}_{workflow-name}\00-INDEX-*.md
```

The INDEX file tells you:
- What the workflow does
- What steps it contains
- What gates/checkpoints exist

### Step 2.2: Execute each step file IN ORDER

For workflow folder `{NN}_{workflow-name}`, read and execute:
- `01-*.md` (first step)
- `02-*.md` (second step)
- ... (all subsequent steps in numerical order)

**DO NOT SKIP STEPS. DO NOT REORDER STEPS.**

### Step 2.3: At each step that involves code:

```bash
# 1. Make the code changes specified in the step

# 2. COMPILE (mandatory)
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# 3. RUN TESTS (if tests exist)
./EIFGENs/{lib}_tests/W_code/{lib}.exe

# 4. Document ACTUAL results
```

### Step 2.4: Create the required outputs

Each step specifies what to create. Create it in the TARGET LIBRARY folder:
- `{lib}/specs/` - Specification documents
- `{lib}/hardening/` - Hardening/test documents
- `{lib}/research/` - Research documents
- `{lib}/audit/` - Audit documents
- `{lib}/testing/` - Actual test .e files

**NOT in the prompts folder. In the library folder.**

### Step 2.5: Complete the verification checkpoint

Before moving to the next step, verify:
```
[ ] I read the step file
[ ] I executed what it says (not just documented)
[ ] I compiled (if code was changed)
[ ] I ran tests (if tests exist)
[ ] I pasted ACTUAL output (not hypothetical)
[ ] I created the required output files
```

---

## PHASE 3: MOVE TO NEXT WORKFLOW

Only after ALL steps in a workflow are complete:

1. Confirm completion with actual evidence
2. Proceed to the next workflow number
3. Read that workflow's INDEX
4. Execute that workflow's steps

---

## CRITICAL RULES (VIOLATIONS = FAILURE)

### FORBIDDEN:
```
✗ Reading a step file and saying "I would do X"
✗ Describing code without writing it
✗ Saying "tests would pass" without running them
✗ Creating documentation without compilation evidence
✗ Skipping steps within a workflow
✗ Executing workflows out of numerical order
✗ Executing a workflow without reading its INDEX first
✗ Executing steps without reading them first
```

### REQUIRED:
```
✓ Read 00_ANTI-SLOP-RULES.md FIRST, every session
✓ Read INDEX file before executing any workflow
✓ Read each step file before executing it
✓ Execute steps in numerical order
✓ Compile after any code change
✓ Run tests after any test addition
✓ Paste ACTUAL output in documentation
✓ Create outputs in the library folder, not prompts folder
```

---

## QUICK REFERENCE: File Locations

```
PROMPTS (read-only templates):
D:\prod\reference_docs\research\AI-slop\prompts\
├── 00_ANTI-SLOP-RULES.md        ← READ FIRST
├── 01_project-creation\         ← For new projects
├── 02_spec-extraction\          ← Extract existing specs
├── 03_deep-research\            ← Domain research
├── 04_spec-from-research\       ← Research → Spec
├── 05_design-audit\             ← Audit design
├── 06_maintenance\              ← Standard maintenance
├── 07_maintenance-xtreme\       ← Hardening
├── 08_bug-hunting\              ← Find bugs
├── 09_bug-fixing\               ← Fix bugs
├── 10_naming-review\            ← Naming convention fixes
├── 11_github-prep\              ← GitHub preparation
└── 12_ecosystem-integration\    ← Find integration opportunities

OUTPUTS (create here):
D:\prod\{library-name}\
├── specs\                       ← From 02, 04
├── research\                    ← From 03
├── audit\                       ← From 05
├── hardening\                   ← From 07
└── testing\                     ← Test .e files

D:\prod\reference_docs\analyses\
└── SIMPLE_{LIB}_INTEGRATION_OPPORTUNITIES.md  ← From 12
```

---

## STARTING THE WORKFLOW

When the user invokes this orchestrator, respond:

```
WORKFLOW ORCHESTRATOR ACTIVATED

Step 1: Loading anti-slop rules...
[Read 00_ANTI-SLOP-RULES.md]

Step 2: Anti-slop rules confirmed:
- No hypothetical results
- Evidence required
- Compile before document
- No imagined behavior
- Document ACTUAL only

Step 3: Target library: {name}
Step 4: Applicable workflows: {list}
Step 5: Beginning workflow {first applicable number}...

[Then READ the INDEX file and begin execution]
```

---

## CONTEXT LOSS / SESSION RESTART

If context is lost and you resume:

1. **FIRST**: Read 00_ANTI-SLOP-RULES.md again
2. Check what's been completed in `{lib}/` folder
3. Determine current workflow and step
4. Read that step's file
5. Continue execution

**DO NOT GUESS where you were. CHECK THE FILES.**
