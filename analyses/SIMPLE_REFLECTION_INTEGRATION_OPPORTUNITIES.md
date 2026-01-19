# simple_reflection Integration Opportunities

## Date: 2026-01-19

## Overview

Analysis of simple_* libraries that could benefit from integrating simple_reflection for runtime type introspection, field access, and object graph walking.

## What simple_reflection Provides

- **Type introspection** - Query type metadata at runtime (name, fields, features)
- **Field access** - Read/write object fields by name without compile-time knowledge
- **Type-safe enumerations** - Base class for enums with name/value mapping
- **Bit flags** - Base class for flag enumerations
- **Object graph walking** - Traverse object references with visitor pattern

## High Value Candidates

### Serialization Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_json** | 2 src, 16 tests | Auto-serialize any object to JSON without manual field mapping. Add `to_json_auto(obj: ANY): JSON_VALUE` that reflects all fields. |
| **simple_xml** | 5 src, 2 tests | Auto-serialize objects to XML elements. Each field becomes a child element or attribute. |
| **simple_yaml** | 1 src, 2 tests | Auto-serialize objects to YAML. Nested objects become nested YAML structures. |
| **simple_csv** | 2 src, 2 tests | Map objects to/from CSV rows. Column headers match field names. |

**Example Integration (simple_json):**
```eiffel
-- Current: manual mapping required
json.put_string (customer.name, "name")
json.put_string (customer.email, "email")

-- With reflection: automatic
json := json_serializer.to_json (customer)  -- reflects all fields
```

### ORM/Database Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_sql** | 44 src, 28 tests | Auto-map objects to database rows. Field names become column names. Could add `insert_object`, `update_object`, `select_into_object`. |
| **simple_postgres** | 7 src, 3 tests | PostgreSQL-specific ORM. Map Eiffel objects to/from pg rows automatically. |

**Example Integration (simple_sql):**
```eiffel
-- Current: manual column mapping
db.execute ("INSERT INTO customers (name, email) VALUES (?, ?)", <<customer.name, customer.email>>)

-- With reflection: automatic
db.insert_object ("customers", customer)  -- reflects fields to columns
```

### Testing/Debugging Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_mock** | 7 src, 2 tests | Dynamic mock generation based on class structure. Verify field mutations. |
| **simple_diff** | 7 src, 4 tests | Deep object comparison by field. Return list of field differences. |
| **simple_testing** | 2 src, 2 tests | Assert on object state: `assert_field_equals (obj, "name", "John")` |
| **simple_logger** | 2 src, 2 tests | Dump object state to logs: `log.debug_object (customer)` |

**Example Integration (simple_diff):**
```eiffel
-- Deep object comparison
diff_result := differ.compare (old_customer, new_customer)
across diff_result as ic loop
    print (ic.field_name + ": " + ic.old_value.out + " -> " + ic.new_value.out)
end
```

### Other Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_validation** | 4 src, 2 tests | Generic field validation rules. Validate any object against a schema. |
| **simple_template** | 2 src, 4 tests | Access object fields in templates: `{{customer.name}}` |
| **simple_factory** | 8 src, 7 tests | Already a dependency of simple_reflection. |
| **simple_telemetry** | 1 src, 2 tests | Introspect objects for automatic metrics collection. |

## Priority Recommendations

### Tier 1 (Highest Impact)

1. **simple_json** - Most commonly needed. Auto-serialization would save significant boilerplate.
2. **simple_mock** - Testing is critical. Auto-mocks based on class structure would be powerful.
3. **simple_diff** - Object comparison is needed in many contexts (testing, auditing, sync).

### Tier 2 (High Impact)

4. **simple_sql** - ORM capabilities would make database code much cleaner.
5. **simple_xml** - Common serialization format.
6. **simple_validation** - Generic validation is widely useful.

### Tier 3 (Medium Impact)

7. **simple_template** - Object field access in templates.
8. **simple_logger** - Debug object dumping.
9. **simple_yaml** / **simple_csv** - Less common but still valuable.

## Implementation Notes

### Adding simple_reflection Dependency

```xml
<library name="simple_reflection" location="$SIMPLE_LIBS/simple_reflection/simple_reflection.ecf"/>
```

### Key Classes to Use

- `SIMPLE_TYPE_REGISTRY` - Entry point for type introspection
- `SIMPLE_REFLECTED_OBJECT` - Wrap any object for field access
- `SIMPLE_FIELD_INFO` - Field metadata and read/write operations
- `SIMPLE_OBJECT_GRAPH_WALKER` - Traverse object references

### Performance Considerations

- Type info is cached in `SIMPLE_TYPE_REGISTRY` - create once, reuse
- Field access uses INTERNAL which has some overhead vs direct access
- For hot paths, consider caching field indices

## Next Steps

1. Start with simple_json integration as proof of concept
2. Create `SIMPLE_JSON_SERIALIZER` class that uses reflection
3. Benchmark performance vs manual serialization
4. If acceptable, roll out to other serialization libraries
5. Apply pattern to simple_mock and simple_diff

---

*Analysis generated during simple_reflection hardening session*
