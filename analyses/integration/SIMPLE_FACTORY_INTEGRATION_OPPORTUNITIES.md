# simple_factory Integration Opportunities

## Date: 2026-01-19

## Overview

Analysis of simple_* libraries that could benefit from integrating simple_factory for object creation patterns, pooling, singletons, and lazy initialization.

## What simple_factory Provides

- **Factory Pattern** - `SIMPLE_FACTORY`, `SIMPLE_TYPE_FACTORY` for type-safe object creation
- **Singleton Pattern** - `SIMPLE_SINGLETON`, `SIMPLE_SHARED_SINGLETON` for single instances
- **Object Pool** - `SIMPLE_OBJECT_POOL` for reusing expensive objects
- **Lazy Initialization** - `SIMPLE_ONCE_CELL`, `SIMPLE_CACHED_VALUE` for deferred computation
- **Creatable Interface** - `SIMPLE_CREATABLE` for factory-compatible objects

## Current Usage

- `simple_factory` (self)
- `simple_reflection` (uses factory patterns)

## High Value Candidates

### Database/Connection Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_sql** | 44 src, 28 tests | **Connection pooling**. Database connections are expensive. Use `SIMPLE_OBJECT_POOL` for connection reuse. |
| **simple_postgres** | 7 src, 3 tests | **Connection pooling**. PostgreSQL connections need pooling for performance. |

**Example Integration (simple_sql):**
```eiffel
class DATABASE_CONNECTION_POOL inherit SIMPLE_OBJECT_POOL [DATABASE_CONNECTION]
feature
    create_item: DATABASE_CONNECTION
        do
            create Result.make (connection_string)
        end
    
    reset_item (a_conn: DATABASE_CONNECTION)
        do
            a_conn.rollback_if_active
        end
end

-- Usage: connections auto-returned to pool
pool.with_item (agent (conn: DATABASE_CONNECTION)
    do
        conn.execute ("SELECT * FROM users")
    end)
```

### Network Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_http** | 8 src, 3 tests | **Connection pooling** for HTTP keep-alive. **Singleton** for shared HTTP client. |
| **simple_websocket** | 4 src, 2 tests | **Connection pooling** for WebSocket connections. |
| **simple_mq** | 8 src, 3 tests | **Connection pooling** for message queue connections. **Singleton** for queue manager. |
| **simple_ipc** | 4 src, 2 tests | **Object pooling** for IPC channel objects. |

**Example Integration (simple_http):**
```eiffel
class SHARED_HTTP_CLIENT inherit SIMPLE_SHARED_SINGLETON [HTTP_CLIENT]
feature
    new_instance: HTTP_CLIENT
        do
            create Result.make_with_timeout (30)
        end
end

-- Usage: single shared client
http := (create {SHARED_HTTP_CLIENT}).instance
response := http.get ("https://api.example.com/data")
```

### Caching/Registry Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_cache** | 2 src, 2 tests | **Singleton** for global cache. **Lazy init** for cache entries. |
| **simple_registry** | 1 src, 2 tests | **Singleton** for global registry. Registry is inherently singleton. |
| **simple_config** | 1 src, 2 tests | **Singleton** for configuration. **Cached value** for parsed config. |

**Example Integration (simple_config):**
```eiffel
class APP_CONFIG inherit SIMPLE_SINGLETON [APP_CONFIG]
feature
    database_url: STRING
        do
            Result := cached_config.item.database_url
        end
    
    cached_config: SIMPLE_CACHED_VALUE [CONFIG_DATA]
        once
            create Result.make (agent load_config)
        end
    
    load_config: CONFIG_DATA
        do
            create Result.make_from_file ("config.yaml")
        end
end
```

### Logging/Telemetry Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_logger** | 2 src, 2 tests | **Singleton** for global logger. Common pattern. |
| **simple_telemetry** | 1 src, 2 tests | **Singleton** for telemetry collector. **Object pool** for metric objects. |

### Testing Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_mock** | 7 src, 2 tests | **Factory** for mock object creation. `SIMPLE_TYPE_FACTORY` for mock generation. |
| **simple_testing** | 2 src, 2 tests | **Factory** for test fixture creation. |

### Other Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_json** | 2 src, 16 tests | **Object pool** for JSON parser objects (if parsing is expensive). |
| **simple_xml** | 5 src, 2 tests | **Object pool** for XML parser/builder objects. |
| **simple_process** | 3 src, 3 tests | **Object pool** for process handles. |

## Priority Recommendations

### Tier 1 (Highest Impact)

1. **simple_sql** - Connection pooling is critical for database performance.
2. **simple_http** - HTTP connection pooling and shared client singleton.
3. **simple_config** - Configuration singleton is a universal need.

### Tier 2 (High Impact)

4. **simple_logger** - Logger singleton is expected in most applications.
5. **simple_cache** - Cache singleton for global caching.
6. **simple_mq** - Message queue connection pooling.

### Tier 3 (Medium Impact)

7. **simple_postgres** - PostgreSQL-specific connection pooling.
8. **simple_registry** - Registry singleton.
9. **simple_mock** - Mock factory patterns.

## Implementation Notes

### Adding simple_factory Dependency

```xml
<library name="simple_factory" location="$SIMPLE_LIBS/simple_factory/simple_factory.ecf"/>
```

### Key Classes to Use

| Pattern | Class | Use Case |
|---------|-------|----------|
| Factory | `SIMPLE_FACTORY [G]` | Abstract factory for type G |
| Type Factory | `SIMPLE_TYPE_FACTORY` | Create objects by type ID |
| Singleton | `SIMPLE_SINGLETON [G]` | Per-class singleton |
| Shared Singleton | `SIMPLE_SHARED_SINGLETON [G]` | Cross-class shared singleton |
| Object Pool | `SIMPLE_OBJECT_POOL [G]` | Reusable object pool |
| Lazy Value | `SIMPLE_CACHED_VALUE [G]` | Cached computation |
| Once Cell | `SIMPLE_ONCE_CELL [G]` | Thread-safe lazy init |
| Creatable | `SIMPLE_CREATABLE` | Interface for factory-compatible objects |

### Object Pool Pattern

```eiffel
-- Define pool
class MY_POOL inherit SIMPLE_OBJECT_POOL [EXPENSIVE_OBJECT]
feature
    create_item: EXPENSIVE_OBJECT do create Result.make end
    reset_item (item: EXPENSIVE_OBJECT) do item.reset end
end

-- Use pool
pool: MY_POOL
create pool.make (10)  -- max 10 objects

item := pool.acquire
-- use item
pool.release (item)

-- Or with automatic release:
pool.with_item (agent (obj: EXPENSIVE_OBJECT) do obj.do_work end)
```

### Singleton Pattern

```eiffel
class MY_SINGLETON inherit SIMPLE_SINGLETON [MY_SINGLETON]
feature
    do_something do ... end
end

-- Usage
singleton := (create {MY_SINGLETON}).instance
singleton.do_something
```

## Next Steps

1. Start with simple_sql connection pooling - highest practical impact
2. Add logger singleton to simple_logger
3. Add config singleton to simple_config
4. Roll out pooling to other connection-based libraries

---

*Analysis generated during ecosystem integration review*
