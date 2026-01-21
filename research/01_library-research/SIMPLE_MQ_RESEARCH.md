# simple_mq Research Notes

## Step 1: Specifications

### Message Queue Standards

**AMQP (Advanced Message Queuing Protocol) - ISO/IEC 19464:2014**
- Wire-level protocol for business messaging
- Features: message orientation, queuing, routing, reliability, security
- Exchange types: direct, fanout, topic, headers
- Message acknowledgements (ack/nack)
- Durable queues and persistent messages

**MQTT (Message Queuing Telemetry Transport) - ISO/IEC 20922:2016**
- Lightweight publish/subscribe protocol
- QoS levels: 0 (at most once), 1 (at least once), 2 (exactly once)
- Designed for constrained devices and low-bandwidth networks
- Retained messages and last will testament

**STOMP (Simple Text Oriented Messaging Protocol)**
- Text-based protocol (like HTTP for messaging)
- Commands: CONNECT, SEND, SUBSCRIBE, UNSUBSCRIBE, ACK, NACK
- Simple to implement, interoperable

### Core Concepts

**Queue**: Point-to-point messaging, one consumer per message
- FIFO ordering (typically)
- Competing consumers pattern
- Message persistence options

**Topic/Pub-Sub**: One-to-many broadcast
- Fan-out to all subscribers
- No message persistence (fire-and-forget)
- Topic hierarchies with wildcards

**Priority Queue**: Messages ordered by priority
- Higher priority = dequeued first
- Ties broken by timestamp (FIFO within priority)

Sources:
- [AMQP - Wikipedia](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol)
- [MQTT - Wikipedia](https://en.wikipedia.org/wiki/MQTT)
- [Message Queue System Design - GeeksforGeeks](https://www.geeksforgeeks.org/system-design/message-queues-system-design/)

---

## Step 2: Tech-Stack Library Analysis

### JavaScript - BullMQ
**Strengths:**
- Redis-backed, production-ready
- Priority queues, rate limiting, delayed jobs
- Repeatable jobs (cron-like)
- Sandboxed processors

**API Patterns:**
```javascript
const queue = new Queue('my-queue');
await queue.add('job', { data: 'hello' }, { priority: 1 });

const worker = new Worker('my-queue', async job => {
  return job.data;
});
```

### Python - Celery
**Strengths:**
- Distributed task queue
- Multiple broker support (Redis, RabbitMQ)
- Task chaining, groups, chords
- Monitoring with Flower

**Limitations:**
- Complex setup
- Heavy dependency footprint

### Python - Kombu
**Strengths:**
- Low-level messaging library
- Multiple transport backends
- Used by Celery internally

### Rust - Lapin (RabbitMQ client)
**Strengths:**
- Async/await native
- Full AMQP 0.9.1 support
- Connection pooling

### Go - NATS
**Strengths:**
- Extremely fast (10M+ messages/sec)
- At-most-once and at-least-once delivery
- JetStream for persistence
- Request/reply pattern built-in

### Redis Streams (Built into Redis 5.0+)
**Strengths:**
- Log-like data structure
- Consumer groups
- Message acknowledgement
- Persistence
- XADD, XREAD, XREADGROUP commands

Sources:
- [BullMQ - GitHub](https://github.com/taskforcesh/bullmq)
- [Benchmark of popular MQ packages](https://better.engineering/message-queues)
- [Dissecting Message Queues](https://bravenewgeek.com/dissecting-message-queues/)

---

## Step 3: Eiffel Ecosystem

### Existing Solutions
**No dedicated message queue library found** in the Eiffel ecosystem:
- EiffelBase: QUEUE class exists (FIFO dispenser), but not message-oriented
- Gobo: Data structures, no messaging
- SCOOP: Concurrency model, but not messaging abstraction

### EiffelBase QUEUE
- Deferred class QUEUE describes general queues
- FIFO internal policy
- Three non-deferred heirs with different representations
- No pub/sub, no message headers, no persistence

### Distributed Eiffel (Historical)
- Academic project from 1990s
- Built on Clouds operating system
- Both shared-memory and message-passing models
- Not maintained or available

### Gap Analysis
Major opportunity - Eiffel lacks:
- Message abstraction with headers/metadata
- Pub/sub topics with fan-out
- Priority queues for messages
- Redis-backed distributed queues
- Wildcard topic matching

Sources:
- [EiffelBase Dispensers](https://www.eiffel.org/doc/solutions/EiffelBase%2C_Dispensers)
- [Distributed Eiffel - IEEE](https://ieeexplore.ieee.org/document/185497/)

---

## Step 4: Developer Pain Points

### Message Delivery Guarantees
1. **At-least-once vs exactly-once confusion**
   - NSQ delivers messages more than once, developers must handle idempotency
   - No standard pattern for deduplication

2. **Message ordering not guaranteed**
   - Many MQs don't guarantee order
   - Priority queues complicate ordering further

### Reliability Issues
1. **Fire-and-forget risks**
   - Redis Pub/Sub has no persistence - disconnected subscribers miss messages
   - NSQ is primarily in-memory, no built-in replication

2. **Message loss scenarios**
   - Server crashes before acknowledgement
   - Network partitions during delivery

### Configuration Complexity
1. **Steep learning curve**
   - Kafka requires experts, no official management UI
   - RabbitMQ config is in Erlang

2. **Platform lock-in**
   - MSMQ is Windows-only (deprecated)
   - Some features are commercial-only

### Client Library Challenges
1. **Writing clients is hard**
   - At-least-once delivery must be implemented in clients
   - High availability requires custom failover code

2. **API inconsistencies**
   - Cannot publish directly to queue by name (RabbitMQ)
   - Cannot fan-out to multiple readers of same queue
   - Cannot dynamically add/remove queue subscriptions

### Performance Trade-offs
1. **Speed vs durability**
   - Redis is fast but loses unread messages on crash
   - RabbitMQ persistent mode is slow (disk writes + acknowledgements)

2. **Large messages problematic**
   - Redis has notable latency for messages > 1MB

### What Developers Want
1. **Simple API** for common cases (send/receive)
2. **Configurable guarantees** (speed vs reliability)
3. **Easy persistence** when needed
4. **Message history** for late-joining subscribers
5. **Wildcard routing** for flexible subscriptions
6. **Priority support** out of the box

Sources:
- [RabbitMQ Issues 2024 - Medium](https://medium.com/@oresoftware/rabbitmq-is-garbage-58d936d10104)
- [AWS: RabbitMQ vs Redis](https://aws.amazon.com/compare/the-difference-between-rabbitmq-and-redis/)
- [Hacker News: MQ architectures](https://news.ycombinator.com/item?id=40723302)

---

## Step 5: Innovation Opportunities

### simple_mq Differentiators

1. **Contract-Based Message Safety**
```eiffel
enqueue (a_message: SIMPLE_MQ_MESSAGE)
    require
        message_attached: a_message /= Void
        not_full: not is_full
    ensure
        count_increased: count = old count + 1
        message_in_queue: has_message (a_message.id)
```

2. **Unified Queue + Pub/Sub API**
- Both patterns in one library
- Clear naming: `SIMPLE_MQ_QUEUE` vs `SIMPLE_MQ_TOPIC`
- Same message type works in both

3. **Message History for Topics**
```eiffel
topic := mq.new_topic_with_history ("events", 100)
-- Late subscribers can get recent history
recent := topic.get_history_since (timestamp)
```

4. **Priority Queue Built-in**
```eiffel
queue.enqueue_with_priority (msg, 10)  -- High priority
queue.enqueue_with_priority (msg2, 1)  -- Low priority
-- Dequeue returns highest priority first
```

5. **Wildcard Topic Matching**
```eiffel
topic.matches_pattern ("events.user.*")   -- Single level
topic.matches_pattern ("events.#")        -- Multi-level
```

6. **Redis Backend for Distribution**
```eiffel
-- In-memory (single process)
queue := mq.new_queue ("tasks")

-- Redis-backed (distributed, persistent)
redis_queue := mq.new_redis_queue ("jobs", redis)
```

7. **Message Metadata**
```eiffel
msg.set_header ("content-type", "application/json")
msg.set_priority (5)
msg.set_correlation_id (request_id)
```

8. **SCOOP-Ready Design**
- Thread-safe queue operations
- No shared mutable state between subscribers

---

## Step 6: Design Strategy

### Core Design Principles
- **Simple**: Send/receive in 2 lines of code
- **Flexible**: In-memory or Redis-backed
- **Safe**: Contracts prevent invalid operations
- **Complete**: Queue + Topic + Priority in one library

### API Surface

#### SIMPLE_MQ_MESSAGE
```eiffel
class SIMPLE_MQ_MESSAGE

create
    make,           -- Auto-generate UUID
    make_with_id    -- Specific ID

feature -- Access
    id: STRING
    payload: STRING
    timestamp: INTEGER_64
    priority: INTEGER
    headers: HASH_TABLE [STRING, STRING]

feature -- Modification
    set_priority (p: INTEGER)
    set_header (key, value: STRING)
    set_correlation_id (id: STRING)
```

#### SIMPLE_MQ_QUEUE
```eiffel
class SIMPLE_MQ_QUEUE

create
    make,               -- Unbounded FIFO
    make_with_capacity, -- Bounded
    make_priority       -- Priority ordering

feature -- Access
    name: STRING
    count: INTEGER
    is_empty, is_full: BOOLEAN
    is_priority_queue: BOOLEAN

feature -- Operations
    enqueue (msg: SIMPLE_MQ_MESSAGE)
    dequeue: detachable SIMPLE_MQ_MESSAGE
    dequeue_batch (n: INTEGER): LIST [SIMPLE_MQ_MESSAGE]
    peek: detachable SIMPLE_MQ_MESSAGE
    remove_by_id (id: STRING): BOOLEAN
```

#### SIMPLE_MQ_TOPIC
```eiffel
class SIMPLE_MQ_TOPIC

create
    make,               -- No history
    make_with_history   -- Retain N messages

feature -- Access
    name: STRING
    subscriber_count: INTEGER
    history_size: INTEGER

feature -- Pub/Sub
    subscribe (handler: SIMPLE_MQ_SUBSCRIBER)
    unsubscribe (handler: SIMPLE_MQ_SUBSCRIBER)
    publish (msg: SIMPLE_MQ_MESSAGE)

feature -- History
    get_history: LIST [SIMPLE_MQ_MESSAGE]
    get_history_since (timestamp: INTEGER_64): LIST [SIMPLE_MQ_MESSAGE]

feature -- Matching
    matches_pattern (pattern: STRING): BOOLEAN
```

#### SIMPLE_MQ (Facade)
```eiffel
class SIMPLE_MQ

feature -- Factory
    new_message (payload: STRING): SIMPLE_MQ_MESSAGE
    new_queue (name: STRING): SIMPLE_MQ_QUEUE
    new_priority_queue (name: STRING): SIMPLE_MQ_QUEUE
    new_topic (name: STRING): SIMPLE_MQ_TOPIC
    new_redis_queue (name: STRING; redis: SIMPLE_REDIS): SIMPLE_MQ_REDIS_QUEUE

feature -- Registry
    register_queue (queue: SIMPLE_MQ_QUEUE)
    queue (name: STRING): detachable SIMPLE_MQ_QUEUE
    topics_matching (pattern: STRING): LIST [SIMPLE_MQ_TOPIC]

feature -- Quick Operations
    send (queue_name, payload: STRING)
    receive (queue_name: STRING): detachable SIMPLE_MQ_MESSAGE
    publish (topic_name, payload: STRING)
```

### Contract Strategy

**Queue Preconditions:**
```eiffel
enqueue (msg: SIMPLE_MQ_MESSAGE)
    require
        not_full: not is_full

dequeue: detachable SIMPLE_MQ_MESSAGE
    -- No precondition (returns Void if empty)
```

**Topic Preconditions:**
```eiffel
subscribe (handler: SIMPLE_MQ_SUBSCRIBER)
    require
        not_already_subscribed: not is_subscribed (handler)
```

**Message Invariants:**
```eiffel
invariant
    id_not_empty: not id.is_empty
    timestamp_valid: timestamp > 0
```

### Integration Plan
- Add to SERVICE_API: `new_mq`, `new_mq_message`, `new_mq_queue`, etc.
- Redis integration via SIMPLE_REDIS from simple_cache
- No external dependencies beyond simple_cache for Redis

---

## Step 7: Implementation Assessment

### Current simple_mq Status

**What's Implemented:**
- SIMPLE_MQ_MESSAGE: id, payload, timestamp, priority, headers, correlation_id
- SIMPLE_MQ_QUEUE: make, make_with_capacity, make_priority, enqueue, dequeue, dequeue_batch, peek, remove_by_id, statistics
- SIMPLE_MQ_TOPIC: make, make_with_history, subscribe, unsubscribe, publish, history, wildcard matching
- SIMPLE_MQ_SUBSCRIBER: deferred subscriber interface
- SIMPLE_MQ_REDIS_QUEUE: Redis-backed queue using LPUSH/RPOP
- SIMPLE_MQ (Facade): All factory methods, registry, quick operations

**What's Well Done (Based on Research):**
1. **Message history** - Solves fire-and-forget problem for late subscribers
2. **Priority queues** - Built-in, not an afterthought
3. **Wildcard matching** - Supports both * (single) and # (multi-level)
4. **Bounded queues** - Capacity limits prevent memory exhaustion
5. **Redis integration** - Distributed/persistent option available
6. **Strong contracts** - Preconditions on full queues, postconditions on counts

**What's Missing (Based on Research):**
1. **Dead letter queue** - No handling for failed messages
2. **Message TTL** - No automatic expiration
3. **Acknowledgement mode** - No explicit ack/nack for at-least-once
4. **Consumer groups** - No competing consumers for topics
5. **Request/reply pattern** - No correlation ID handling
6. **Delayed messages** - No scheduled delivery

**Contract Gaps:**
- Missing: `is_subscribed` check could be stronger
- Missing: Invariant on message ID uniqueness per queue

### Recommendations

1. **Add message TTL** - Auto-expire old messages
2. **Add dead letter queue** - Route failed messages for inspection
3. **Consider ack mode** - Optional acknowledgement for reliability
4. **Add delayed messages** - Schedule future delivery
5. **Document thread safety** - SCOOP compatibility notes

### Comparison to Research Findings

| Feature | Research Priority | simple_mq Status |
|---------|------------------|------------------|
| Simple API | High | Implemented |
| Queue + Pub/Sub | High | Implemented |
| Priority Queue | Medium | Implemented |
| Message History | Medium | Implemented |
| Wildcard Topics | Medium | Implemented |
| Redis Backend | Medium | Implemented |
| Message TTL | Medium | NOT implemented |
| Dead Letter | Low | NOT implemented |
| Consumer Groups | Low | NOT implemented |
| Delayed Messages | Low | NOT implemented |

---

## Checklist

- [x] Formal specifications reviewed (AMQP, MQTT, STOMP)
- [x] Top libraries studied (BullMQ, Celery, NATS, Redis Streams)
- [x] Eiffel ecosystem researched (gap identified)
- [x] Developer pain points documented
- [x] Innovation opportunities identified
- [x] Design strategy synthesized
- [x] Implementation assessment completed
- [ ] Missing features implemented (TTL, dead letter)
- [ ] Contracts strengthened

