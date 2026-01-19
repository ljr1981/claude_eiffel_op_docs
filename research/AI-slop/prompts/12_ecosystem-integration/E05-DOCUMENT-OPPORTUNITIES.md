# E05: Document Integration Opportunities

## Objective
Create a comprehensive report for reference_docs/analyses/.

## Steps

### 1. Create Report File

```bash
# Report location
/d/prod/reference_docs/analyses/SIMPLE_{LIB_NAME}_INTEGRATION_OPPORTUNITIES.md
```

### 2. Report Structure

```markdown
# simple_{lib_name} Integration Opportunities

## Date: {date}

## Overview
Analysis of simple_* libraries that could benefit from integrating simple_{lib_name}.

## What simple_{lib_name} Provides
- {Capability 1}
- {Capability 2}
- {Capability 3}

## Current Usage
- {List of libraries already using it}

## High Value Candidates

### {Category 1} Libraries
| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **{lib}** | X src, Y tests | {Description} |

### {Category 2} Libraries
| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **{lib}** | X src, Y tests | {Description} |

## Priority Recommendations

### Tier 1 (Highest Impact)
1. **{lib_1}** - {Why it's high priority}
2. **{lib_2}** - {Why it's high priority}

### Tier 2 (High Impact)
3. **{lib_3}** - {Description}

### Tier 3 (Medium Impact)
4. **{lib_4}** - {Description}

## Implementation Notes

### Adding simple_{lib_name} Dependency
```xml
<library name="simple_{lib_name}" location="$SIMPLE_LIBS/simple_{lib_name}/simple_{lib_name}.ecf"/>
```

### Key Classes to Use
- `CLASS_1` - {Usage}
- `CLASS_2` - {Usage}

### Common Patterns
```eiffel
{Example code showing integration pattern}
```

## Next Steps
1. Start with {highest priority candidate}
2. Create proof of concept
3. Benchmark if performance-sensitive
4. Roll out to other candidates

---

*Analysis generated during ecosystem integration review*
```

### 3. Save Report

```bash
# Write the report
python3 /d/prod/scripts/python/write_file.py \
  "/d/prod/reference_docs/analyses/SIMPLE_{LIB_NAME}_INTEGRATION_OPPORTUNITIES.md" \
  --stdin << 'REPORT'
{report_content}
REPORT
```

### 4. Verify Report

```bash
# Check report exists
ls -la /d/prod/reference_docs/analyses/SIMPLE_*_INTEGRATION_OPPORTUNITIES.md
```

## Output
`reference_docs/analyses/SIMPLE_{LIB_NAME}_INTEGRATION_OPPORTUNITIES.md`

## Verification Checkpoint
- [ ] Report created with all sections
- [ ] Priorities clearly stated
- [ ] Example code included
- [ ] Next steps defined
- [ ] File saved to reference_docs/analyses/

## Workflow Complete
Ecosystem integration analysis is complete. The report can be used to:
1. Plan future integration work
2. Guide ecosystem evolution
3. Identify synergies between libraries
