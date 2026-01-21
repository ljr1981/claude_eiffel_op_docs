# simple_scheduler Research Document

## Target Description
Job scheduling for Eiffel: cron expressions, delayed tasks, recurring jobs, persistent schedules

---

## Step 1: Industry Specifications

### POSIX Crontab (IEEE Std 1003.1)
The authoritative specification for cron expressions. Standard format:
- 5 fields: minute, hour, day-of-month, month, day-of-week
- Asterisk (*) = all valid values
- Comma-separated lists (1,5,10)
- Hyphen ranges (1-5)
- Step values (*/5) - extension beyond POSIX

Extended formats (Quartz/Spring):
- 6 fields: add seconds at beginning
- 7 fields: add year at end
- Special characters: L (last), W (weekday), # (nth occurrence)

### RFC 5545 (iCalendar RRULE)
Defines recurrence rules for calendar events:
- FREQ: SECONDLY, MINUTELY, HOURLY, DAILY, WEEKLY, MONTHLY, YEARLY
- INTERVAL: frequency multiplier
- COUNT: number of occurrences
- UNTIL: end date
- BYDAY, BYMONTH, BYMONTHDAY, etc.

Example: `RRULE:FREQ=WEEKLY;BYDAY=MO,WE,FR;COUNT=10`

---

## Step 2: Tech-Stack Libraries (Top by GitHub Stars)

### 1. Quartz Scheduler (Java) - ~6k stars
- Enterprise-grade job scheduling
- Cron + simple triggers
- Clustering and persistence (JDBC)
- JTA transaction support
- Job listeners and plugins

### 2. Celery Beat (Python) - Part of Celery (24k+ stars)
- Distributed task scheduling
- Requires message broker (Redis/RabbitMQ)
- Periodic task scheduling
- Dynamic schedule updates

### 3. APScheduler (Python) - ~5k stars
- In-process scheduling
- Multiple trigger types: cron, interval, date
- Job stores: memory, SQLAlchemy, MongoDB, Redis
- Thread/process pool executors

### 4. robfig/cron (Go) - ~13k stars
- Cron expression parsing
- Timezone support (CRON_TZ)
- Optional seconds field
- Job wrapping (recover, delay, skip)

### 5. gocron (Go) - ~5k stars
- Human-friendly API
- Built on robfig/cron
- Job tags and unique jobs
- Event listeners

### 6. node-schedule (Node.js) - ~9k stars
- Date-based scheduling
- Cron-style expressions
- Recurrence rules

---

## Step 3: Eiffel Ecosystem Analysis

### Current State
**No dedicated scheduler library exists in the Eiffel ecosystem.**

### Available Building Blocks
1. **DATE_TIME** (time library): Current time, date arithmetic
2. **DATE_TIME_DURATION**: Time intervals
3. **SIMPLE_LOG_TIMER** (simple_logger): Elapsed time measurement
4. **NS_TIMER** (objc_base): macOS-only timer via Cocoa
5. **SCOOP**: Concurrency model for background execution

### Gap Analysis
- No cron expression parser
- No recurring job scheduler
- No persistent job storage
- No SCOOP-based timer/scheduler
- Must build from scratch

---

## Step 4: Developer Pain Points

### Cron Syntax Issues
1. **Unfriendly syntax**: "10 8,20 * 8 1-5" is hard to read
2. **Error-prone**: Easy to make mistakes
3. **No validation feedback**: Silent failures
4. **Inconsistent implementations**: Different libraries, different features

### Missing Features in Basic Schedulers
1. **No run history**: Must manually implement logging
2. **No error handling**: Failed jobs go unnoticed
3. **No manual triggers**: Can't run job outside schedule
4. **No UI/visibility**: Hard to see what's scheduled
5. **No persistence**: Jobs lost on restart

### Reliability Challenges
1. **Missed jobs on downtime**: No backfill capability
2. **Overlapping executions**: Previous job still running
3. **Clock drift**: Time synchronization issues
4. **No idempotency**: Double execution problems
5. **Memory leaks**: Long-running schedulers

### Scaling Issues
1. **Single process**: No distributed execution
2. **Resource contention**: Too many jobs at once
3. **No prioritization**: All jobs equal
4. **No rate limiting**: Overwhelms downstream systems

---

## Step 5: Innovation Opportunities

### 1. Human-Readable API
```eiffel
scheduler.every (5).minutes.do (agent process_queue)
scheduler.daily.at ("09:00").do (agent send_report)
scheduler.weekly.on_monday.at ("08:00").do (agent team_standup)
```

### 2. Contract-Based Scheduling
```eiffel
schedule (a_job)
  require
    job_valid: a_job.is_valid
    not_duplicate: not has_job (a_job.id)
  ensure
    scheduled: has_job (a_job.id)
    next_run_set: a_job.next_run /= Void
```

### 3. Integrated Telemetry
- Automatic span creation for job execution
- Metrics: job_duration, job_success_rate, job_queue_depth
- Log correlation with trace IDs

### 4. Persistence Options
- Memory (default, fast)
- simple_sql (SQLite persistence)
- simple_cache (Redis persistence)

### 5. SCOOP-Compatible Design
- Background scheduler processor
- Separate worker processors
- Non-blocking job execution

---

## Step 6: Design Strategy

### Core Classes

```
SIMPLE_SCHEDULER
├── SIMPLE_CRON_EXPRESSION     (cron parsing)
├── SIMPLE_TRIGGER
│   ├── SIMPLE_CRON_TRIGGER    (cron-based)
│   ├── SIMPLE_INTERVAL_TRIGGER (fixed interval)
│   └── SIMPLE_DATE_TRIGGER    (one-time)
├── SIMPLE_JOB
│   └── SIMPLE_JOB_RESULT
├── SIMPLE_JOB_STORE
│   ├── SIMPLE_MEMORY_JOB_STORE
│   └── SIMPLE_SQL_JOB_STORE
└── SIMPLE_SCHEDULER_LISTENER
```

### SIMPLE_SCHEDULER (Facade)
```eiffel
class SIMPLE_SCHEDULER
feature -- Scheduling
  schedule (a_job: SIMPLE_JOB; a_trigger: SIMPLE_TRIGGER)
  schedule_cron (a_job: SIMPLE_JOB; a_expression: STRING)
  schedule_interval (a_job: SIMPLE_JOB; a_seconds: INTEGER)
  schedule_at (a_job: SIMPLE_JOB; a_time: DATE_TIME)
  unschedule (a_job_id: STRING)

feature -- Fluent API
  every (a_count: INTEGER): SIMPLE_INTERVAL_BUILDER
  daily: SIMPLE_DAILY_BUILDER
  weekly: SIMPLE_WEEKLY_BUILDER
  cron (a_expression: STRING): SIMPLE_CRON_BUILDER

feature -- Control
  start
  stop
  pause
  resume
  trigger_now (a_job_id: STRING)

feature -- Query
  is_running: BOOLEAN
  job_count: INTEGER
  pending_jobs: ARRAYED_LIST [SIMPLE_JOB]
  next_fire_time: detachable DATE_TIME
end
```

### SIMPLE_CRON_EXPRESSION
```eiffel
class SIMPLE_CRON_EXPRESSION
create
  make, make_extended

feature -- Access
  minute: STRING
  hour: STRING
  day_of_month: STRING
  month: STRING
  day_of_week: STRING
  second: STRING  -- optional (extended)

feature -- Calculation
  next_fire_time (a_after: DATE_TIME): DATE_TIME
  previous_fire_time (a_before: DATE_TIME): DATE_TIME
  matches (a_time: DATE_TIME): BOOLEAN

feature -- Validation
  is_valid: BOOLEAN
  validation_error: detachable STRING

feature -- Human Readable
  to_description: STRING
    -- "Every 5 minutes"
    -- "At 9:00 AM on Monday through Friday"
end
```

### SIMPLE_JOB
```eiffel
class SIMPLE_JOB
feature -- Access
  id: STRING
  name: STRING
  action: PROCEDURE
  context: detachable ANY

feature -- State
  is_enabled: BOOLEAN
  last_run: detachable DATE_TIME
  last_result: detachable SIMPLE_JOB_RESULT
  run_count: INTEGER
  error_count: INTEGER

feature -- Options
  allow_concurrent: BOOLEAN  -- default False
  retry_count: INTEGER       -- on failure
  timeout_seconds: INTEGER   -- max execution time
end
```

### Patterns Applied
1. **Facade**: SIMPLE_SCHEDULER unifies all scheduling
2. **Strategy**: Trigger types (cron, interval, date)
3. **Builder**: Fluent API for schedule creation
4. **Observer**: Listeners for job events
5. **Repository**: Job stores for persistence

---

## Step 7: Implementation Assessment

### Level of Effort: Medium-High

### Phase 1: Core (3-4 files)
- SIMPLE_CRON_EXPRESSION: Parse and calculate cron schedules
- SIMPLE_TRIGGER + implementations: Trigger types
- SIMPLE_JOB + SIMPLE_JOB_RESULT: Job representation
- SIMPLE_SCHEDULER: Main facade with timer loop

### Phase 2: Fluent API (2-3 files)
- SIMPLE_INTERVAL_BUILDER
- SIMPLE_DAILY_BUILDER
- SIMPLE_WEEKLY_BUILDER

### Phase 3: Persistence (1-2 files)
- SIMPLE_JOB_STORE interface
- SIMPLE_MEMORY_JOB_STORE
- SIMPLE_SQL_JOB_STORE (optional)

### Dependencies
- **base**: Core Eiffel
- **time**: DATE_TIME, DATE_TIME_DURATION
- **simple_testing**: Test framework

### Risks
1. **Cron parsing complexity**: Many edge cases
2. **Timer accuracy**: System load affects timing
3. **SCOOP integration**: Separate processor coordination
4. **Long-running stability**: Memory leaks, resource exhaustion

### Mitigation
- Start with subset of cron syntax (standard 5-field)
- Use simple polling loop initially
- Test with extended duration runs
- Add telemetry for monitoring

---

## Recommended Approach

### MVP Features
1. Cron expression parsing (standard 5-field)
2. Interval triggers
3. One-time date triggers
4. Memory job store
5. Basic scheduler with start/stop
6. Job execution with error handling

### Fluent API Examples
```eiffel
scheduler.every (30).seconds.do (agent heartbeat)
scheduler.every (5).minutes.do (agent process_queue)
scheduler.every (1).hours.do (agent cleanup)
scheduler.daily.at ("09:00").do (agent morning_report)
scheduler.cron ("0 */6 * * *").do (agent sync_data)  -- Every 6 hours
```

### Human-Readable Descriptions
```
"*/5 * * * *"  -> "Every 5 minutes"
"0 9 * * 1-5"  -> "At 9:00 AM, Monday through Friday"
"0 0 1 * *"    -> "At midnight on the 1st of every month"
```

---

## Sources
- [POSIX crontab](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/crontab.html)
- [RFC 5545 iCalendar](https://datatracker.ietf.org/doc/html/rfc5545)
- [Quartz Scheduler](https://github.com/quartz-scheduler)
- [APScheduler](https://github.com/agronholm/apscheduler)
- [robfig/cron](https://github.com/robfig/cron)
- [gocron](https://pkg.go.dev/github.com/go-co-op/gocron)
- [Google SRE: Distributed Cron](https://sre.google/sre-book/distributed-periodic-scheduling/)
- [cron-parser](https://github.com/harrisiirak/cron-parser)
