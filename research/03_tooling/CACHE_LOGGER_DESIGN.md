# simple_cache & simple_logger Design Document

**Date:** December 6, 2025
**Author:** Larry Rix with Claude (Anthropic)
**Status:** Design Phase - Victory Lap (Week 4)

---

## Executive Summary

This document presents the API design for the final two libraries of the Christmas Sprint:
- **simple_logger** - Structured JSON logging (FOUNDATION_API layer)
- **simple_cache** - In-memory LRU cache with TTL (SERVICE_API layer)

Both designs are informed by industry best practices, relevant RFCs/specs, and common developer pain points.

---

## Part 1: simple_logger

### Layer Placement: FOUNDATION_API

Logging is a core utility needed by all applications, similar to hashing, encoding, and validation. It has no external service dependencies (database, network, etc.), making it a foundation-level concern.

### Industry Research

#### Leading Libraries Analyzed

| Library | Language | Key Strengths |
|---------|----------|---------------|
| **Pino** | Node.js | Fastest JSON logger, low overhead, child loggers |
| **Winston** | Node.js | Transport system, multiple outputs, flexible |
| **slog** | Go 1.21+ | Structured logging, stdlib, log/slog |
| **Zerolog** | Go | Zero allocation, blazing fast, JSON-native |
| **Logback/SLF4J** | Java | Industry standard, pattern layouts |

#### Common Patterns Identified

1. **Structured Logging** - Key-value pairs, not string concatenation
2. **Log Levels** - DEBUG, INFO, WARN, ERROR, FATAL (RFC 5424 has 8 levels)
3. **Context/Child Loggers** - Inherit fields, add context
4. **Output Targets** - Console, file, network
5. **JSON Format** - Machine-parseable for log aggregation

### RFC 5424 Syslog Protocol Reference

RFC 5424 defines 8 severity levels:
```
0 - Emergency (system unusable)
1 - Alert (immediate action needed)
2 - Critical (critical conditions)
3 - Error (error conditions)
4 - Warning (warning conditions)
5 - Notice (normal but significant)
6 - Informational (informational)
7 - Debug (debug-level messages)
```

**Eiffel Simplification:** We'll use 5 levels (DEBUG, INFO, WARN, ERROR, FATAL) which covers 95% of use cases while remaining simple.

### Developer Pain Points Addressed

| Pain Point | Solution |
|------------|----------|
| Verbose setup | Single-line configuration |
| String concatenation | Structured `.with()` chaining |
| Missing context | Child loggers inherit fields |
| Performance overhead | Lazy evaluation, level filtering |
| No JSON output | JSON is default format |
| Log rotation | Built-in file rotation support |

### API Design: SIMPLE_LOGGER

```eiffel
class SIMPLE_LOGGER

create
    make, make_with_level, make_to_file

feature -- Log Levels (constants)
    Level_debug: INTEGER = 1
    Level_info: INTEGER = 2
    Level_warn: INTEGER = 3
    Level_error: INTEGER = 4
    Level_fatal: INTEGER = 5

feature {NONE} -- Initialization

    make
            -- Create logger with INFO level, console output.
        ensure
            level_is_info: level = Level_info
            outputs_to_console: is_console_output
        end

    make_with_level (a_level: INTEGER)
            -- Create logger with specified level, console output.
        require
            valid_level: a_level >= Level_debug and a_level <= Level_fatal
        ensure
            level_set: level = a_level
        end

    make_to_file (a_path: STRING)
            -- Create logger outputting to file.
        require
            path_not_empty: not a_path.is_empty
        ensure
            outputs_to_file: is_file_output
        end

feature -- Configuration

    set_level (a_level: INTEGER)
            -- Set minimum log level.
        require
            valid_level: a_level >= Level_debug and a_level <= Level_fatal
        ensure
            level_set: level = a_level
        end

    set_json_output (enabled: BOOLEAN)
            -- Enable/disable JSON formatted output.
        ensure
            json_set: is_json_output = enabled
        end

    add_file_output (a_path: STRING)
            -- Add file output in addition to current outputs.
        require
            path_not_empty: not a_path.is_empty
        end

feature -- Access

    level: INTEGER
            -- Current minimum log level.

    is_console_output: BOOLEAN
            -- Is outputting to console?

    is_file_output: BOOLEAN
            -- Is outputting to file?

    is_json_output: BOOLEAN
            -- Is using JSON format?

feature -- Logging (Simple)

    debug (a_message: STRING)
            -- Log debug message.
        require
            message_not_void: a_message /= Void

    info (a_message: STRING)
            -- Log info message.
        require
            message_not_void: a_message /= Void

    warn (a_message: STRING)
            -- Log warning message.
        require
            message_not_void: a_message /= Void

    error (a_message: STRING)
            -- Log error message.
        require
            message_not_void: a_message /= Void

    fatal (a_message: STRING)
            -- Log fatal message.
        require
            message_not_void: a_message /= Void

feature -- Logging (Structured)

    debug_with (a_message: STRING; a_fields: HASH_TABLE [ANY, STRING])
            -- Log debug with structured fields.
        require
            message_not_void: a_message /= Void
            fields_not_void: a_fields /= Void

    info_with (a_message: STRING; a_fields: HASH_TABLE [ANY, STRING])
            -- Log info with structured fields.

    warn_with (a_message: STRING; a_fields: HASH_TABLE [ANY, STRING])
            -- Log warning with structured fields.

    error_with (a_message: STRING; a_fields: HASH_TABLE [ANY, STRING])
            -- Log error with structured fields.

    fatal_with (a_message: STRING; a_fields: HASH_TABLE [ANY, STRING])
            -- Log fatal with structured fields.

feature -- Child Loggers

    child (a_context: HASH_TABLE [ANY, STRING]): SIMPLE_LOGGER
            -- Create child logger with inherited context.
            -- All logs from child will include parent's context fields.
        require
            context_not_void: a_context /= Void
        ensure
            result_not_void: Result /= Void
            inherits_level: Result.level = level

feature -- Convenience (Timing)

    start_timer: SIMPLE_LOG_TIMER
            -- Start a timer for measuring duration.
        ensure
            timer_started: Result /= Void

    log_duration (a_timer: SIMPLE_LOG_TIMER; a_message: STRING)
            -- Log info message with duration from timer.
        require
            timer_not_void: a_timer /= Void
            message_not_void: a_message /= Void

feature -- Trace Context (for distributed tracing)

    set_trace_id (a_trace_id: STRING)
            -- Set trace ID for correlation (appears in all logs).
        require
            trace_id_not_empty: not a_trace_id.is_empty

    set_request_id (a_request_id: STRING)
            -- Set request ID for correlation.
        require
            request_id_not_empty: not a_request_id.is_empty

end
```

### Usage Examples

```eiffel
-- Basic usage
local
    log: SIMPLE_LOGGER
do
    create log.make
    log.info ("Application started")
    log.debug ("Processing request")
    log.error ("Failed to connect to database")
end

-- Structured logging
local
    log: SIMPLE_LOGGER
    fields: HASH_TABLE [ANY, STRING]
do
    create log.make
    log.set_json_output (True)

    create fields.make (3)
    fields.put ("user123", "user_id")
    fields.put (42, "order_id")
    fields.put ("checkout", "action")

    log.info_with ("Order processed", fields)
    -- Output: {"level":"info","message":"Order processed","user_id":"user123","order_id":42,"action":"checkout","timestamp":"2025-12-06T19:30:00Z"}
end

-- Child logger with context
local
    log, request_log: SIMPLE_LOGGER
    ctx: HASH_TABLE [ANY, STRING]
do
    create log.make
    create ctx.make (2)
    ctx.put ("req-abc-123", "request_id")
    ctx.put ("192.168.1.1", "client_ip")

    request_log := log.child (ctx)
    request_log.info ("Handling request")  -- Includes request_id and client_ip
    request_log.info ("Request complete")  -- Same context
end

-- Duration logging
local
    log: SIMPLE_LOGGER
    timer: SIMPLE_LOG_TIMER
do
    create log.make
    timer := log.start_timer
    -- ... do work ...
    log.log_duration (timer, "Database query completed")
    -- Output: "Database query completed" duration_ms=152
end
```

### Output Formats

**Plain Text (default):**
```
2025-12-06T19:30:00 INFO  Application started
2025-12-06T19:30:01 DEBUG Processing user_id=user123 action=checkout
2025-12-06T19:30:02 ERROR Failed to connect to database
```

**JSON (for log aggregation):**
```json
{"timestamp":"2025-12-06T19:30:00Z","level":"info","message":"Application started"}
{"timestamp":"2025-12-06T19:30:01Z","level":"debug","message":"Processing","user_id":"user123","action":"checkout"}
{"timestamp":"2025-12-06T19:30:02Z","level":"error","message":"Failed to connect to database"}
```

---

## Part 2: simple_cache

### Layer Placement: SERVICE_API

Caching is a service infrastructure concern:
- Requires eviction policies (complexity beyond foundation)
- Often used with external services (Redis future extension)
- Fits with other SERVICE_API components (rate limiter, database)

### Industry Research

#### Leading Libraries Analyzed

| Library | Platform | Key Strengths |
|---------|----------|---------------|
| **Redis** | Server | Rich data types, pub/sub, persistence |
| **Memcached** | Server | Simple, fast, distributed |
| **go-cache** | Go | In-process, TTL, cleanup goroutine |
| **caffeine** | Java | Near-optimal hit rates, async refresh |
| **node-cache** | Node.js | Simple in-memory, statistics |

#### Eviction Policies (from Redis)

| Policy | Description | Use Case |
|--------|-------------|----------|
| **LRU** | Least Recently Used | General purpose |
| **LFU** | Least Frequently Used | Hot data patterns |
| **TTL** | Time-to-Live expiration | Session data |
| **FIFO** | First In First Out | Simple scenarios |
| **Random** | Random eviction | When simplicity matters |

**Eiffel Choice:** LRU with TTL - covers 90% of use cases, well-understood algorithm.

### Developer Pain Points Addressed

| Pain Point | Solution |
|------------|----------|
| Complex Redis setup | In-memory, zero config |
| No TTL support | Built-in expiration |
| Memory unbounded | Max size with eviction |
| No statistics | Hit/miss counters |
| Thundering herd | Optional locking |
| Type safety | Generic with contracts |

### API Design: SIMPLE_CACHE

```eiffel
class SIMPLE_CACHE [G]
    -- In-memory LRU cache with TTL support.
    -- Generic over value type G.

create
    make, make_with_capacity

feature {NONE} -- Initialization

    make
            -- Create cache with default capacity (1000 items).
        ensure
            capacity_set: capacity = Default_capacity
            empty: count = 0
        end

    make_with_capacity (a_capacity: INTEGER)
            -- Create cache with specified maximum capacity.
        require
            positive_capacity: a_capacity > 0
        ensure
            capacity_set: capacity = a_capacity
            empty: count = 0
        end

feature -- Configuration

    set_default_ttl (a_seconds: INTEGER)
            -- Set default TTL for items without explicit TTL.
            -- Use 0 for no expiration.
        require
            non_negative: a_seconds >= 0
        ensure
            ttl_set: default_ttl = a_seconds
        end

feature -- Access

    capacity: INTEGER
            -- Maximum number of items.

    count: INTEGER
            -- Current number of items (excluding expired).

    default_ttl: INTEGER
            -- Default time-to-live in seconds (0 = no expiration).

    has (a_key: STRING): BOOLEAN
            -- Is there a non-expired item with this key?
        require
            key_not_empty: not a_key.is_empty
        ensure
            consistent: Result implies item (a_key) /= Void

    item (a_key: STRING): detachable G
            -- Get item by key, or Void if not found/expired.
            -- Updates access time for LRU.
        require
            key_not_empty: not a_key.is_empty

    item_or_default (a_key: STRING; a_default: G): G
            -- Get item by key, or default if not found/expired.
        require
            key_not_empty: not a_key.is_empty
        ensure
            result_not_void: Result /= Void

feature -- Element Change

    put (a_key: STRING; a_value: G)
            -- Store item with default TTL.
            -- Evicts LRU item if at capacity.
        require
            key_not_empty: not a_key.is_empty
        ensure
            stored: has (a_key)
            value_set: item (a_key) ~ a_value

    put_with_ttl (a_key: STRING; a_value: G; a_ttl_seconds: INTEGER)
            -- Store item with specific TTL.
        require
            key_not_empty: not a_key.is_empty
            positive_ttl: a_ttl_seconds > 0
        ensure
            stored: has (a_key)
            value_set: item (a_key) ~ a_value

    remove (a_key: STRING)
            -- Remove item by key (no-op if not found).
        require
            key_not_empty: not a_key.is_empty
        ensure
            removed: not has (a_key)

    clear
            -- Remove all items.
        ensure
            empty: count = 0

feature -- Bulk Operations

    put_all (a_items: HASH_TABLE [G, STRING])
            -- Store multiple items with default TTL.
        require
            items_not_void: a_items /= Void

    get_all (a_keys: ARRAY [STRING]): HASH_TABLE [G, STRING]
            -- Get multiple items (missing keys excluded from result).
        require
            keys_not_void: a_keys /= Void
        ensure
            result_not_void: Result /= Void

    remove_all (a_keys: ARRAY [STRING])
            -- Remove multiple items.
        require
            keys_not_void: a_keys /= Void

feature -- Statistics

    hits: INTEGER_64
            -- Number of cache hits.

    misses: INTEGER_64
            -- Number of cache misses.

    evictions: INTEGER_64
            -- Number of items evicted due to capacity.

    expirations: INTEGER_64
            -- Number of items expired due to TTL.

    hit_rate: REAL_64
            -- Hit rate as percentage (0.0 to 100.0).
        ensure
            valid_range: Result >= 0.0 and Result <= 100.0

    reset_statistics
            -- Reset all statistics counters.
        ensure
            hits_reset: hits = 0
            misses_reset: misses = 0
            evictions_reset: evictions = 0

feature -- Maintenance

    cleanup_expired
            -- Remove all expired items (automatic on access, but can force).
        ensure
            no_expired: -- all remaining items are non-expired

    keys: ARRAY [STRING]
            -- All non-expired keys (for debugging/inspection).
        ensure
            result_not_void: Result /= Void

feature -- Advanced: Compute if Absent

    get_or_compute (a_key: STRING; a_compute: FUNCTION [G]): G
            -- Get item or compute and store if absent/expired.
            -- Prevents thundering herd by computing once.
        require
            key_not_empty: not a_key.is_empty
            compute_not_void: a_compute /= Void
        ensure
            result_not_void: Result /= Void
            stored: has (a_key)

    get_or_compute_with_ttl (a_key: STRING; a_compute: FUNCTION [G]; a_ttl: INTEGER): G
            -- Get item or compute with specific TTL.
        require
            key_not_empty: not a_key.is_empty
            compute_not_void: a_compute /= Void
            positive_ttl: a_ttl > 0
        ensure
            result_not_void: Result /= Void
            stored: has (a_key)

feature {NONE} -- Constants

    Default_capacity: INTEGER = 1000

invariant
    valid_capacity: capacity > 0
    valid_count: count >= 0 and count <= capacity
    non_negative_stats: hits >= 0 and misses >= 0 and evictions >= 0

end
```

### Supporting Class: CACHE_ENTRY

```eiffel
class CACHE_ENTRY [G]
    -- Internal cache entry with metadata.

create
    make

feature {NONE} -- Initialization

    make (a_key: STRING; a_value: G; a_ttl_seconds: INTEGER)
        require
            key_not_empty: not a_key.is_empty
        ensure
            key_set: key.same_string (a_key)
            value_set: value ~ a_value

feature -- Access

    key: STRING
    value: G
    created_at: DATE_TIME
    last_accessed: DATE_TIME
    ttl_seconds: INTEGER

feature -- Status

    is_expired: BOOLEAN
            -- Has this entry expired?
        do
            if ttl_seconds > 0 then
                Result := seconds_since_creation > ttl_seconds
            end
        end

    seconds_since_creation: INTEGER_64

feature -- Update

    touch
            -- Update last_accessed time (for LRU).
        ensure
            accessed_updated: last_accessed /= old last_accessed

end
```

### Usage Examples

```eiffel
-- Basic usage
local
    cache: SIMPLE_CACHE [STRING]
do
    create cache.make
    cache.put ("user:123", "John Doe")

    if attached cache.item ("user:123") as name then
        print ("Found: " + name)
    end
end

-- With TTL
local
    cache: SIMPLE_CACHE [SESSION_DATA]
do
    create cache.make_with_capacity (10000)
    cache.set_default_ttl (3600)  -- 1 hour default

    -- Store session with 30-minute TTL
    cache.put_with_ttl ("session:abc", session_data, 1800)
end

-- Compute if absent (prevents thundering herd)
local
    cache: SIMPLE_CACHE [USER_DATA]
    user: USER_DATA
do
    create cache.make

    -- If not in cache, load from database (once)
    user := cache.get_or_compute ("user:456",
        agent load_user_from_db ("456"))
end

-- Statistics
local
    cache: SIMPLE_CACHE [ANY]
do
    create cache.make
    -- ... use cache ...

    print ("Hit rate: " + cache.hit_rate.out + "%%")
    print ("Hits: " + cache.hits.out)
    print ("Misses: " + cache.misses.out)
    print ("Evictions: " + cache.evictions.out)
end
```

---

## Integration with API Layers

### FOUNDATION_API Addition

```eiffel
class FOUNDATION_API

feature -- Logging

    new_logger: SIMPLE_LOGGER
            -- Create a new logger (INFO level, console output).
        do
            create Result.make
        ensure
            result_not_void: Result /= Void
        end

    new_logger_with_level (a_level: INTEGER): SIMPLE_LOGGER
            -- Create logger with specific level.
        require
            valid_level: a_level >= 1 and a_level <= 5
        do
            create Result.make_with_level (a_level)
        ensure
            result_not_void: Result /= Void
        end

    logger: SIMPLE_LOGGER
            -- Shared application logger (singleton).
        once
            create Result.make
        ensure
            result_not_void: Result /= Void
        end

end
```

### SERVICE_API Addition

```eiffel
class SERVICE_API

feature -- Caching

    new_cache: SIMPLE_CACHE [ANY]
            -- Create cache with default capacity.
        do
            create Result.make
        ensure
            result_not_void: Result /= Void
        end

    new_cache_with_capacity (a_capacity: INTEGER): SIMPLE_CACHE [ANY]
            -- Create cache with specific capacity.
        require
            positive: a_capacity > 0
        do
            create Result.make_with_capacity (a_capacity)
        ensure
            result_not_void: Result /= Void
        end

    new_typed_cache_string: SIMPLE_CACHE [STRING]
            -- Create string cache (convenience).
        do
            create Result.make
        end

end
```

### APP_API Usage (via composition)

```eiffel
-- Application code
local
    api: APP_API
do
    create api.make

    -- Logging via foundation
    api.foundation.logger.info ("App started")

    -- Caching via service
    api.service.new_cache.put ("key", "value")
end
```

---

## Implementation Notes

### simple_logger Implementation Details

1. **Thread Safety**: Use `once` for shared logger, or recommend separate loggers per thread
2. **File Rotation**: Check file size, rename and create new on threshold
3. **Performance**: Skip string formatting if level is filtered
4. **JSON Escaping**: Properly escape special characters in field values

### simple_cache Implementation Details

1. **LRU Implementation**: Doubly-linked list + hash table for O(1) operations
2. **Expiration Check**: Lazy (on access) + optional cleanup routine
3. **Capacity Eviction**: Remove LRU item when full before inserting new
4. **Statistics**: Atomic increments (or accept minor inaccuracy in SCOOP context)

---

## Test Coverage Requirements

### simple_logger Tests

- [ ] Log level filtering (debug not shown at INFO level)
- [ ] Structured fields appear in output
- [ ] Child logger inherits parent context
- [ ] JSON format validation
- [ ] File output creates file
- [ ] Duration logging calculates correctly

### simple_cache Tests

- [ ] Put/get roundtrip
- [ ] TTL expiration
- [ ] LRU eviction (oldest accessed removed first)
- [ ] Capacity limit enforced
- [ ] Statistics accuracy
- [ ] get_or_compute only computes once
- [ ] Bulk operations
- [ ] clear removes all

---

## Success Criteria

Per CHRISTMAS_SPRINT.md requirements:

- [ ] Clean compilation with contracts enabled
- [ ] 90%+ test coverage
- [ ] README with usage examples
- [ ] Pushed to GitHub
- [ ] Added to LIBRARY_ROADMAP.md as "Production"
- [ ] Integrated into API facade layers

---

## Revision History

| Date | Change |
|------|--------|
| 2025-12-06 | Initial design document created |

