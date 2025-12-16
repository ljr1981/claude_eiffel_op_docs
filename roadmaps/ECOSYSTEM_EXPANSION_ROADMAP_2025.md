# Simple Eiffel Ecosystem Expansion Roadmap 2025

**Created:** 2025-12-13
**Status:** APPROVED
**Oracle Reference:** Query "roadmap" or "priority matrix"

---

## Executive Summary

Based on competitive analysis of Rust, Go, C#/.NET, Python, and Java/Spring ecosystems, this document defines the expansion plan for simple_* libraries to achieve feature parity with modern development platforms.

**Current State:** 62 libraries
**Target State:** 68 libraries (6 new + 4 extended)

---

## Table of Contents

1. [Priority Matrix](#1-priority-matrix)
2. [Phase 1: Quick Wins](#2-phase-1-quick-wins)
3. [Phase 2: Core Infrastructure](#3-phase-2-core-infrastructure)
4. [Phase 3: Advanced Capabilities](#4-phase-3-advanced-capabilities)
5. [Detailed Specifications](#5-detailed-specifications)
6. [Dependencies](#6-dependencies)
7. [Success Metrics](#7-success-metrics)

---

## 1. Priority Matrix

### Criticality vs Ease of Implementation

```
                        EASE OF IMPLEMENTATION
                    HARD ←────────────────→ EASY
                      │
              HIGH    │  ┌─────────────────────────────────────┐
                ▲     │  │ QUADRANT 2        │ QUADRANT 1      │
                │     │  │ INVEST EFFORT     │ DO FIRST        │
                │     │  │                   │                 │
           C    │     │  │ • telemetry       │ • web+resilience│
           R    │     │  │ • mq              │ • cache+redis   │
           I    │     │  │ • grpc            │ • sql+orm       │
           T    │     │  │                   │                 │
           I    │     │  ├───────────────────┼─────────────────┤
           C    │     │  │ QUADRANT 4        │ QUADRANT 3      │
           A    │     │  │ DEFER             │ QUICK WINS      │
           L    │     │  │                   │                 │
           I    │     │  │ • scheduler       │ • graph         │
           T    │     │  │                   │ • math          │
           Y    │     │  │                   │ • service+med   │
              LOW     │  └─────────────────────────────────────┘
```

### Ranked Priority List

| Rank | ID | Library | Type | Critical | Easy | Score | Phase |
|------|----|---------|------|----------|------|-------|-------|
| 1 | WEB-RES | simple_web + Resilience | EXTEND | 9/10 | 8/10 | 17 | 1 |
| 2 | CACHE-RED | simple_cache + Redis | EXTEND | 7/10 | 7/10 | 14 | 1 |
| 3 | GRAPH | simple_graph | NEW | 5/10 | 8/10 | 13 | 1 |
| 4 | MATH | simple_math | NEW | 5/10 | 7/10 | 12 | 1 |
| 5 | SQL-ORM | simple_sql + ORM | EXTEND | 8/10 | 6/10 | 14 | 2 |
| 6 | MQ | simple_mq | NEW | 8/10 | 5/10 | 13 | 2 |
| 7 | SVC-MED | simple_service_api + Mediator | EXTEND | 5/10 | 7/10 | 12 | 2 |
| 8 | TELEM | simple_telemetry | NEW | 9/10 | 4/10 | 13 | 3 |
| 9 | SCHED | simple_scheduler | NEW | 6/10 | 4/10 | 10 | 3 |
| 10 | GRPC | simple_grpc | NEW | 7/10 | 3/10 | 10 | 3 |

---

## 2. Phase 1: Quick Wins

**Timeline:** First priority
**Goal:** Immediate production value with minimal complexity

### 2.1 WEB-RES: simple_web + Resilience Middleware

**Oracle ID:** WEB-RES
**Type:** EXTEND existing library
**Effort:** 1-2 weeks

**Purpose:** Add retry, circuit breaker, timeout, bulkhead, fallback patterns to simple_web client/server.

**Capabilities:**
- Retry with exponential backoff (1s, 2s, 4s, 8s...)
- Circuit breaker (after N failures, stop trying for cooldown period)
- Timeout (don't wait forever for slow services)
- Bulkhead (limit concurrent calls to protect resources)
- Fallback (return cached/default value when service unavailable)

**API Design:**
```eiffel
client: SIMPLE_WEB_CLIENT

-- Configure resilience per-endpoint
client.with_resilience (
    create {SIMPLE_RESILIENCE_POLICY}
        .with_retry (max_attempts: 3, backoff: exponential)
        .with_circuit_breaker (failure_threshold: 5, cooldown_seconds: 30)
        .with_timeout (seconds: 10)
        .with_fallback (agent cached_response)
)

-- All requests through this client are now protected
response := client.get ("https://flaky-service.com/api")

-- Server-side middleware
router.use (create {SIMPLE_WEB_RATE_LIMIT_MIDDLEWARE}.make (100, per_minute))
router.use (create {SIMPLE_WEB_CIRCUIT_BREAKER_MIDDLEWARE}.make (downstream_services))
```

**Why Easy:** Middleware pattern already exists in simple_web. Pure Eiffel, no external dependencies. State machine logic is straightforward.

**Why Critical:** Production systems need this NOW. Without resilience, one slow service cascades failures everywhere.

**Files to Create/Modify:**
- `simple_web/src/middleware/simple_web_resilience_middleware.e`
- `simple_web/src/client/simple_resilience_policy.e`
- `simple_web/src/client/simple_circuit_breaker.e`
- `simple_web/src/client/simple_retry_policy.e`

**Tests Required:**
- Circuit breaker opens after failures
- Circuit breaker half-opens after cooldown
- Retry with backoff timing
- Fallback returns correct value
- Timeout fires correctly

---

### 2.2 CACHE-RED: simple_cache + Redis Backend

**Oracle ID:** CACHE-RED
**Type:** EXTEND existing library
**Effort:** 1-2 weeks

**Purpose:** Add Redis connection, data types, pub/sub, distributed locking to existing simple_cache.

**Capabilities:**
- Redis connection over TCP
- All Redis data types: strings, hashes, lists, sets, sorted sets
- Pub/Sub for real-time messaging
- Distributed locking for coordination
- TTL management with Redis native expiry

**API Design:**
```eiffel
cache: SIMPLE_CACHE

-- Switch to Redis backend
cache.use_redis ("redis://localhost:6379")

-- Basic caching (existing API, now with Redis)
cache.put ("user:123", user.to_json, ttl_minutes: 30)
cached := cache.get ("user:123")

-- Redis-specific: Hash operations
cache.redis.hash_set ("session:abc", "user_id", "123")
cache.redis.hash_set ("session:abc", "role", "admin")
all_fields := cache.redis.hash_get_all ("session:abc")

-- Redis-specific: Pub/Sub
cache.redis.subscribe ("notifications", agent handle_notification)
cache.redis.publish ("notifications", "New order received")

-- Distributed lock
lock := cache.redis.acquire_lock ("order:456:processing", timeout_seconds: 30)
if lock.acquired then
    process_order (456)
    lock.release
end
```

**Why Easy:** Redis protocol is simple text over TCP. Example:
```
SET mykey "Hello"\r\n  →  +OK\r\n
GET mykey\r\n          →  $5\r\nHello\r\n
```
TCP sockets already work in Eiffel. RESP protocol is ~100 lines to implement.

**Why Critical:** Enables shared cache across processes/servers. Foundation for simple_mq (Redis Streams). Every production system needs distributed caching.

**Files to Create/Modify:**
- `simple_cache/src/backends/simple_cache_redis_backend.e`
- `simple_cache/src/redis/simple_redis_connection.e`
- `simple_cache/src/redis/simple_redis_commands.e`
- `simple_cache/src/redis/simple_redis_pubsub.e`
- `simple_cache/src/redis/simple_redis_lock.e`

**Dependencies:**
- TCP socket support (exists in Eiffel base)
- RESP protocol parser (new, ~100 lines)

---

### 2.3 GRAPH: simple_graph

**Oracle ID:** GRAPH
**Type:** NEW library (wrap ISE graph)
**Effort:** 1 week

**Purpose:** Graph data structures and algorithms for dependencies, workflows, networks, routing.

**Capabilities:**
- Graph types: directed, undirected, weighted, DAG
- Classic algorithms: BFS, DFS, Dijkstra, A*, topological sort, cycle detection
- Network analysis: connected components, centrality
- Dependency resolution for build systems, package managers

**API Design:**
```eiffel
graph: SIMPLE_GRAPH [STRING]  -- Generic node type

-- Build graph
graph.add_node ("A")
graph.add_node ("B")
graph.add_edge ("A", "B", weight: 5)
graph.add_edge ("B", "C", weight: 3)
graph.add_edge ("A", "C", weight: 10)

-- Shortest path (Dijkstra)
path := graph.shortest_path ("A", "C")
-- Result: ["A", "B", "C"], total_cost: 8

-- Topological sort (for dependencies)
build_order := graph.topological_sort
-- Result: ["C", "B", "A"] (reverse dependency order)

-- Cycle detection
if graph.has_cycle then
    print ("Circular dependency detected!")
end

-- BFS traversal
reachable := graph.bfs_from ("A")
-- Result: ["A", "B", "C"]

-- Connected components
components := graph.connected_components
```

**Why Easy:** Wrap ISE graph library (`$ISE_LIBRARY/library/graph`). Pure algorithms, no I/O, no external dependencies. Well-understood computer science.

**Why Useful:** Dependencies (build systems, package managers), workflows (task ordering), networks (routing), social graphs. Graphs are everywhere but painful to implement correctly.

**Files to Create:**
- `simple_graph/src/simple_graph.e` (facade)
- `simple_graph/src/simple_directed_graph.e`
- `simple_graph/src/simple_weighted_graph.e`
- `simple_graph/src/algorithms/simple_graph_bfs.e`
- `simple_graph/src/algorithms/simple_graph_dijkstra.e`
- `simple_graph/src/algorithms/simple_graph_topological_sort.e`

---

### 2.4 MATH: simple_math

**Oracle ID:** MATH
**Type:** NEW library (wrap ISE eapml)
**Effort:** 1 week

**Purpose:** Scientific and arbitrary precision computing beyond floating-point.

**Capabilities:**
- Arbitrary precision integers (thousands of digits)
- Arbitrary precision decimals (exact financial math)
- Matrix operations (linear algebra)
- Statistical functions (mean, median, std_dev, distributions)

**API Design:**
```eiffel
math: SIMPLE_MATH

-- Arbitrary precision integers
big_int := math.big_integer ("123456789012345678901234567890")
result := big_int * big_int  -- No overflow, exact result

-- Precise decimals (financial calculations)
price := math.decimal ("19.99")
tax := math.decimal ("0.0825")
total := price * (math.one + tax)  -- Exact: 21.6391750

-- Matrix operations
matrix := math.matrix_3x3 (<<1,2,3>, <4,5,6>, <7,8,9>>)
determinant := matrix.determinant
inverse := matrix.inverse
product := matrix * other_matrix

-- Statistics
data := <<1.5, 2.3, 4.1, 3.8, 2.9>>
print (math.mean (data))       -- 2.92
print (math.median (data))     -- 2.9
print (math.std_dev (data))    -- 0.97
print (math.variance (data))   -- 0.94
```

**Why Easy:** Wrap ISE eapml library (`$ISE_LIBRARY/contrib/library/math/eapml`). Pure computation, no I/O. Mathematical operations are well-defined.

**Why Useful:** simple_decimal handles business math but not scientific computing. Cryptography, physics simulations, ML preprocessing, financial modeling all need this.

**Files to Create:**
- `simple_math/src/simple_math.e` (facade)
- `simple_math/src/simple_big_integer.e`
- `simple_math/src/simple_big_decimal.e`
- `simple_math/src/simple_matrix.e`
- `simple_math/src/simple_statistics.e`

---

## 3. Phase 2: Core Infrastructure

**Timeline:** After Phase 1 complete
**Goal:** Enable microservices architecture patterns

### 3.1 SQL-ORM: simple_sql + ORM Layer

**Oracle ID:** SQL-ORM
**Type:** EXTEND existing library
**Effort:** 2-3 weeks

**Purpose:** Add entity mapping, query builder, migrations to existing simple_sql.

**Capabilities:**
- Entity mapping: Eiffel classes map to tables automatically
- Query builder: Type-safe queries without string concatenation
- Migrations: Version-controlled schema changes
- Relationships: has_many, belongs_to, many_to_many
- Unit of Work: Track changes, batch commits

**API Design:**
```eiffel
-- Entity definition
class ORDER inherit SIMPLE_ENTITY

feature -- Mapping
    table_name: STRING = "orders"

feature -- Columns
    id: INTEGER
    customer_name: STRING
    total: DECIMAL
    created_at: DATE_TIME

feature -- Relationships
    items: LIST [ORDER_ITEM]  -- has_many :items
    customer: CUSTOMER        -- belongs_to :customer

end

-- Usage
db: SIMPLE_SQL_ORM

-- Query builder (type-safe)
orders := db.query (ORDER)
    .where ("total > ?", 100)
    .where ("created_at > ?", yesterday)
    .order_by ("created_at", descending)
    .include ("items")  -- eager load relationship
    .include ("customer")
    .limit (10)
    .offset (20)
    .execute

-- Iterate results
across orders as o loop
    print (o.item.customer_name)
    across o.item.items as i loop
        print (i.item.product_name)
    end
end

-- Create/Update
create order.make
order.customer_name := "John Doe"
order.total := decimal ("99.99")
db.save (order)  -- INSERT if new, UPDATE if exists

-- Migrations
db.migrate_to (5)  -- Apply migrations 1 through 5
```

**Files to Create/Modify:**
- `simple_sql/src/orm/simple_entity.e`
- `simple_sql/src/orm/simple_sql_orm.e`
- `simple_sql/src/orm/simple_query_builder.e`
- `simple_sql/src/orm/simple_migration.e`
- `simple_sql/src/orm/simple_relationship.e`

---

### 3.2 MQ: simple_mq

**Oracle ID:** MQ
**Type:** NEW library
**Effort:** 2-3 weeks

**Purpose:** Message queue and event bus for asynchronous service communication.

**Capabilities:**
- Publish/Subscribe: Broadcast events to multiple consumers
- Work Queues: Distribute tasks across workers
- Event Sourcing: Store events as source of truth
- Backends: Redis Streams first, then RabbitMQ, Kafka adapters

**API Design:**
```eiffel
mq: SIMPLE_MQ

-- Connect (Redis Streams backend)
mq.connect ("redis://localhost:6379")

-- Publish event
mq.publish ("orders", order.to_json)
mq.publish ("orders", order.to_json, headers: <<"priority", "high">>)

-- Subscribe (in another process/service)
mq.subscribe ("orders", agent handle_order)
mq.subscribe ("orders.*", agent handle_all_order_events)  -- wildcard

-- Work queue with acknowledgment
mq.queue ("pdf_jobs").push (job.to_json)

-- Worker process
task := mq.queue ("pdf_jobs").pop_blocking (timeout_seconds: 30)
if attached task then
    process_pdf (task.payload)
    task.acknowledge  -- Remove from queue
    -- or task.reject to retry later
end

-- Consumer groups (multiple workers share load)
mq.queue ("emails").join_group ("email_workers")
task := mq.queue ("emails").pop_from_group ("email_workers")
```

**Why Depends on CACHE-RED:** Uses Redis Streams which requires Redis connection from simple_cache.

**Files to Create:**
- `simple_mq/src/simple_mq.e` (facade)
- `simple_mq/src/simple_mq_message.e`
- `simple_mq/src/simple_mq_queue.e`
- `simple_mq/src/backends/simple_mq_redis_backend.e`
- `simple_mq/src/simple_mq_consumer_group.e`

---

### 3.3 SVC-MED: simple_service_api + Mediator

**Oracle ID:** SVC-MED
**Type:** EXTEND existing library
**Effort:** 1-2 weeks

**Purpose:** Add mediator pattern, command/query separation, pipeline behaviors.

**Capabilities:**
- Mediator pattern: Route commands/queries to handlers without coupling
- CQRS: Commands change state, queries read state
- Pipeline behaviors: Cross-cutting concerns as middleware
- Event dispatching: Publish domain events after commands

**API Design:**
```eiffel
-- Command definition
class CREATE_ORDER_COMMAND inherit SIMPLE_COMMAND [ORDER]
feature
    customer_id: INTEGER
    items: LIST [ORDER_ITEM_DTO]
end

-- Handler definition
class CREATE_ORDER_HANDLER inherit SIMPLE_COMMAND_HANDLER [CREATE_ORDER_COMMAND, ORDER]
feature
    handle (cmd: CREATE_ORDER_COMMAND): ORDER
        do
            create Result.make (cmd.customer_id, cmd.items)
            repository.save (Result)
            -- Publish domain event
            mediator.publish (create {ORDER_CREATED_EVENT}.make (Result))
        end
end

-- Usage (controller doesn't know handler)
mediator: SIMPLE_MEDIATOR

create cmd.make (customer_id, items)
order := mediator.send (cmd)  -- Automatically routes to CREATE_ORDER_HANDLER

-- Pipeline behaviors (applied to ALL commands)
mediator.add_behavior (create {LOGGING_BEHAVIOR})      -- Log all commands
mediator.add_behavior (create {VALIDATION_BEHAVIOR})   -- Validate before handle
mediator.add_behavior (create {TRANSACTION_BEHAVIOR})  -- Wrap in transaction
```

**Files to Create/Modify:**
- `simple_service_api/src/mediator/simple_mediator.e`
- `simple_service_api/src/mediator/simple_command.e`
- `simple_service_api/src/mediator/simple_query.e`
- `simple_service_api/src/mediator/simple_pipeline_behavior.e`

---

## 4. Phase 3: Advanced Capabilities

**Timeline:** After Phase 2 complete
**Goal:** Full microservices platform capabilities

### 4.1 TELEM: simple_telemetry

**Oracle ID:** TELEM
**Type:** NEW library
**Effort:** 4-6 weeks

**Purpose:** Observability platform with distributed tracing, metrics, and log correlation.

**Capabilities:**
- Distributed tracing: Track requests across services
- Metrics: Counters, gauges, histograms
- Log correlation: Tag logs with trace_id/span_id
- OpenTelemetry Protocol (OTLP): Industry standard export

**API Design:**
```eiffel
tracer: SIMPLE_TRACER
metrics: SIMPLE_METRICS

-- Start trace span
span := tracer.start_span ("process_order")
span.set_attribute ("order_id", order.id)
span.set_attribute ("customer", customer.name)

-- Child span (auto-nests)
db_span := tracer.start_span ("database_query")
-- ... execute query ...
db_span.finish

span.set_status_ok
span.finish

-- Metrics
metrics.counter ("orders_processed").increment
metrics.counter ("orders_processed").increment_by (5)
metrics.gauge ("active_connections").set (current_connections)
metrics.histogram ("order_value").record (order.total)
metrics.histogram ("response_time_ms").record (elapsed_ms)

-- Export to backends
telemetry.export_to_otlp ("http://collector:4317")
telemetry.export_to_prometheus (port: 9090)
```

**Why Hard:** OTLP protocol is complex (protobuf over gRPC or HTTP). Context propagation across SCOOP regions is tricky. Need to instrument existing libraries.

**Why Critical:** Cannot debug production microservices without observability. "Why is this slow?" becomes answerable.

---

### 4.2 SCHED: simple_scheduler

**Oracle ID:** SCHED
**Type:** NEW library
**Effort:** 3-4 weeks

**Purpose:** Background jobs, delayed execution, recurring tasks.

**Capabilities:**
- Delayed execution: "Run in 24 hours"
- Recurring jobs: Cron syntax support
- Persistent job storage: Survives restarts
- Distributed: Multiple workers
- Retry on failure with backoff

**API Design:**
```eiffel
scheduler: SIMPLE_SCHEDULER
scheduler.connect_storage (simple_sql_connection)

-- Delayed job
scheduler.schedule_once (
    agent send_reminder_email (order),
    delay_hours (24)
)

-- Recurring job (cron syntax)
scheduler.schedule_recurring (
    "daily_report",
    agent generate_daily_report,
    cron ("0 9 * * *")  -- 9am daily
)

-- Immediate background job
scheduler.enqueue (agent process_large_file (file_path))

-- Worker process
scheduler.work_loop  -- Blocks, processes jobs forever
```

---

### 4.3 GRPC: simple_grpc

**Oracle ID:** GRPC
**Type:** NEW library
**Effort:** 6-8 weeks

**Purpose:** High-performance binary RPC with streaming support.

**Capabilities:**
- Protocol Buffers: Binary serialization
- HTTP/2: Multiplexed connections
- Streaming: Server, client, bidirectional
- Code generation: .proto to Eiffel

**API Design:**
```eiffel
-- Generated from .proto file
client: GREETER_CLIENT
client.connect ("localhost:50051")

-- Unary call
request: HELLO_REQUEST
request.set_name ("Larry")
response := client.say_hello (request)
print (response.message)  -- "Hello, Larry!"

-- Server streaming
across client.stream_updates (request) as update loop
    print (update.item.status)
end

-- Bidirectional streaming
stream := client.chat
stream.send (message1)
stream.send (message2)
across stream.responses as resp loop
    print (resp.item.text)
end
```

**Why Hardest:** Requires protobuf parser, HTTP/2 framing, code generator tool. Significant complexity.

**Why Important:** Service mesh standard. What Kubernetes, Envoy, and cloud-native tools expect.

---

## 5. Detailed Specifications

Each item above includes:
- **Oracle ID**: For cross-referencing with oracle todo list
- **Type**: NEW or EXTEND
- **Effort**: Estimated time
- **Capabilities**: Feature list
- **API Design**: Code examples
- **Files**: What to create/modify
- **Dependencies**: What must exist first
- **Tests**: What to verify

---

## 6. Dependencies

```
Phase 1 (no dependencies):
  WEB-RES ──────────────────────────────────┐
  GRAPH ────────────────────────────────────┤
  MATH ─────────────────────────────────────┤
  CACHE-RED ────────────────────────────────┤
                                            │
Phase 2 (depends on Phase 1):               │
  SQL-ORM ──────────────────────────────────┤
  MQ ──────────── requires CACHE-RED ───────┤
  SVC-MED ──────────────────────────────────┤
                                            │
Phase 3 (depends on Phase 2):               │
  TELEM ────────────────────────────────────┤
  SCHED ──────── can use MQ for queues ─────┤
  GRPC ─────────────────────────────────────┘
```

---

## 7. Success Metrics

### Per-Library Metrics
- [ ] All tests pass
- [ ] Documentation complete
- [ ] Example code works
- [ ] Integrated into simple_showcase

### Ecosystem Metrics
- [ ] 68 libraries total (from 62)
- [ ] All god script tests pass
- [ ] Oracle updated with each completion

---

## 8. Pre-Build Research Process

**Reference:** `/d/prod/reference_docs/claude/simple_library_design_process.md`

Before implementing ANY library from this roadmap, follow the Seven-Step Process:

### Step 1: Deep Web Research - Specifications
- RFCs relevant to the domain
- ECMA, ISO, IEEE formal specifications
- W3C recommendations (for web-related)
- Industry standards documentation

### Step 2: Research Latest Tech-Stack Libraries
- JavaScript/npm ecosystem leaders
- Python/PyPI top packages
- Rust/crates.io popular crates
- Go standard library and packages
- Java Apache Commons, Guava patterns
- C#/.NET BCL and NuGet packages

### Step 3: Research Eiffel Ecosystem
- Gobo library suite
- Eiffel Loop (Finnian Reilly)
- ISE EiffelStudio libraries
- Historical Eiffel libraries

### Step 4: Research Developer Pain Points
- Stack Overflow questions/complaints
- GitHub issues on popular libraries
- Reddit discussions
- Blog posts about library frustrations

### Step 5: Innovation Hat
- What would make developers' lives easier?
- What patterns could be simplified?
- What contracts prevent common errors?
- What convenience methods address 80% of use cases?

### Step 6: Synthesize Design Strategy
- Core design principles
- API surface design
- Contract strategy
- Integration plan
- Test strategy

### Step 7: Design/Implement/Strategy Report
- Executive summary
- Research findings
- Competitive analysis
- API design specifications
- Contract specifications
- Implementation roadmap
- Test plan

**Output:** Research report in `/d/prod/reference_docs/research/[library_name]_research.md`

---

## Appendix: Competitive Gap Coverage

| Gap | Covered By | Phase |
|-----|------------|-------|
| Observability/Tracing | simple_telemetry | 3 |
| Message Queues | simple_mq | 2 |
| Resilience Patterns | simple_web + Resilience | 1 |
| gRPC | simple_grpc | 3 |
| ORM | simple_sql + ORM | 2 |
| Redis | simple_cache + Redis | 1 |
| Background Jobs | simple_scheduler | 3 |
| CQRS/Mediator | simple_service_api + Mediator | 2 |
| Scientific Computing | simple_math | 1 |
| Graph Algorithms | simple_graph | 1 |

---

**Document Version:** 1.0
**Last Updated:** 2025-12-13
**Author:** Claude Code + Larry Rix
