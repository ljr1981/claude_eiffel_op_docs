# Mediator Extension Research Notes

## Step 1: Specifications

### Mediator Pattern (GoF)
The Mediator pattern defines an object that encapsulates how a set of objects interact. It promotes loose coupling by keeping objects from referring to each other explicitly.

**Core Concepts:**
- Mediator: Interface defining communication protocol
- Concrete Mediator: Implements coordination logic
- Colleagues: Objects that communicate via mediator

**Benefits:**
- Reduces coupling between components
- Centralizes control logic
- Simplifies object protocols

### CQRS (Command Query Responsibility Segregation)
Originated from Bertrand Meyer's Command-Query Separation principle:
- **Command**: Operation that changes state (void return)
- **Query**: Operation that returns data (no side effects)

CQRS extends this to separate read and write data models entirely:
- Commands: Create, Update, Delete operations
- Queries: Read operations optimized for retrieval
- Separate handlers for commands vs queries

**Benefits:**
- Independent scaling of reads/writes
- Optimized data models for each operation type
- Cleaner separation of concerns

### Event Sourcing (Related Pattern)
- Store state changes as sequence of events
- Rebuild state by replaying events
- Full audit trail
- Often combined with CQRS

### Pub/Sub vs Observer
- **Observer**: Direct coupling, synchronous
- **Pub/Sub**: Broker intermediary, asynchronous possible
- Event Bus is a pub/sub broker implementation

Sources:
- [CQRS Pattern - Microsoft Azure](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)
- [CQRS and Mediator Patterns - Medium](https://medium.com/@darshana-edirisinghe/cqrs-and-mediator-design-patterns-f11d2e9e9c2e)
- [Observer Pattern - Wikipedia](https://en.wikipedia.org/wiki/Observer_pattern)

---

## Step 2: Tech-Stack Library Analysis

### .NET - MediatR
**Strengths:**
- De facto standard for .NET CQRS
- No dependencies
- In-process messaging
- Requests/Responses, Commands, Queries, Notifications
- Pipeline behaviors (middleware)

**API Patterns:**
```csharp
// Command
public record CreateUserCommand(string Name) : IRequest<int>;

// Handler
public class CreateUserHandler : IRequestHandler<CreateUserCommand, int>
{
    public Task<int> Handle(CreateUserCommand cmd, CancellationToken ct)
    {
        // Create user, return ID
    }
}

// Usage
var userId = await mediator.Send(new CreateUserCommand("John"));
```

**Key Features:**
- Generic request/response typing
- Notification (one-to-many) vs Request (one-to-one)
- Behaviors for cross-cutting concerns (logging, validation)

### Python - python-mediator
**Strengths:**
- Clean architecture patterns
- Automatic handler inspection
- Event-driven design support

**API Patterns:**
```python
@mediator.handler
def handle_create_user(cmd: CreateUserCommand) -> int:
    return user_id
```

### Rust - mediator-rs
**Strengths:**
- Type-safe mediator pattern
- Inspired by MediatR

**Challenges:**
- Rust ownership makes traditional mediator difficult
- Top-down ownership approach required

### JavaScript - Event Emitter / RxJS
**Strengths:**
- Built-in EventEmitter in Node.js
- RxJS for reactive streams
- Redux for state + actions

**Limitations:**
- No built-in CQRS abstractions
- Manual type safety

### ABP Framework Event Bus
**Strengths:**
- Local and distributed event bus
- Automatic handler discovery
- Transaction integration

Sources:
- [MediatR - GitHub](https://github.com/jbogard/MediatR)
- [python-mediator - PyPI](https://pypi.org/project/python-mediator/)
- [mediator-rs - Rust](https://docs.rs/mediator/latest/mediator/)
- [ABP Event Bus](https://abp.io/docs/latest/framework/infrastructure/event-bus)

---

## Step 3: Eiffel Ecosystem

### EiffelStudio Eventing (ESS)
- EVENT_TYPE and EVENT_TYPE_I interfaces
- EVENT_TYPE_PUBLISHER_I for publishing
- Observer pattern implementation
- Used in EiffelStudio Services

### EiffelVision2 Action Sequences
- ACTION_SEQUENCE for GUI events
- Connect agents to events
- Synchronous invocation

### SCOOP Observer Pattern
- Example in EiffelStudio documentation
- Uses agents instead of direct calls
- Pre/post-parse callbacks
- Top-down ownership (mediator owns components)

### Gap Analysis
**No dedicated mediator/CQRS library** in Eiffel:
- EVENT_TYPE exists but tied to ESS
- ACTION_SEQUENCE is GUI-focused
- No command/query separation abstraction
- No generic event bus

### Eiffel Strengths for Mediator
- Agents are perfect for callbacks
- Contracts can enforce command/query separation
- SCOOP enables safe concurrent event handling

Sources:
- [Eventing in Services - EiffelStudio](https://dev.eiffel.com/Eventing_in_Services)
- [Event Programming with Agents](https://www.eiffel.org/doc/solutions/Event_Programming_with_Agents)
- [Observer Pattern - Eiffel.org](https://www.eiffel.org/doc/solutions/Observer_pattern)

---

## Step 4: Developer Pain Points

### Code Complexity
1. **Boilerplate explosion**
   - Each command/query needs handler class
   - "The largest trade-off is the amount of object creation"
   - Cannot scaffold with ORM tools

2. **Growing codebase**
   - Multiple layers of abstraction
   - Different developers use different patterns
   - Constant flux with changing requirements

### System Complexity
1. **Data consistency challenges**
   - Multiple databases for read/write = eventual consistency
   - Need fault tolerance strategy for sync failures

2. **Growing dependencies**
   - Controller needs multiple injected interfaces
   - Application complexity increases over time

### When NOT to Use
1. **Simple CRUD applications**
   - CQRS adds unnecessary complexity
   - "Keep it simple" - only use when needed

2. **Anti-patterns**
   - Calling handlers from other handlers
   - Mediator becoming "God Object"
   - Over-engineering simple features

### What Developers Want
1. **Clear separation** of commands vs queries
2. **Simple handler registration** - minimal boilerplate
3. **Event history** for debugging
4. **Type-safe** event data
5. **Pipeline behaviors** for cross-cutting concerns
6. **Enable/disable** handlers dynamically

Sources:
- [CQRS and MediatR in ASP.NET Core](https://codewithmukesh.com/blog/cqrs-and-mediatr-in-aspnet-core/)
- [Why and how I implemented CQRS - ITNEXT](https://itnext.io/why-and-how-i-implemented-cqrs-and-mediator-patterns-in-a-microservice-b07034592b6d)

---

## Step 5: Innovation Opportunities

### simple_mediator Differentiators

1. **Contract-Enforced Command/Query Separation**
```eiffel
-- Commands change state
class SIMPLE_COMMAND
    -- Contracts ensure commands are handled appropriately

-- Queries are side-effect free
class SIMPLE_QUERY [RESULT]
    -- Generic result type for type-safe queries
```

2. **Agent-Based Handlers**
```eiffel
-- No need for handler classes, use agents directly
mediator.subscribe_to ("user.created", agent on_user_created)

-- Or full handler classes for complex logic
mediator.subscribe (my_handler)
```

3. **Event History with Time Travel**
```eiffel
-- Enable history
mediator.enable_event_history

-- Query recent events
recent := event_bus.recent_events (10)

-- Events since timestamp
events := event_bus.events_since (timestamp)
```

4. **Handler Enable/Disable**
```eiffel
handler.disable  -- Temporarily stop receiving events
handler.enable   -- Resume receiving events
```

5. **Command Results with Correlation**
```eiffel
result := mediator.send (command)
if result.is_success then
    id := result.data.item ("id")
else
    error := result.error_message
end
```

6. **Automatic Command Executed Events**
```eiffel
-- Mediator automatically publishes "mediator.command.executed"
-- with command_name, success, correlation_id
```

7. **SCOOP-Safe Design**
- No mutable shared state
- Handler isolation
- Safe concurrent publishing

---

## Step 6: Design Strategy

### Core Design Principles
- **Meyer-Compliant**: True command/query separation
- **Agent-Friendly**: Leverage Eiffel's agents
- **Contract-Safe**: Validate at boundaries
- **Simple API**: Common cases easy

### API Surface

#### SIMPLE_EVENT
```eiffel
class SIMPLE_EVENT

create
    make,            -- Name only
    make_with_data   -- Name + payload

feature -- Access
    name: STRING
    timestamp: DATE_TIME
    data: HASH_TABLE [ANY, STRING]
    source: detachable ANY

feature -- Typed Data Access
    has_key (key: STRING): BOOLEAN
    item (key: STRING): detachable ANY
    string_item (key: STRING): detachable STRING
    integer_item (key: STRING): INTEGER

feature -- Modification
    put (key: STRING; value: ANY)
    put_string (key: STRING; value: STRING)
    set_source (src: ANY)
```

#### SIMPLE_EVENT_HANDLER (Deferred)
```eiffel
deferred class SIMPLE_EVENT_HANDLER

feature -- Access
    handled_events: LIST [STRING]
        deferred

feature -- Status
    handles_event (name: STRING): BOOLEAN
    is_enabled: BOOLEAN

feature -- Execution
    handle (event: SIMPLE_EVENT)
        require
            handles_this_event: handles_event (event.name)
            is_enabled: is_enabled
        deferred
```

#### SIMPLE_EVENT_BUS
```eiffel
class SIMPLE_EVENT_BUS

feature -- Subscription
    subscribe (handler: SIMPLE_EVENT_HANDLER)
    unsubscribe (handler: SIMPLE_EVENT_HANDLER)
    subscribe_procedure (event_name: STRING; proc: PROCEDURE [SIMPLE_EVENT])

feature -- Publishing
    publish (event: SIMPLE_EVENT)
    publish_named (event_name: STRING)
    publish_with_data (name: STRING; data: HASH_TABLE)

feature -- History
    enable_history
    recent_events (count: INTEGER): LIST [SIMPLE_EVENT]
    clear_history
```

#### SIMPLE_COMMAND / SIMPLE_QUERY
```eiffel
class SIMPLE_COMMAND
    command_name: STRING
    correlation_id: detachable STRING
    timestamp: DATE_TIME
    parameters: HASH_TABLE [ANY, STRING]

class SIMPLE_QUERY [G]
    query_name: STRING
    parameters: HASH_TABLE [ANY, STRING]
    -- G is the expected result type
```

#### SIMPLE_MEDIATOR (Facade)
```eiffel
class SIMPLE_MEDIATOR

feature -- Event Operations
    publish (event: SIMPLE_EVENT)
    subscribe (handler: SIMPLE_EVENT_HANDLER)
    subscribe_to (event_name: STRING; procedure: PROCEDURE)

feature -- Command Operations
    register_command_handler (name: STRING; handler: COMMAND_HANDLER)
    send (command: SIMPLE_COMMAND): SIMPLE_COMMAND_RESULT
    send_and_forget (command: SIMPLE_COMMAND)

feature -- Query Operations
    register_query_handler (name: STRING; handler: QUERY_HANDLER)
    query (query: SIMPLE_QUERY): detachable ANY

feature -- Statistics
    event_handler_count: INTEGER
    command_handler_count: INTEGER
```

### Contract Strategy

**Event Handler Preconditions:**
```eiffel
handle (event: SIMPLE_EVENT)
    require
        event_attached: event /= Void
        handles_this_event: handles_event (event.name)
        is_enabled: is_enabled
```

**Command Result Postconditions:**
```eiffel
send (command: SIMPLE_COMMAND): SIMPLE_COMMAND_RESULT
    ensure
        result_attached: Result /= Void
```

**Subscription Preconditions:**
```eiffel
subscribe (handler: SIMPLE_EVENT_HANDLER)
    require
        not_already_subscribed: not is_subscribed (handler)
    ensure
        subscribed: is_subscribed (handler)
```

### Integration Plan
- Part of simple_service_api (SERVICE_API facade)
- Factory methods: `new_mediator`, `new_event_bus`, `new_event`
- Singletons: `mediator`, `event_bus` for convenience

---

## Step 7: Implementation Assessment

### Current Mediator Extension Status

**What's Implemented:**
- SIMPLE_EVENT: name, timestamp, data, source, typed accessors
- SIMPLE_EVENT_HANDLER: deferred class with enable/disable
- SIMPLE_PROCEDURE_HANDLER: agent-based handler
- SIMPLE_EVENT_BUS: subscribe, unsubscribe, publish, history
- SIMPLE_COMMAND: command_name, correlation_id, parameters
- SIMPLE_QUERY [G]: query_name, parameters, generic result type
- SIMPLE_COMMAND_RESULT: is_success, error_message, data
- SIMPLE_COMMAND_HANDLER: deferred command handler
- SIMPLE_QUERY_HANDLER: deferred query handler
- SIMPLE_MEDIATOR: unified facade for events + CQRS

**What's Well Done (Based on Research):**
1. **Agent-based handlers** - SIMPLE_PROCEDURE_HANDLER for simple cases
2. **Handler enable/disable** - Dynamic activation
3. **Event history** - Configurable limit, time-based queries
4. **Command results** - Success/failure with data
5. **Automatic events** - "mediator.command.executed" published
6. **Generic queries** - Type parameter for result

**What's Missing (Based on Research):**
1. **Pipeline behaviors** - No middleware/interceptors
2. **Notification vs Request** - No distinction
3. **Handler priority** - No ordering control
4. **Event filtering** - No wildcard patterns
5. **Async support** - No SCOOP integration yet
6. **Validation pipeline** - No built-in validation step

**Contract Gaps:**
- Missing: Handler priority invariant
- Missing: Event name pattern validation
- Could strengthen: Query result type safety

### Recommendations

1. **Add pipeline behaviors** - Pre/post execution hooks
2. **Add handler priority** - Control execution order
3. **Add wildcard event patterns** - Like MQ topics
4. **Consider SCOOP integration** - Async event publishing
5. **Add validation step** - Command validation before handling

### Comparison to Research Findings

| Feature | Research Priority | Extension Status |
|---------|------------------|------------------|
| Event Bus | High | Implemented |
| Command/Query Sep | High | Implemented |
| Agent Handlers | High | Implemented |
| Event History | Medium | Implemented |
| Enable/Disable | Medium | Implemented |
| Command Results | Medium | Implemented |
| Pipeline Behaviors | Medium | NOT implemented |
| Handler Priority | Low | NOT implemented |
| Wildcard Patterns | Low | NOT implemented |
| Async (SCOOP) | Low | NOT implemented |

---

## Checklist

- [x] Formal specifications reviewed (Mediator, CQRS, Pub/Sub)
- [x] Top libraries studied (MediatR, python-mediator, RxJS)
- [x] Eiffel ecosystem researched (ESS, ACTION_SEQUENCE)
- [x] Developer pain points documented
- [x] Innovation opportunities identified
- [x] Design strategy synthesized
- [x] Implementation assessment completed
- [ ] Missing features implemented (pipeline, priority)
- [ ] Contracts strengthened

