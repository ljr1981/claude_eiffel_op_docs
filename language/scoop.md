# SCOOP - Simple Concurrent Object-Oriented Programming

## Overview

SCOOP is Eiffel's native concurrency model, designed by Bertrand Meyer. It makes concurrent programming safer by:
- Eliminating data races through the type system
- Using Design by Contract for synchronization
- Providing a single keyword (`separate`) instead of threads/locks/mutexes

**Key insight**: SCOOP uses argument passing to identify when exclusive access is needed, making concurrency reasoning simpler.

## Enabling SCOOP

### In ECF File

```xml
<target name="my_target">
    <setting name="concurrency" value="scoop"/>
    <!-- ... -->
</target>
```

### In EiffelStudio GUI

1. Project Settings → Target → Advanced
2. Change **Concurrency** from "None" to "SCOOP"
3. Use precompiled library: `base-scoop-safe` (or none)
4. Clean compile required after changing

**Important**: When using a precompiled library, its Concurrency setting overrides your project's setting. Use `base-scoop-safe` for SCOOP projects.

## Core Concepts

### The `separate` Keyword

Declares that an object may be handled by a different processor (thread):

```eiffel
my_worker: separate WORKER
    -- my_worker may run on a different processor
```

When SCOOP is disabled, `separate` is ignored and code runs sequentially.

### Processors

A **processor** is an autonomous thread of control that executes operations on objects. In EiffelStudio, processors are implemented as OS threads.

- Each object is handled by exactly one processor
- Objects on the same processor execute sequentially
- Objects on different processors may execute concurrently
- Maximum processors per system: 1024

### Regions

A **region** is a set of objects handled by the same processor. All objects in a region share the same execution context.

## Separate Calls

### The Separate Argument Rule

**Critical rule**: A separate call `x.f(a)` is valid only if `x` is a formal argument of the enclosing routine.

```eiffel
-- INVALID: Cannot call on separate attribute directly
my_worker: separate WORKER

do_work
    do
        my_worker.perform  -- COMPILE ERROR!
    end

-- VALID: Pass as argument to lock it
do_work
    do
        launch_worker (my_worker)
    end

launch_worker (a_worker: separate WORKER)
    do
        a_worker.perform  -- OK: a_worker is locked
    end
```

### Synchronous vs Asynchronous

| Call Type | Behavior |
|-----------|----------|
| Non-separate call | Always synchronous (sequential) |
| Separate command | Asynchronous (caller continues immediately) |
| Separate query | Synchronous ("wait by necessity" - must wait for result) |

```eiffel
process_worker (a_worker: separate WORKER)
    do
        a_worker.start_task      -- Asynchronous: caller continues
        a_worker.do_something    -- Asynchronous: queued after start_task
        print (a_worker.result)  -- Synchronous: waits for result
    end
```

## Inline Separate Block

Syntactic sugar to avoid creating wrapper routines:

```eiffel
person: separate PERSON

show_age
    do
        -- Instead of creating a wrapper routine:
        separate person as l_person do
            print (l_person.age)
        end
    end
```

Equivalent to:

```eiffel
show_age
    do
        show_age_wrapper (person)
    end

show_age_wrapper (l_person: separate PERSON)
    do
        print (l_person.age)
    end
```

The local variable (`l_person`) is:
- Read-only
- Controlled (has exclusive access)
- Can access enclosing routine's locals

## Exclusive Access

### How Locking Works

When a routine has separate arguments, the runtime **atomically** locks all regions before executing the body:

```eiffel
transfer (a_from, a_to: separate ACCOUNT; amount: INTEGER)
        -- Both accounts locked atomically - no deadlock possible
    do
        a_from.withdraw (amount)
        a_to.deposit (amount)
    end
```

### Controlled Objects

An object is **controlled** (safely accessible) if:
1. It has a non-separate type, OR
2. It is a formal argument of the enclosing routine, OR
3. It is a local variable of an inline separate block

Uncontrolled separate objects cannot be called directly.

### Non-Blocking Access

Since EiffelStudio 15.01, gaining exclusive access is **non-blocking**. The system creates queues for each processor. Waiting only occurs on synchronous calls (queries).

**Only way to deadlock**: A processor stuck waiting for a separate query result.

## Wait Conditions (Contract-Based Synchronization)

### Preconditions as Wait Conditions

In SCOOP, preconditions on separate arguments become **wait conditions** instead of correctness checks:

```eiffel
consume (a_buffer: separate BUFFER)
    require
        not_empty: not a_buffer.is_empty  -- WAIT condition, not failure
    do
        process (a_buffer.get)
    end
```

If `a_buffer.is_empty`, the processor **waits** until the condition becomes true (another processor adds items), rather than failing.

### Rules for Wait Conditions

| Precondition Type | Behavior |
|-------------------|----------|
| Involves separate argument not yet locked | Wait condition |
| Involves separate argument already locked | Correctness condition (fail if false) |
| No separate involvement | Correctness condition |

### Postconditions

Postconditions work as in sequential code but be cautious: state may change after the call returns due to concurrent access.

### Class Invariants

Class invariants **cannot contain separate calls** (no formal arguments available). They are always evaluated by the object's own processor.

## Patterns

### Producer-Consumer

```eiffel
class PRODUCER
feature
    produce (a_buffer: separate BUFFER)
        do
            from until done loop
                a_buffer.put (create_item)
            end
        end
end

class CONSUMER
feature
    consume (a_buffer: separate BUFFER)
        require
            not a_buffer.is_empty  -- Wait condition
        do
            process (a_buffer.get)
        end
end
```

### Worker Pool

```eiffel
class COORDINATOR
feature
    workers: ARRAY [separate WORKER]

    distribute_work (tasks: LIST [TASK])
        local
            i: INTEGER
        do
            from i := 1 until i > tasks.count loop
                launch (workers [i \\ workers.count + 1], tasks [i])
                i := i + 1
            end
        end

    launch (a_worker: separate WORKER; a_task: TASK)
        do
            a_worker.execute (a_task)  -- Asynchronous
        end
end
```

### Waiting for Completion

```eiffel
wait_for_completion (a_worker: separate WORKER)
    require
        a_worker.is_done  -- Wait until done
    do
        -- Proceed only when worker is done
    end
```

## Common Gotchas

### 1. Cannot Call Separate Attributes Directly

```eiffel
-- WRONG
my_separate_obj.do_something  -- Compile error

-- RIGHT
call_on_separate (my_separate_obj)

call_on_separate (obj: separate MY_TYPE)
    do
        obj.do_something
    end
```

### 2. Queries Block

Separate queries are synchronous - they wait for results:

```eiffel
process (a_worker: separate WORKER)
    local
        result: INTEGER
    do
        a_worker.start_long_task  -- Returns immediately (async)
        result := a_worker.get_result  -- BLOCKS until complete
    end
```

### 3. Postcondition Timing

State may change between postcondition check and caller continuing:

```eiffel
add_item (a_buffer: separate BUFFER)
    do
        a_buffer.put (item)
    ensure
        not a_buffer.is_empty  -- True now, but...
    end
    -- Another processor might empty it before caller continues!
```

### 4. Separate Objects in Containers

Be careful with containers of separate objects - each access needs proper locking.

## SCOOP Examples in EiffelStudio

Located in `$ISE_EIFFEL/examples/scoop/`:

| Example | Demonstrates |
|---------|--------------|
| Producer-consumer | Basic async communication via shared buffer |
| Dining philosophers | Multi-resource locking without deadlock |
| Barbershop | Scheduling with multiple concurrent agents |
| Quicksort | Parallel divide-and-conquer algorithm |
| Observer pattern | Concurrent event notification |
| Counter | Simple shared state |
| Baboon crossing | Resource constraints |
| Search-insert-delete | Concurrent data structure operations |

## Best Practices

1. **Profile first** - Don't add concurrency without evidence it helps
2. **Minimize shared state** - Fewer separate objects = simpler reasoning
3. **Use commands for async, queries sparingly** - Queries block
4. **Design for coarse-grained parallelism** - Fine-grained has overhead
5. **Test thoroughly** - Concurrency bugs are timing-dependent
6. **Use wait conditions** - Let SCOOP handle synchronization via preconditions
7. **Prefer inline separate for simple cases** - Cleaner than wrapper routines

## Performance Considerations

- Each SCOOP processor = OS thread (overhead)
- Async calls have queueing overhead
- Sync calls (queries) block the caller
- Finalized builds give better SCOOP performance
- Profile before and after to verify improvement

## Sources

- [Concurrent Programming with SCOOP](https://www.eiffel.org/doc/solutions/Concurrent_programming_with_SCOOP)
- [Getting Started with SCOOP](https://www.eiffel.org/doc/solutions/Getting_Started_with_SCOOP)
- [Separate Calls](https://www.eiffel.org/doc/solutions/Separate_Calls)
- [Exclusive Access](https://www.eiffel.org/doc/solutions/Exclusive_Access)
- [Design by Contract in SCOOP](https://www.eiffel.org/doc/solutions/Design_by_Contract_in_SCOOP)
- [SCOOP Examples](https://www.eiffel.org/doc/solutions/SCOOP_examples)
- [SCOOP Tutorial](https://www.eiffel.org/doc/solutions/SCOOP_tutorial)
