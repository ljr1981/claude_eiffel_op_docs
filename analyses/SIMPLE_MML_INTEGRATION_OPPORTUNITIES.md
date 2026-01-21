# simple_mml Integration Opportunities

## Date: 2025-01-21

## Overview
Analysis of simple_* libraries that could benefit from integrating simple_mml for model-based contracts.

## What simple_mml Provides
- **MML_MODEL**: Base class with model equality operator `|=|`
- **MML_SET**: Finite mathematical sets for set-theoretic postconditions
- **MML_SEQUENCE**: Ordered collections for stack/queue/list models
- **MML_BAG**: Multisets for counting element occurrences
- **MML_MAP**: Key-value maps for dictionary models
- **MML_RELATION**: Multi-valued relations for many-to-many associations
- **MML_INTERVAL**: Integer ranges for index domains

## Current Usage
- None (library just released)

## High Value Candidates

### Container/Collection Libraries
| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_container** | 14 src, 3 tests | LINQ-style operations need model-based postconditions |
| **simple_sorter** | 6 src, 0 tests | Already enhanced; could add MML postconditions |
| **simple_graph** | 2 src, 2 tests | Graph operations need relation/set models |
| **simple_cache** | 2 src, 2 tests | Cache state could use map models |

### Data Structure Libraries
| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_json** | Multiple src | Object models for serialization contracts |
| **simple_xml** | Multiple src | Tree/set models for document contracts |
| **simple_config** | Multiple src | Map models for configuration state |

## Priority Recommendations

### Tier 1 (Highest Impact)
1. **simple_container** - Core collection operations (partition, group_by, filter, map) would greatly benefit from model-based postconditions that precisely specify output relationships to input. Example:
   ```eiffel
   partition (condition): TUPLE [satisfying, not_satisfying]
       ensure
           is_partition: model_satisfying + model_not_satisfying |=| model
           correct_split: model_satisfying.for_all (agent condition.satisfied_by)
   ```

2. **simple_sorter** - Already has `is_sorted` postcondition; could add `is_permutation` using MML_BAG:
   ```eiffel
   sort (array)
       ensure
           result_sorted: is_sorted (array)
           result_permutation: array.to_bag |=| old array.to_bag
   ```

### Tier 2 (High Impact)
3. **simple_graph** - Graph algorithms (BFS, DFS, shortest path) could use relation models:
   ```eiffel
   edges: MML_RELATION [NODE, NODE]
   reachable (from, to): BOOLEAN
       ensure
           in_transitive_closure: Result = edges.transitive_closure [from, to]
   ```

4. **simple_cache** - Cache operations could model key-value state:
   ```eiffel
   cache_model: MML_MAP [KEY, VALUE]
   put (k, v)
       ensure
           model_updated: cache_model |=| old cache_model.updated (k, v)
   ```

### Tier 3 (Medium Impact)
5. **simple_json** - Object serialization could use map models
6. **simple_config** - Configuration state as map model
7. **simple_diff** - Diff algorithms could use sequence models

## Implementation Notes

### Adding simple_mml Dependency
```xml
<library name="simple_mml" location="$SIMPLE_LIBS/simple_mml/simple_mml.ecf"/>
```

### Key Classes to Use
- `MML_SEQUENCE [G]` - For ordered collections (stacks, queues, lists)
- `MML_SET [G]` - For unique element collections, partitioning
- `MML_BAG [G]` - For permutation checks, counting
- `MML_MAP [K, V]` - For key-value state (caches, configs)
- `MML_RELATION [G, H]` - For graphs, many-to-many associations

### Generic Constraint Pattern
When using MML in your classes, match the detachability:
```eiffel
class MY_CONTAINER [G -> detachable ANY]
feature
    model: MML_SEQUENCE [G]
        -- Model of container contents
```

### Common Integration Patterns

#### Pattern 1: Model Query
```eiffel
feature -- Model
    model: MML_SEQUENCE [G]
            -- Mathematical model of contents.
        do
            create Result.make_from_list (internal_list)
        end
```

#### Pattern 2: Model-Based Postcondition
```eiffel
feature -- Modification
    append (x: G)
            -- Add x to end.
        do
            internal_list.extend (x)
        ensure
            model_extended: model |=| old model & x
        end
```

#### Pattern 3: Set Operations
```eiffel
feature -- Query
    unique_elements: MML_SET [G]
            -- Set of distinct elements.
        do
            Result := model.range
        ensure
            subset_of_model: Result <= model.range
        end
```

## Next Steps
1. **Start with simple_container** - Highest value, already has partial contracts
2. Apply X03-CONTRACT-ASSAULT workflow to enhance contracts with MML
3. Run tests to verify contracts don't break existing functionality
4. Document patterns for broader ecosystem adoption

## Integration Checklist Template

For each integration:
- [ ] Add simple_mml dependency to ECF
- [ ] Add `model` query returning appropriate MML type
- [ ] Enhance postconditions with model-based assertions
- [ ] Run X03-CONTRACT-ASSAULT
- [ ] Verify all tests pass
- [ ] Update README with model documentation

---

*Analysis generated during ecosystem integration review*
