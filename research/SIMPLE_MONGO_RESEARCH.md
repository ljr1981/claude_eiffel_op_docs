# SIMPLE_MONGO Research Report

**Date:** 2025-12-08
**Library:** simple_mongo
**Wraps:** ISE MongoDB library (`$ISE_LIBRARY/contrib/library/persistency/mongo/mongodb/mongodb.ecf`)

---

## Seven-Step Research Process

### Step 1: ISE Library Structure

**Location:** `$ISE_LIBRARY/contrib/library/persistency/mongo/`

**Key Components:**
```
mongo/
├── Readme.md                 -- Overview documentation
├── package.iron              -- Iron package configuration
├── C/                        -- MongoDB C driver build utilities
│   └── driver/               -- Pre-built libmongoc binaries
│       ├── include/          -- C headers (libmongoc-1.0)
│       └── lib/              -- Library files (mongoc-1.0.lib)
├── bson/                     -- BSON library wrapper
│   └── lib/
│       ├── bson.ecf          -- BSON ECF
│       └── src/
│           ├── bson.e                    -- BSON document
│           ├── bson_iterator.e           -- Document iteration
│           ├── bson_oid.e                -- ObjectID
│           ├── bson_error.e              -- Error handling
│           ├── bson_value.e              -- Value wrapper
│           ├── bson_type.e               -- Type constants
│           └── bson_wrapper_base.e       -- Base class
└── mongodb/                  -- MongoDB driver
    ├── mongodb.ecf           -- Main ECF
    └── src/
        ├── mongodb_client.e              -- Connection to server
        ├── mongodb_collection.e          -- CRUD operations
        ├── mongodb_database.e            -- Database operations
        ├── mongodb_cursor.e              -- Query result iteration
        ├── mongodb_driver.e              -- Driver initialization
        ├── mongodb_uri.e                 -- Connection string parsing
        ├── mongodb_bulk_operation.e      -- Bulk writes
        ├── mongodb_client_session.e      -- Transaction sessions
        ├── mongodb_read_concern.e        -- Read consistency
        ├── mongodb_write_concern.e       -- Write durability
        └── mongodb_find_and_modify_options.e
```

### Step 2: Core Classes Analysis

**MONGODB_DRIVER**
- Must be initialized before using any MongoDB operations
- `use` - Initialize the C driver (calls mongoc_init)
- `cleanup` - Clean up driver resources (calls mongoc_cleanup)

**MONGODB_CLIENT**
- Main entry point for MongoDB operations
- `make (a_uri)` - Connect using connection string (e.g., "mongodb://localhost:27017")
- `make_from_uri (a_uri: MONGODB_URI)` - Connect using parsed URI
- `collection (a_db, a_collection)` - Get collection handle
- `database (a_dbname)` - Get database handle
- `database_names` - List all databases
- `ping (a_db)` - Test server connectivity
- `start_session` - Begin a transaction session
- `destroy` - Disconnect and release resources

**MONGODB_COLLECTION**
- CRUD operations on a collection
- `insert_one (a_document, a_opts, a_reply)` - Insert single document
- `insert_many (a_documents, a_opts, a_reply)` - Insert multiple documents
- `find_with_opts (a_filter, a_opts, a_read_prefs)` - Query documents
- `count_documents (a_filter, a_opts, a_read_prefs, a_reply)` - Count matching documents
- `update_one (a_selector, a_update, a_opts, a_reply)` - Update first match
- `update_many (a_selector, a_update, a_opts, a_reply)` - Update all matches
- `delete_one (a_selector, a_opts, a_reply)` - Delete first match
- `delete_many (a_selector, a_opts, a_reply)` - Delete all matches
- `aggregate (a_pipeline, a_opts, a_read_pref)` - Aggregation pipeline
- `find_and_modify` - Atomic find and update/delete
- `drop` - Delete the collection
- `last_call_succeed` - Check if last operation succeeded
- `last_error` - Get error details if failed

**MONGODB_CURSOR**
- Lazy iteration over query results
- `next` - Get next BSON document (Void when exhausted)
- `cursor_error` - Check for iteration errors

**BSON**
- Binary JSON document representation
- `make` - Create empty document
- `make_from_json (a_json)` - Parse JSON string to BSON
- `bson_append_utf8 (a_key, a_value)` - Add string field
- `bson_append_integer_32 (a_key, a_value)` - Add 32-bit integer
- `bson_append_integer_64 (a_key, a_value)` - Add 64-bit integer
- `bson_append_double (a_key, a_value)` - Add floating point
- `bson_append_boolean (a_key, a_value)` - Add boolean
- `bson_append_null (a_key)` - Add null value
- `bson_append_oid (a_key, a_oid)` - Add ObjectID
- `bson_append_date_time (a_key, a_value)` - Add datetime (milliseconds since epoch)
- `bson_append_document (a_key, a_document)` - Add nested document
- `bson_append_array (a_key, a_array)` - Add array
- `bson_as_canonical_extended_json` - Convert to JSON string
- `bson_as_relaxed_extended_json` - Convert to relaxed JSON
- `bson_has_field (a_key)` - Check if field exists
- `bson_count_keys` - Count top-level fields

**BSON_OID**
- MongoDB ObjectID (12-byte unique identifier)
- `make` - Generate new ObjectID
- `make_from_string` - Parse hex string
- `to_string` - Convert to 24-char hex string

### Step 3: Connection String Format

MongoDB URI format:
```
mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[database][?options]]
```

Examples:
- Local: `mongodb://localhost:27017`
- With auth: `mongodb://user:password@localhost:27017/mydb`
- Replica set: `mongodb://host1:27017,host2:27017,host3:27017/mydb?replicaSet=myrs`
- With options: `mongodb://localhost:27017/mydb?authSource=admin&retryWrites=true`

### Step 4: BSON vs JSON

The ISE library provides excellent BSON/JSON interoperability:
- `BSON.make_from_json (json_string)` - Parse JSON to BSON
- `BSON.bson_as_canonical_extended_json` - Strict JSON output
- `BSON.bson_as_relaxed_extended_json` - Human-readable JSON

This allows our wrapper to work with JSON for ease of use while using BSON internally.

### Step 5: Design Decisions for simple_mongo

**CRITICAL: Server Requirement**
MongoDB operations require a running MongoDB server. This affects testing strategy:
1. Unit tests should use mocking/stubbing
2. Integration tests require actual MongoDB server
3. API should be designed to allow dependency injection for testing

**Simplifications:**
1. **JSON-first API** - Accept/return JSON strings instead of raw BSON
2. **Fluent builder pattern** - For building queries
3. **Simplified error handling** - Check success/failure, get message
4. **Auto driver initialization** - Handle mongoc_init/cleanup automatically
5. **Connection pooling abstraction** - Hide complexity of connection management

**Key Features:**
- Simple connect/disconnect
- Full CRUD operations (insert, find, update, delete)
- Query builders for common patterns
- JSON document creation helpers
- Aggregation pipeline support
- Index management (simplified)
- Transaction support (optional advanced feature)

**Testing Strategy:**
- Unit tests: Test JSON/BSON conversion, query building, error handling (no server needed)
- Integration tests: Require running MongoDB server (marked as optional)
- Mock client for dependency injection in tests

### Step 6: API Design

```eiffel
class SIMPLE_MONGO

create
  make,                          -- Connect to localhost:27017
  make_with_uri,                 -- Connect with URI string
  make_for_testing               -- Create mock/testable instance

feature -- Connection
  connect: BOOLEAN               -- Establish connection (if not already)
  disconnect                     -- Close connection
  is_connected: BOOLEAN          -- Connection status
  ping: BOOLEAN                  -- Test server connectivity
  last_error: STRING             -- Last error message
  has_error: BOOLEAN             -- Did last operation fail?

feature -- Database Operations
  database (a_name): SIMPLE_MONGO_DATABASE
  database_names: LIST [STRING]
  drop_database (a_name): BOOLEAN

feature -- Collection Access
  collection (a_db, a_name): SIMPLE_MONGO_COLLECTION
  use (a_db, a_collection)       -- Set default collection for convenience methods

feature -- Convenience CRUD (on default collection)
  insert (a_json): STRING        -- Returns inserted _id
  insert_many (a_json_array): LIST [STRING]
  find (a_query_json): LIST [STRING]
  find_one (a_query_json): detachable STRING
  count (a_query_json): INTEGER_64
  update (a_query_json, a_update_json): INTEGER  -- Returns modified count
  update_one (a_query_json, a_update_json): BOOLEAN
  delete (a_query_json): INTEGER  -- Returns deleted count
  delete_one (a_query_json): BOOLEAN
  find_and_modify (a_query_json, a_update_json): detachable STRING

feature -- Query Building
  query: SIMPLE_MONGO_QUERY      -- Start building a query

feature -- Index Management
  create_index (a_field, a_ascending): BOOLEAN
  drop_index (a_name): BOOLEAN
  list_indexes: LIST [STRING]
end

class SIMPLE_MONGO_COLLECTION

feature -- CRUD
  insert_one (a_json): detachable STRING
  insert_many (a_json_array): LIST [STRING]
  find (a_query, a_opts): SIMPLE_MONGO_CURSOR
  find_all: SIMPLE_MONGO_CURSOR
  find_one (a_query): detachable STRING
  count (a_query): INTEGER_64
  update_one (a_query, a_update): BOOLEAN
  update_many (a_query, a_update): INTEGER
  delete_one (a_query): BOOLEAN
  delete_many (a_query): INTEGER
  aggregate (a_pipeline_json): SIMPLE_MONGO_CURSOR
  drop: BOOLEAN

feature -- Status
  has_error: BOOLEAN
  last_error: STRING
  name: STRING
  database_name: STRING
end

class SIMPLE_MONGO_CURSOR

feature -- Iteration
  start                          -- Begin iteration
  item: detachable STRING        -- Current document as JSON
  forth                          -- Move to next
  after: BOOLEAN                 -- End of results?
  to_list: LIST [STRING]         -- Convert all to list

feature -- Status
  has_error: BOOLEAN
  error_message: STRING
end

class SIMPLE_MONGO_QUERY

feature -- Filters
  where (a_field, a_value): like Current
  where_eq (a_field, a_value): like Current
  where_ne (a_field, a_value): like Current
  where_gt (a_field, a_value): like Current
  where_gte (a_field, a_value): like Current
  where_lt (a_field, a_value): like Current
  where_lte (a_field, a_value): like Current
  where_in (a_field, a_values: ARRAY): like Current
  where_regex (a_field, a_pattern): like Current
  where_exists (a_field, a_exists: BOOLEAN): like Current

feature -- Logical
  and_ (a_query): like Current
  or_ (a_query): like Current
  not_ (a_query): like Current

feature -- Options
  sort (a_field, a_ascending): like Current
  limit (a_count): like Current
  skip (a_count): like Current
  project (a_fields: ARRAY [STRING]): like Current

feature -- Execution
  to_json: STRING                -- Get query as JSON
  execute: SIMPLE_MONGO_CURSOR   -- Run the query
  execute_one: detachable STRING -- Get first result
  count: INTEGER_64              -- Count matches
end

class SIMPLE_MONGO_JSON

feature -- Document Building
  make
  put_string (a_key, a_value)
  put_integer (a_key, a_value)
  put_double (a_key, a_value)
  put_boolean (a_key, a_value)
  put_null (a_key)
  put_array (a_key, a_values: ARRAY)
  put_object (a_key, a_value: SIMPLE_MONGO_JSON)
  put_date (a_key, a_date: DATE_TIME)
  put_object_id (a_key)          -- Auto-generate ObjectId
  to_json: STRING
end
```

### Step 7: Dependencies

**ISE Libraries:**
- `$ISE_LIBRARY/contrib/library/persistency/mongo/mongodb/mongodb.ecf`
- `$ISE_LIBRARY/contrib/library/persistency/mongo/bson/lib/bson.ecf`
- `$ISE_LIBRARY/library/base/base.ecf`
- `$ISE_LIBRARY/library/time/time.ecf`

**External Dependencies:**
- MongoDB C Driver (libmongoc) - Pre-built in `mongo/C/driver/`
- MongoDB server (for actual database operations)

**Key Classes Used:**
- MONGODB_DRIVER - Driver initialization
- MONGODB_CLIENT - Server connection
- MONGODB_COLLECTION - CRUD operations
- MONGODB_CURSOR - Result iteration
- MONGODB_DATABASE - Database-level operations
- BSON - Document representation
- BSON_OID - Object identifiers
- BSON_ERROR - Error information

---

## Implementation Summary

**Files to Create:**
- `simple_mongo.ecf` - ECF configuration
- `src/simple_mongo.e` - Main wrapper class
- `src/simple_mongo_collection.e` - Collection operations
- `src/simple_mongo_cursor.e` - Result iteration
- `src/simple_mongo_query.e` - Query builder
- `src/simple_mongo_json.e` - Document builder
- `testing/mongo_test_app.e` - Test application

**Test Strategy:**
1. **Unit tests (no server required):**
   - JSON/BSON conversion
   - Query builder construction
   - Document builder
   - URI parsing

2. **Integration tests (server required):**
   - Connection/disconnection
   - CRUD operations
   - Aggregation
   - Index operations

---

## Notes

1. **Server Requirement:** Unlike other simple_* wrappers, simple_mongo requires a running MongoDB server for most operations. The test suite should clearly separate unit tests from integration tests.

2. **C Driver Dependency:** The MongoDB library wraps the MongoDB C driver (libmongoc). Pre-built binaries are included in the ISE library for Windows.

3. **JSON-First API:** To simplify usage, our wrapper accepts JSON strings and returns JSON strings. This hides BSON complexity from users while maintaining full functionality.

4. **Error Handling:** All operations should set has_error and last_error for consistent error checking across the API.

5. **Driver Lifecycle:** MONGODB_DRIVER.use must be called before any operations. Our wrapper will handle this automatically in the connect/make feature.

6. **Thread Safety:** MongoDB cursors are NOT thread safe. Document this limitation clearly.

7. **Memory Management:** The C driver allocates memory that must be freed. The ISE wrapper handles this through Eiffel's garbage collection and dispose patterns.

