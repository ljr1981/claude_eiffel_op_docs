# simple_telemetry Research Notes

## Step 1: Specifications

### OpenTelemetry (OTEL) - CNCF Incubating Project
The de facto standard for observability instrumentation:
- **Tracing**: Distributed request tracking across services
- **Metrics**: Numerical measurements of system behavior
- **Logging**: Timestamped event records

### W3C Trace Context
Standard HTTP headers for distributed tracing:
- **traceparent**: `{version}-{trace-id}-{parent-id}-{flags}`
  - Version: 2-digit hex (00)
  - Trace ID: 32-hex globally unique identifier
  - Parent ID: 16-hex span identifier
  - Flags: 2-hex (01 = sampled)
- **tracestate**: Vendor-specific trace data

### OpenTelemetry Tracing Concepts

**Span**: Single unit of work
- Name, start time, end time
- Span kind: Client, Server, Internal, Producer, Consumer
- Attributes: key-value metadata
- Events: timestamped annotations
- Status: Ok, Error, Unset
- Parent/child relationships

**Trace**: Directed acyclic graph (DAG) of spans
- Shared trace ID across all spans
- Parent-child links form call tree

**Context**: Propagates trace/span IDs across boundaries

### OpenTelemetry Metrics Concepts

**Meter**: Creates metric instruments

**Instruments**:
- Counter: Monotonically increasing value
- UpDownCounter: Can increase or decrease
- Gauge: Point-in-time value
- Histogram: Distribution of values

**Attributes**: Dimensions for metrics (labels)

### OpenTelemetry Logging Concepts

**Log Record**:
- Timestamp
- Severity level
- Body (message)
- Attributes
- Trace ID / Span ID (correlation)

### The Three Pillars of Observability
1. **Logs**: Historical records, debugging detail
2. **Metrics**: Numerical health indicators, alerting
3. **Traces**: Request flow, performance bottlenecks

**Key Insight**: Correlated pillars are more valuable than isolated data.

Sources:
- [OpenTelemetry - Traces](https://opentelemetry.io/docs/concepts/signals/traces/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [Three Pillars of Observability - IBM](https://www.ibm.com/think/insights/observability-pillars)
- [OpenTelemetry Specification](https://github.com/open-telemetry/opentelemetry-specification)

---

## Step 2: Tech-Stack Library Analysis

### JavaScript - @opentelemetry/sdk-node
**Strengths:**
- Official OTEL implementation
- Auto-instrumentation for popular frameworks
- Multiple exporters (OTLP, Jaeger, Zipkin)

**API Patterns:**
```javascript
const tracer = trace.getTracer('my-service');
const span = tracer.startSpan('operation');
span.setAttribute('key', 'value');
span.end();
```

### Python - opentelemetry-python
**Strengths:**
- Official OTEL SDK
- Django, Flask, FastAPI auto-instrumentation
- Prometheus, Jaeger exporters

### Rust - opentelemetry-rust
**Strengths:**
- High performance
- Multiple exporters
- Async support

**Key Crates:**
- `opentelemetry`: API crate
- `opentelemetry_sdk`: SDK implementation
- `opentelemetry-otlp`: OTLP exporter

### Go - go.opentelemetry.io/otel
**Strengths:**
- First-class OTEL support
- Context propagation built-in
- gRPC instrumentation

### Common Patterns Across Languages
1. **Tracer/Meter/Logger providers** - Central configuration
2. **Start/End span pattern** - Explicit lifecycle
3. **Context propagation** - Pass trace context
4. **Attributes** - Key-value metadata
5. **Exporters** - Send to backends

Sources:
- [OpenTelemetry Languages](https://opentelemetry.io/docs/languages/)
- [opentelemetry-rust](https://github.com/open-telemetry/opentelemetry-rust)
- [OpenTelemetry JavaScript](https://opentelemetry.io/docs/languages/js/)

---

## Step 3: Eiffel Ecosystem

### Existing Capabilities

**EiffelStudio Tracing**
- Built-in tracing facility
- Flow of control logging
- `disable_tracing` / enable selectively
- Writes to standard output
- Debugger integration

**Limitations:**
- Tied to EiffelStudio debugger
- Not distributed tracing
- No trace ID propagation
- No metric collection

**EiffelStudio Profiling**
- Built-in metrics and profiling
- System-wide information
- Not runtime observable

**Design by Contract as Instrumentation**
- Preconditions/postconditions log violations
- Class invariants catch state errors
- Natural form of defensive programming

### Gap Analysis
**No production observability library** in Eiffel:
- No W3C trace context support
- No span/trace abstractions
- No metric instruments (Counter, Gauge)
- No log correlation with traces
- No OTEL-compatible exporters

### Eiffel Strengths for Telemetry
- Contracts provide built-in validation
- Once features for singletons (providers)
- Agents for callbacks (span events)
- Strong typing for attributes

Sources:
- [Tracing - Eiffel.org](https://www.eiffel.org/doc/eiffelstudio/Tracing)
- [Eiffel Debugging - Programming Homework Help](https://www.programminghomeworkhelp.com/blog/eiffel-debugging-mastery-best-practices-guide/)

---

## Step 4: Developer Pain Points

### Knowledge & Skills Gap
1. **#1 Challenge (48% of developers)**
   - Understanding OTEL concepts
   - Knowing what to instrument
   - Interpreting telemetry data

### Data Volume & Cost
1. **High telemetry volumes**
   - 23% YoY growth in data
   - 53% cite log storage costs
   - Outmoded per-GB pricing

2. **Sampling decisions**
   - What to keep vs drop
   - Tail-based vs head-based sampling

### Integration Complexity
1. **No standardization**
   - CI/CD tools emit non-standard telemetry
   - Different teams use different tools
   - Data silos

2. **Correlation challenges**
   - Linking logs to traces
   - Matching metrics to specific requests
   - Cross-service context

### Implementation Challenges
1. **Manual instrumentation**
   - Tedious to add spans everywhere
   - Easy to forget cleanup (end span)
   - Context propagation across async

2. **Cardinality explosion**
   - Too many unique attribute combinations
   - Metric dimensions grow unbounded

### MTTR (Mean Time to Recovery)
1. **Increasing MTTR (73% report hours)**
   - Can't find root cause quickly
   - Data silos impair diagnosis
   - Too much noise

### What Developers Want
1. **Simple API** - Start/end spans easily
2. **Auto-correlation** - Link logs/traces automatically
3. **Reasonable defaults** - Work out of box
4. **Structured logging** - Not just strings
5. **Cost control** - Sampling, filtering
6. **Standard format** - OTEL/W3C compatible

Sources:
- [Observability Pulse 2024 - Logz.io](https://logz.io/observability-pulse-2024/)
- [Observability Challenges - DevOps.com](https://devops.com/how-to-alleviate-your-observability-challenges/)
- [Observability in 2024 - The New Stack](https://thenewstack.io/observability-in-2024-more-opentelemetry-less-confusion/)

---

## Step 5: Innovation Opportunities

### simple_telemetry Differentiators

1. **Contract-Based Instrumentation**
```eiffel
start_span (a_name: STRING): SIMPLE_SPAN
    require
        name_not_empty: not a_name.is_empty
    ensure
        span_started: Result.is_active
        has_trace_id: not Result.trace_id.is_empty
```

2. **Automatic Span Scoping**
```eiffel
-- With agent-based scoping
tracer.with_span ("operation", agent do_work)

-- Span automatically ended when agent completes
-- Even on exception
```

3. **Correlated Logging Built-in**
```eiffel
-- Logger automatically includes trace/span IDs
logger.info ("Processing request")
-- Output: [trace_id=abc123 span_id=def456] Processing request
```

4. **Simple Metric Instruments**
```eiffel
-- Counter
counter := meter.new_counter ("requests")
counter.add (1)
counter.add_with_attributes (1, [["method", "GET"]])

-- Gauge
gauge := meter.new_gauge ("temperature")
gauge.record (23.5)

-- Histogram
histogram := meter.new_histogram ("latency_ms")
histogram.record (42)
```

5. **W3C Trace Context Compatible**
```eiffel
-- Extract from HTTP headers
context := propagator.extract (headers)

-- Inject into outgoing request
propagator.inject (context, request.headers)
```

6. **Eiffel-Native Structured Logging**
```eiffel
logger.info_structured ([
    "message", "User logged in",
    "user_id", user.id,
    "ip_address", request.ip
])
```

7. **SCOOP-Safe Design**
- Thread-safe metric updates
- Context propagation across processors
- No shared mutable state

8. **Reasonable Defaults**
```eiffel
-- One-line setup
telemetry := create {SIMPLE_TELEMETRY}.make ("my-service")

-- Ready to use
span := telemetry.tracer.start_span ("operation")
```

---

## Step 6: Design Strategy

### Core Design Principles
- **OTEL-Compatible**: Follow OTEL semantic conventions
- **Simple First**: Common cases in few lines
- **Correlated**: Logs/traces/metrics linked
- **Pure Eiffel**: No C dependencies initially

### API Surface

#### SIMPLE_TELEMETRY (Main Facade)
```eiffel
class SIMPLE_TELEMETRY

create
    make,               -- Service name
    make_with_config    -- Full configuration

feature -- Providers
    tracer: SIMPLE_TRACER
    meter: SIMPLE_METER
    logger: SIMPLE_TELEMETRY_LOGGER

feature -- Configuration
    set_exporter (exporter: SIMPLE_TELEMETRY_EXPORTER)
    set_sampler (sampler: SIMPLE_SAMPLER)
    enable, disable

feature -- Convenience
    trace (name: STRING; action: PROCEDURE)
        -- Execute action within a span
```

#### SIMPLE_SPAN
```eiffel
class SIMPLE_SPAN

feature -- Access
    name: STRING
    trace_id: STRING       -- 32-hex
    span_id: STRING        -- 16-hex
    parent_span_id: detachable STRING
    start_time: DATE_TIME
    end_time: detachable DATE_TIME
    kind: INTEGER          -- CLIENT, SERVER, INTERNAL, etc.
    status: INTEGER        -- OK, ERROR, UNSET
    attributes: HASH_TABLE [ANY, STRING]
    events: LIST [SIMPLE_SPAN_EVENT]

feature -- Status
    is_active: BOOLEAN
    is_recording: BOOLEAN

feature -- Modification
    set_attribute (key: STRING; value: ANY)
    add_event (name: STRING)
    add_event_with_data (name: STRING; data: HASH_TABLE)
    set_status (code: INTEGER; message: STRING)
    end_span

feature -- Context
    context: SIMPLE_TRACE_CONTEXT
```

#### SIMPLE_TRACER
```eiffel
class SIMPLE_TRACER

create
    make   -- tracer name

feature -- Span Creation
    start_span (name: STRING): SIMPLE_SPAN
    start_span_with_parent (name: STRING; parent: SIMPLE_SPAN): SIMPLE_SPAN
    start_span_with_kind (name: STRING; kind: INTEGER): SIMPLE_SPAN

feature -- Scoped Spans
    with_span (name: STRING; action: PROCEDURE)
        -- Execute action within auto-ended span

feature -- Context
    current_span: detachable SIMPLE_SPAN
```

#### SIMPLE_METER
```eiffel
class SIMPLE_METER

create
    make   -- meter name

feature -- Instruments
    new_counter (name: STRING): SIMPLE_COUNTER
    new_up_down_counter (name: STRING): SIMPLE_UP_DOWN_COUNTER
    new_gauge (name: STRING): SIMPLE_GAUGE
    new_histogram (name: STRING): SIMPLE_HISTOGRAM
```

#### SIMPLE_COUNTER
```eiffel
class SIMPLE_COUNTER

feature -- Recording
    add (value: INTEGER_64)
    add_with_attributes (value: INTEGER_64; attrs: ARRAY [TUPLE [key, value: STRING]])
```

#### SIMPLE_TELEMETRY_LOGGER
```eiffel
class SIMPLE_TELEMETRY_LOGGER

feature -- Logging (auto-correlates with current span)
    debug (message: STRING)
    info (message: STRING)
    warn (message: STRING)
    error (message: STRING)

feature -- Structured Logging
    log_structured (level: INTEGER; data: HASH_TABLE [ANY, STRING])
```

#### SIMPLE_TRACE_CONTEXT
```eiffel
class SIMPLE_TRACE_CONTEXT

feature -- Access
    trace_id: STRING
    span_id: STRING
    trace_flags: INTEGER
    trace_state: detachable STRING

feature -- W3C Format
    to_traceparent: STRING
        -- "00-{trace_id}-{span_id}-{flags}"

feature -- Parsing
    from_traceparent (header: STRING)
```

### Contract Strategy

**Span Lifecycle:**
```eiffel
end_span
    require
        is_active: is_active
    ensure
        ended: not is_active
        has_end_time: end_time /= Void
```

**Metric Recording:**
```eiffel
add (value: INTEGER_64)
    require
        non_negative: value >= 0  -- Counters are monotonic
```

**Context Propagation:**
```eiffel
start_span_with_parent (name: STRING; parent: SIMPLE_SPAN)
    require
        parent_active: parent.is_active
    ensure
        linked: Result.parent_span_id.same_string (parent.span_id)
        same_trace: Result.trace_id.same_string (parent.trace_id)
```

### Integration Plan
- Add to SERVICE_API: `new_telemetry`, `tracer`, `meter`, `telemetry_logger`
- Console exporter by default
- JSON file exporter for persistence
- Future: OTLP exporter for backends

---

## Step 7: Implementation Assessment

### Current simple_telemetry Status

**Not Yet Implemented** - New library

### Planned Implementation

**Phase 1 - Core:**
1. SIMPLE_TELEMETRY facade
2. SIMPLE_TRACER with span creation
3. SIMPLE_SPAN with full lifecycle
4. SIMPLE_TRACE_CONTEXT for W3C format
5. Console exporter

**Phase 2 - Metrics:**
1. SIMPLE_METER
2. SIMPLE_COUNTER, SIMPLE_GAUGE
3. SIMPLE_HISTOGRAM

**Phase 3 - Logging:**
1. SIMPLE_TELEMETRY_LOGGER
2. Auto-correlation with spans
3. Structured logging

**Phase 4 - Exporters:**
1. JSON file exporter
2. OTLP exporter (requires HTTP)

### Comparison to Research Findings

| Feature | Research Priority | Planned |
|---------|------------------|---------|
| Span/Trace | High | Phase 1 |
| W3C Trace Context | High | Phase 1 |
| Metrics (Counter/Gauge) | High | Phase 2 |
| Correlated Logging | Medium | Phase 3 |
| Histogram | Medium | Phase 2 |
| Sampling | Medium | Phase 1 |
| Auto-instrumentation | Low | Future |
| OTLP Export | Low | Phase 4 |

### Key Design Decisions

1. **Pure Eiffel First** - No C dependencies initially
2. **Console/File Export** - Usable without backend
3. **W3C Compatible** - Industry standard format
4. **Contract-Heavy** - Eiffel strength
5. **SCOOP-Ready** - Thread-safe design

---

## Checklist

- [x] Formal specifications reviewed (OTEL, W3C Trace Context)
- [x] Top libraries studied (OTEL JS, Python, Rust, Go)
- [x] Eiffel ecosystem researched (tracing, profiling gaps)
- [x] Developer pain points documented
- [x] Innovation opportunities identified
- [x] Design strategy synthesized
- [x] Implementation assessment completed
- [ ] Phase 1 implemented (tracing)
- [ ] Phase 2 implemented (metrics)
- [ ] Phase 3 implemented (logging)
- [ ] Phase 4 implemented (exporters)

