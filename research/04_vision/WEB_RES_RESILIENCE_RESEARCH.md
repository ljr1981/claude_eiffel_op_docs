# WEB-RES: simple_web Resilience Extension Research Report

**Date:** 2025-12-13
**Status:** RESEARCH COMPLETE
**Roadmap ID:** WEB-RES
**Phase:** 1 (Quick Wins)

---

## Executive Summary

This document captures the Seven-Step Research Process for adding resilience patterns (retry, circuit breaker, timeout, bulkhead, fallback) to simple_web. Research confirms this extension is feasible with significant reuse of existing simple_* infrastructure.

---

## 1. RFC/Specification Research

### Circuit Breaker Pattern
- **Origin:** Michael Nygard, "Release It!" (2007)
- **Reference:** [Martin Fowler - Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
- **States:** Closed (normal) → Open (failing) → Half-Open (testing recovery)
- **No formal RFC** - de facto industry standard

### Exponential Backoff with Jitter
- **Reference:** [AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
- **Formula:** `wait = base * 2^attempt + random_jitter`
- **Jitter Types:** Full Jitter, Equal Jitter, Decorrelated Jitter
- **Purpose:** Prevents thundering herd / retry storms
- **No formal RFC** - best practice documented by AWS, Google, Microsoft

### Bulkhead Pattern
- **Origin:** Ship compartmentalization metaphor
- **Purpose:** Isolate resources to prevent cascading failures
- **Types:** Semaphore-based (concurrent calls), Thread pool-based (dedicated threads)

---

## 2. Tech-Stack Library Analysis

### Polly (.NET)
- **Source:** [Polly Documentation](https://www.pollydocs.org/)
- **Pattern:** Builder/Pipeline
- **Circuit Breaker States:** Closed, Open, HalfOpen, **Isolated** (manual override)
- **Retry Config:** MaxAttempts, Delay, BackoffType, UseJitter, MaxDelay
- **Key Feature:** Failure ratio-based circuit breaker (not just count)

### Resilience4j (Java)
- **Source:** [Resilience4j GitHub](https://github.com/resilience4j/resilience4j)
- **Pattern:** Decorator (wrap functions)
- **Bulkhead Types:** Semaphore OR Thread pool
- **Key Feature:** Stackable decorators, Prometheus/Grafana integration
- **Requires:** Java 17+

### Sony gobreaker (Go)
- **Source:** [gobreaker GitHub](https://github.com/sony/gobreaker)
- **Pattern:** Struct-based state machine
- **Default:** Trips after 5 consecutive failures, 60s timeout
- **Key Feature:** Simple, minimal API

### Comparison

| Feature | Polly | Resilience4j | gobreaker | **simple_* (proposed)** |
|---------|-------|--------------|-----------|-------------------------|
| Retry | Yes | Yes | No (separate lib) | Yes (simple_http) |
| Circuit Breaker | Yes | Yes | Yes | **NEW** |
| Bulkhead | Yes | Yes | No | **NEW** |
| Fallback | Yes | Yes | No | **NEW** |
| Contracts | No | No | No | **YES (DBC)** |
| Thread Safety | Manual | Manual | Manual | **SCOOP** |

---

## 3. Eiffel Ecosystem Check

### Existing Assets

| Library | Component | Status |
|---------|-----------|--------|
| simple_http | `SIMPLE_HTTP_RETRY_POLICY` | EXISTS - full exponential backoff + jitter |
| simple_http | Timeout settings | EXISTS |
| simple_web | `SIMPLE_WEB_MIDDLEWARE` | EXISTS - base class |
| simple_web | `SIMPLE_WEB_MIDDLEWARE_PIPELINE` | EXISTS - chain execution |
| simple_web | Auth, CORS, Logging middleware | EXISTS |
| simple_rate_limiter | Token bucket, sliding window | EXISTS |

### Reuse Strategy
- **REUSE:** `SIMPLE_HTTP_RETRY_POLICY` (copy or depend on simple_http)
- **REUSE:** Middleware pipeline architecture
- **CREATE:** Circuit breaker, bulkhead, unified policy, middleware

---

## 4. Developer Pain Points

### Production Incident Data
- OpenAI: 112+ incidents in H1 2024 ([ACM Survey](https://dl.acm.org/doi/10.1145/3715005))
- Retry storms increased load 400% during partial outages ([Microsoft Azure](https://learn.microsoft.com/en-us/azure/architecture/antipatterns/retry-storm/))
- AWS observed synchronized retries every 32 seconds

### Common Failure Patterns
1. **Cascading Failures** - One service failure triggers chain reaction
2. **Retry Storms** - Synchronized retries overwhelm recovering services
3. **Death Spiral** - Healthy nodes receive overflow traffic → fail → cascade
4. **Resource Exhaustion** - Missing bulkhead allows one call to consume all connections

### Root Causes
- Timeout misconfiguration between services
- Retry without backoff/jitter
- Missing circuit breakers
- No bulkhead isolation
- Cache failures overwhelming databases

---

## 5. Innovation Opportunities

### Eiffel Advantages

| Innovation | Eiffel Capability | Competitor Gap |
|------------|-------------------|----------------|
| Contract-Enforced Health | DBC invariants verify state machine | No compile-time checking |
| SCOOP-Safe State | Automatic thread synchronization | Manual locks required |
| Void Safety | No null pointer in resilience code | Runtime NPE possible |
| Agent-based Fallbacks | Clean callback mechanism | Lambda/closure complexity |

### Unique Value Proposition
```eiffel
-- DBC ensures circuit breaker correctness
invariant
    closed_implies_healthy: state = Closed implies failure_count < threshold
    open_implies_cooling: state = Open implies (now - last_failure) < cooldown
```

No other resilience library has compile-time + runtime contract verification.

---

## 6. Design Strategy

### Architecture

```
simple_web/src/resilience/
├── simple_circuit_breaker.e          -- State machine
├── simple_bulkhead.e                 -- Concurrency limiter
├── simple_resilience_policy.e        -- Unified builder
├── simple_fallback.e                 -- Fallback wrapper
└── simple_web_resilience_middleware.e -- Pipeline integration
```

### Class Design: SIMPLE_CIRCUIT_BREAKER

```eiffel
class SIMPLE_CIRCUIT_BREAKER
create
    make, make_with_config

feature -- State (Closed=0, Open=1, HalfOpen=2)
    state: INTEGER
    failure_count: INTEGER
    success_count: INTEGER
    last_failure_time: DATE_TIME

feature -- Configuration
    failure_threshold: INTEGER     -- Default: 5
    success_threshold: INTEGER     -- Default: 3 (for half-open)
    cooldown_seconds: INTEGER      -- Default: 30

feature -- Operations
    allow_request: BOOLEAN
        -- Can request proceed?
        require
            valid_state: state >= 0 and state <= 2
        do
            inspect state
            when State_closed then Result := True
            when State_open then Result := is_cooldown_elapsed
            when State_half_open then Result := True
            end
        end

    record_success
        -- Record successful call
        do
            if state = State_half_open then
                success_count := success_count + 1
                if success_count >= success_threshold then
                    transition_to_closed
                end
            else
                failure_count := 0  -- Reset on success
            end
        ensure
            success_resets_failures: state = State_closed implies failure_count = 0
        end

    record_failure
        -- Record failed call
        do
            failure_count := failure_count + 1
            last_failure_time := create {DATE_TIME}.make_now
            if failure_count >= failure_threshold then
                transition_to_open
            end
        ensure
            threshold_opens: failure_count >= failure_threshold implies state = State_open
        end

invariant
    valid_state: state >= State_closed and state <= State_half_open
    threshold_positive: failure_threshold > 0
    cooldown_positive: cooldown_seconds > 0
end
```

### Class Design: SIMPLE_RESILIENCE_POLICY (Builder)

```eiffel
class SIMPLE_RESILIENCE_POLICY
create
    make

feature -- Builder (fluent API returning like Current)
    with_retry (a_max: INTEGER): like Current
    with_retry_backoff (a_initial_ms, a_max_ms: INTEGER; a_jitter: BOOLEAN): like Current
    with_circuit_breaker (a_threshold, a_cooldown: INTEGER): like Current
    with_timeout (a_seconds: INTEGER): like Current
    with_bulkhead (a_max_concurrent: INTEGER): like Current
    with_fallback (a_handler: FUNCTION [ANY, TUPLE, detachable ANY]): like Current

feature -- Execution
    execute [G] (a_operation: FUNCTION [ANY, TUPLE, G]): G
        -- Execute with all policies applied
        -- Order: Bulkhead → Circuit Breaker → Timeout → Retry → Operation
end
```

### Usage Example

```eiffel
-- Client-side resilience
client: SIMPLE_WEB_CLIENT
policy: SIMPLE_RESILIENCE_POLICY

create policy.make
policy.with_retry (3)
      .with_circuit_breaker (5, 30)
      .with_timeout (10)
      .with_fallback (agent cached_response)

response := policy.execute (agent client.get ("https://api.example.com/data"))

-- Server-side middleware
router.use (create {SIMPLE_WEB_RESILIENCE_MIDDLEWARE}.make (
    create {SIMPLE_RESILIENCE_POLICY}.make
        .with_circuit_breaker (10, 60)
        .with_bulkhead (100)
))
```

---

## 7. Implementation Plan

### Files to Create

| File | Lines (est.) | Complexity |
|------|--------------|------------|
| `simple_circuit_breaker.e` | 150 | Medium |
| `simple_bulkhead.e` | 100 | Low |
| `simple_resilience_policy.e` | 200 | Medium |
| `simple_web_resilience_middleware.e` | 80 | Low |

### Tests Required

1. Circuit breaker opens after N failures
2. Circuit breaker transitions to half-open after cooldown
3. Circuit breaker closes after N successes in half-open
4. Retry with exponential backoff timing
5. Retry with jitter prevents synchronization
6. Bulkhead rejects requests beyond limit
7. Timeout fires correctly
8. Fallback returns correct value on failure
9. Policy composition works correctly

### Dependencies

- simple_http (for SIMPLE_HTTP_RETRY_POLICY) - OR copy class
- simple_datetime (for timing)
- simple_web (extends middleware)

---

## 8. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| SCOOP complexity | Medium | Medium | Start with non-SCOOP version |
| Thread timing issues | Low | High | Use DATE_TIME consistently |
| Integration breaks existing code | Low | Medium | Non-breaking additions only |

---

## 9. Success Metrics

- [ ] All 9 test scenarios pass
- [ ] Zero compilation warnings
- [ ] DBC contracts cover all state transitions
- [ ] API matches roadmap specification
- [ ] Integrates with existing middleware pipeline

---

## Sources

- [Martin Fowler - Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
- [AWS Exponential Backoff and Jitter](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
- [Polly Documentation](https://www.pollydocs.org/)
- [Resilience4j GitHub](https://github.com/resilience4j/resilience4j)
- [Sony gobreaker](https://github.com/sony/gobreaker)
- [Microsoft Azure - Retry Storm Antipattern](https://learn.microsoft.com/en-us/azure/architecture/antipatterns/retry-storm/)
- [Cerbos - Handling Failures in Microservices](https://www.cerbos.dev/blog/handling-failures-in-microservice-architectures)

---

**Research Complete. Ready for Implementation.**
