# S01: Inventory Project Structure

## Context
Phase: **DISCOVERY** (1 of 3)
Purpose: Map all source material before extraction

## Input Required
- ECF file(s)
- Source directory structure
- Any existing documentation (README, docs/, comments)

## Prompt

```
TASK: Create comprehensive inventory of project artifacts for spec extraction.

INPUT:
{ecf_files}
{directory_listing}
{readme_or_docs}

═══════════════════════════════════════════════════════════════
PROJECT IDENTITY
What is this library?
═══════════════════════════════════════════════════════════════

From ECF and README extract:
- Library name: {name}
- Library UUID: {uuid}
- Stated purpose: {from note clause or README}
- Version: {if available}

═══════════════════════════════════════════════════════════════
DEPENDENCY ANALYSIS
What does this library depend on?
═══════════════════════════════════════════════════════════════

For each library dependency in ECF:
- Dependency: {library name}
- Location: {path or $VAR}
- Why needed: {infer from usage}

DEPENDENCY CATEGORIES:
- Core (EiffelBase, etc.)
- Simple ecosystem (simple_*)
- External (third-party)

═══════════════════════════════════════════════════════════════
CLUSTER ANALYSIS
How is the code organized?
═══════════════════════════════════════════════════════════════

For each cluster:
- Cluster: {name}
- Location: {path}
- Classes: {list}
- Apparent purpose: {infer from name and contents}

CLUSTER PATTERNS:
- src/ = main implementation
- testing/ = test classes
- internal/ = private implementation
- api/ = public interface

═══════════════════════════════════════════════════════════════
CLASS INVENTORY
What classes exist?
═══════════════════════════════════════════════════════════════

For each class:
CLASS: {name}
  File: {path}
  Cluster: {cluster}
  Has note clause: [YES | NO]
  Creation procedures: {list}
  Public features: {count}
  Has invariant: [YES | NO]
  Inherits from: {parent list}
  Role: [FACADE | ENGINE | DATA | HELPER | TEST]

═══════════════════════════════════════════════════════════════
FACADE IDENTIFICATION
Which class is the main entry point?
═══════════════════════════════════════════════════════════════

Facade candidates (usually named SIMPLE_{X} or matches library name):
- Class: {name}
- Evidence: {why this is the facade}

═══════════════════════════════════════════════════════════════
TEST INVENTORY
What tests exist?
═══════════════════════════════════════════════════════════════

Test classes found:
- Class: {name}
- Test count: {number of test_ features}
- Tests: {list of test names}

OUTPUT FORMAT:
PROJECT INVENTORY:
  Name: {library_name}
  Purpose: {stated or inferred}

DEPENDENCIES: {count}
  - {dep1}: {purpose}
  - {dep2}: {purpose}

CLUSTERS: {count}
  - {cluster1}: {class_count} classes - {purpose}

CLASSES: {count}
  Facades: {list}
  Engines: {list}
  Data: {list}
  Helpers: {list}
  Tests: {list}

TEST COVERAGE:
  Test classes: {count}
  Total tests: {count}
  Features tested: {list or "unknown"}

DOCUMENTATION STATUS:
  README: [PRESENT | ABSENT]
  Note clauses: {X}% of classes
  Header comments: {X}% of features
```

## Success Criteria
- All ECF dependencies mapped
- All clusters identified with purpose
- All classes inventoried with role
- Facade class(es) identified
- Test coverage baseline established

## Next Step
→ S02-EXTRACT-DOMAIN-MODEL.md
