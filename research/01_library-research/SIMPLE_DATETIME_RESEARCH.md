# simple_datetime Deep Research

**Date:** December 7, 2025
**Purpose:** Research best practices and pain points before implementing simple_datetime

---

## Part 1: Industry Best Practices

### Sources:
- [Oracle Java Date-Time Design Principles](https://docs.oracle.com/javase/tutorial//datetime/overview/design.html)
- [Fluent Interface Patterns](https://java-design-patterns.com/patterns/fluent-interface/)
- [Phrase: Best JavaScript Date/Time Libraries](https://phrase.com/blog/posts/best-javascript-date-time-libraries/)
- [Better Stack: Moment.js Alternatives](https://betterstack.com/community/guides/scaling-nodejs/momentjs-alternatives/)

### Key Design Principles

1. **Immutability** - All modern datetime libraries emphasize immutable objects
   - Java 8's java.time: "Most classes are immutable, meaning after creation they cannot be modified"
   - Methods return NEW instances, never modify in place
   - Thread-safe by definition

2. **Fluent API** - Chainable methods for readability
   - `date.plus_days(7).at_time(14, 30).in_timezone("America/New_York")`
   - Return `Current` (or new instance) from each method

3. **No Nulls** - Methods don't accept or return null
   - Use Option/Maybe pattern or sensible defaults
   - Clear preconditions

4. **`with` instead of `set`** - Since objects are immutable
   - `new_date := date.with_year(2025)` returns a new object
   - Original `date` unchanged

5. **Separation of Concerns**
   - DATE (calendar date only, no time)
   - TIME (time of day only, no date)
   - DATETIME (both)
   - DURATION (length of time)
   - INTERVAL (between two points)

---

## Part 2: Developer Pain Points

### Sources:
- [Jon Skeet: Common DateTime Mistakes](https://codeblog.jonskeet.uk/2015/05/05/common-mistakes-in-datetime-formatting-and-parsing/)
- [Falsehoods Programmers Believe About Time](https://gist.github.com/timvisee/fcda9bbdff88d45cc9061606b4b923ca)
- [UTC is Enough for Everyone, Right?](https://zachholman.com/talk/utc-is-enough-for-everyone-right)
- [Caktus: Coding for Timezones](https://www.caktusgroup.com/blog/2019/03/21/coding-time-zones-and-daylight-saving-time/)

### Top Pain Points

1. **Timezone Confusion**
   - "More than 50% of developers struggle with timezones"
   - Confusing offset (-05:00) with timezone (America/New_York)
   - DST transitions cause times to not exist or exist twice

2. **Parsing Hell**
   - Every format needs different parsing code
   - "2025-12-07" vs "12/07/2025" vs "Dec 7, 2025"
   - ISO 8601 should be standard but isn't universal

3. **Formatting Verbosity**
   - Complex format strings ("yyyy-MM-dd'T'HH:mm:ss.SSSZ")
   - Different libraries use different patterns

4. **Arithmetic Complexity**
   - "Add 1 month to January 31" - what's the result?
   - Business days vs calendar days
   - Leap year handling

5. **Age Calculation Edge Cases**
   - Born on Feb 29 - when do you age up?
   - Different countries have different rules!

6. **Naive vs Aware Datetimes**
   - "Never use naive datetimes" - but they're the default
   - Converting between them is error-prone

---

## Part 3: Common Use Cases Developers Need

### Sources:
- [Stack Overflow: Calculate Age](https://stackoverflow.com/questions/9/how-do-i-calculate-someones-age-based-on-a-datetime-type-birthday)
- [Bomberbot: Python Business Days](https://www.bomberbot.com/python/python-mastering-date-calculations-for-business-days/)
- [Hightouch: SQL Relative Dates](https://hightouch.com/sql-dictionary/sql-relative-dates)

### Must-Have Operations

1. **Current Time**
   - `now` - current datetime in local timezone
   - `utc_now` - current datetime in UTC
   - `today` - current date only
   - `now_in ("America/New_York")` - current time in specific zone

2. **Creation**
   - From components: `date(2025, 12, 25)`
   - From string: `parse("2025-12-25")`
   - From timestamp: `from_timestamp(1735084800)`

3. **Formatting**
   - ISO 8601: `to_iso8601`
   - Custom: `format("YYYY-MM-DD")`
   - Human: `to_human` → "December 25, 2025"

4. **Arithmetic**
   - `plus_days(7)`, `minus_months(1)`
   - `plus_business_days(5)` - skip weekends
   - `next_weekday` - skip to Monday if weekend

5. **Comparison**
   - `is_before`, `is_after`, `is_same_day`
   - `days_until`, `days_since`
   - `is_past`, `is_future`

6. **Extraction**
   - `year`, `month`, `day`, `hour`, `minute`, `second`
   - `day_of_week`, `day_of_year`, `week_of_year`
   - `quarter`

7. **Boundaries**
   - `start_of_day`, `end_of_day`
   - `start_of_month`, `end_of_month`
   - `start_of_year`, `end_of_year`
   - `start_of_week` (with configurable first day)

8. **Age/Duration**
   - `age_from(birthdate)` → years, months, days
   - `duration_since(past_date)`
   - `duration_until(future_date)`

---

## Part 4: Innovative "Ahead of the Curve" Features

### Sources:
- [Chrono: Natural Language Parser](https://github.com/wanasit/chrono)
- [Dateparser Python](https://www.zyte.com/blog/parse-natural-language-dates-with-dateparser/)
- [PrettyTime NLP](https://www.ocpsoft.org/prettytime/nlp/)
- [Arrow/Pendulum Comparison](https://medium.com/@marcnealer/python-datetimes-with-arrow-or-pendulum-6ec7495a9112)

### Innovation Opportunities

1. **Human-Friendly Relative Time**
   ```eiffel
   dt.time_ago        -- "3 hours ago", "2 days ago", "just now"
   dt.time_until      -- "in 5 minutes", "tomorrow", "next week"
   dt.humanize        -- "last Tuesday at 3pm"
   ```

2. **Smart Parsing (Try Multiple Formats)**
   ```eiffel
   -- Automatically detects format
   parse ("2025-12-07")           -- ISO
   parse ("12/07/2025")           -- US
   parse ("07/12/2025")           -- EU (with locale hint)
   parse ("December 7, 2025")     -- Natural
   parse ("next Tuesday")         -- Relative
   parse ("in 3 days")            -- Relative
   ```

3. **Business Calendar Support**
   ```eiffel
   dt.plus_business_days (5)                    -- Skip weekends
   dt.plus_business_days_with_holidays (5, us_holidays)
   dt.is_business_day
   dt.next_business_day
   dt.business_days_until (end_date)
   ```

4. **Range/Interval First-Class**
   ```eiffel
   range := date_range (start, end)
   range.days                     -- Iterate days
   range.weeks                    -- Iterate weeks
   range.contains (some_date)
   range.overlaps (other_range)
   range.business_days_count
   ```

5. **Convenience Queries**
   ```eiffel
   dt.is_weekend
   dt.is_weekday
   dt.is_leap_year
   dt.is_last_day_of_month
   dt.is_end_of_quarter
   dt.days_in_month               -- 28/29/30/31
   dt.quarter                     -- 1, 2, 3, or 4
   ```

6. **Relative Date Builders**
   ```eiffel
   next_monday
   last_friday
   first_day_of_next_month
   last_day_of_previous_month
   next_quarter_start
   fiscal_year_start (starting_month)
   ```

7. **Age Calculation Done Right**
   ```eiffel
   age := birthdate.age           -- SIMPLE_AGE object
   age.years                      -- 25
   age.months                     -- 3
   age.days                       -- 14
   age.total_days                 -- 9245
   age.to_string                  -- "25 years, 3 months, 14 days"
   age.is_adult                   -- True (configurable threshold)
   ```

8. **Timezone-Aware by Default**
   ```eiffel
   -- All datetimes carry timezone
   dt.timezone                    -- "America/New_York"
   dt.in_utc
   dt.in_timezone ("Europe/London")
   dt.utc_offset                  -- "-05:00"
   dt.is_dst                      -- True/False
   ```

9. **Duration Arithmetic**
   ```eiffel
   duration := 2.days + 3.hours + 30.minutes
   future := now + duration
   past := now - duration

   -- Or fluent
   duration := duration.days(2).hours(3).minutes(30)
   ```

10. **Recurring Event Support**
    ```eiffel
    -- Every Tuesday at 2pm
    schedule := recurrence.weekly.on_day (tuesday).at_time (14, 0)
    next_5 := schedule.next_occurrences (5)

    -- Every month on the 15th
    schedule := recurrence.monthly.on_day (15)

    -- Every 2 weeks
    schedule := recurrence.every_n_weeks (2).starting (some_date)
    ```

---

## Part 5: Proposed simple_datetime API

### Core Classes

```
SIMPLE_DATETIME (main facade)
    ├── now, utc_now, today, tomorrow, yesterday
    ├── date (y, m, d), time (h, m, s), datetime (y, m, d, h, m, s)
    ├── parse (string), parse_iso8601 (string)
    ├── from_timestamp (unix_time)
    └── timezone utilities

SIMPLE_DATE (date only, no time)
    ├── year, month, day, day_of_week, day_of_year
    ├── plus_days, minus_days, plus_months, plus_years
    ├── start_of_month, end_of_month, start_of_year
    ├── is_before, is_after, is_same_day
    ├── is_weekend, is_weekday, is_leap_year
    ├── format, to_iso8601, to_human
    └── immutable, fluent returns

SIMPLE_TIME (time only, no date)
    ├── hour, minute, second, millisecond
    ├── plus_hours, plus_minutes, plus_seconds
    ├── is_before, is_after
    ├── format, to_iso8601
    └── immutable, fluent returns

SIMPLE_DATE_TIME (date + time + timezone)
    ├── All of SIMPLE_DATE
    ├── All of SIMPLE_TIME
    ├── timezone, in_utc, in_timezone
    ├── time_ago, time_until, humanize
    ├── to_timestamp, to_iso8601
    └── immutable, fluent returns

SIMPLE_DURATION (length of time)
    ├── days, hours, minutes, seconds
    ├── plus, minus (combine durations)
    ├── from components or string
    └── humanize ("2 hours, 30 minutes")

SIMPLE_AGE (specialized for age calculation)
    ├── years, months, days
    ├── total_days, total_months
    ├── to_string
    └── is_adult, is_minor (configurable)

SIMPLE_DATE_RANGE (interval between dates)
    ├── start_date, end_date
    ├── days, weeks, months (iterators)
    ├── contains, overlaps
    ├── business_days_count
    └── duration
```

### Integration with FOUNDATION_API

```eiffel
class FOUNDATION_API

feature -- DateTime (Innovative!)

    -- Current time
    now: SIMPLE_DATE_TIME
    utc_now: SIMPLE_DATE_TIME
    today: SIMPLE_DATE
    tomorrow: SIMPLE_DATE
    yesterday: SIMPLE_DATE

    -- Creation
    date (y, m, d: INTEGER): SIMPLE_DATE
    time (h, m, s: INTEGER): SIMPLE_TIME
    datetime (y, m, d, h, m, s: INTEGER): SIMPLE_DATE_TIME

    -- Parsing (smart - tries multiple formats)
    parse_date (s: STRING): SIMPLE_DATE
    parse_datetime (s: STRING): SIMPLE_DATE_TIME
    parse_iso8601 (s: STRING): SIMPLE_DATE_TIME

    -- Relative dates (innovative!)
    next_monday: SIMPLE_DATE
    last_friday: SIMPLE_DATE
    first_of_month: SIMPLE_DATE
    last_of_month: SIMPLE_DATE

    -- Age calculation (done right!)
    age_from (birthdate: SIMPLE_DATE): SIMPLE_AGE

    -- Business days
    add_business_days (d: SIMPLE_DATE; n: INTEGER): SIMPLE_DATE
    business_days_between (start, end: SIMPLE_DATE): INTEGER

    -- Ranges
    date_range (start, end: SIMPLE_DATE): SIMPLE_DATE_RANGE

    -- Human-friendly (innovative!)
    time_ago (dt: SIMPLE_DATE_TIME): STRING  -- "3 hours ago"
    time_until (dt: SIMPLE_DATE_TIME): STRING  -- "in 2 days"
```

---

## Part 6: Differentiators from Other Libraries

| Feature | Python datetime | Arrow | Pendulum | Luxon | **simple_datetime** |
|---------|-----------------|-------|----------|-------|---------------------|
| Immutable | No | Yes | Yes | Yes | **Yes** |
| Timezone-aware default | No | Yes | Yes | Yes | **Yes** |
| Business days | No | No | No | No | **Yes** |
| Age calculation | No | No | Limited | No | **First-class** |
| `time_ago` | No | Yes | Yes | Yes | **Yes** |
| Date ranges | No | Limited | Limited | Limited | **First-class** |
| Smart parsing | No | Yes | Yes | Limited | **Yes** |
| Relative dates | No | Limited | Limited | No | **First-class** |
| Design by Contract | No | No | No | No | **Yes** |
| Fluent API | No | Yes | Yes | Yes | **Yes** |

---

## Part 7: Implementation Priority

### Phase 1: Core (Day 1-2)
- SIMPLE_DATETIME facade
- SIMPLE_DATE with basic operations
- SIMPLE_TIME with basic operations
- SIMPLE_DATE_TIME combining both
- Basic parsing and formatting

### Phase 2: Advanced (Day 2-3)
- SIMPLE_DURATION
- SIMPLE_AGE
- SIMPLE_DATE_RANGE
- Business day support
- Relative date helpers

### Phase 3: Polish (Day 3-4)
- `time_ago` / `time_until` humanization
- Smart multi-format parsing
- Timezone support
- Integration with FOUNDATION_API
- Full test coverage

---

---

## Part 8: Eiffel Ecosystem Date/Time Patterns

### From Eiffel Loop (Finnian Reilly) - Ideas Only (NOT Void-Safe!)

**Location:** https://github.com/finnianr/Eiffel-Loop/tree/master/library/base/text/date-time

**Directory Structure:**
- `types/` - Core types (EL_DATE, EL_TIME, EL_DATE_TIME, EL_TIME_DURATION, EL_ISO_8601_DATE_TIME)
- `formatting/` - Output formatting (EL_DATE_TEXT, EL_DATE_TEXT_TEMPLATE, EL_DATE_TIME_CODE_STRING)
- `helper/` - Utilities (EL_SYSTEM_TIME, EL_TIME_ROUTINES, EL_EXECUTION_TIMER)
- `parser/` - Parsing support
- `conversion/` - Format conversion
- `shared/` - Shared constants

**Key Design Patterns from EL_DATE:**
1. **Adapter Pattern** - Implements interface (EL_TIME_DATE_I) while extending DATE
2. **Factory Methods** - Multiple make_* routines for different input sources
3. **Flyweight Pattern** - `Once_date_time` for shared reusable objects
4. **Validation** - `valid_string_for_code()` validates before parsing

**Localization Pattern (EL_DATE_TEXT, EL_MONTH_TEXTS, EL_DAY_OF_WEEK_TEXTS):**
- Separate text classes for localized names
- Deferred locale interface (EL_DEFERRED_LOCALE_I)
- Default to English (EL_ENGLISH_DATE_TEXT)

### From Pylon Library (Portable Foundation)

**P_DATE / P_TIME / P_DATE_TIME patterns:**
- `is_valid` query for validation
- Multiple output formats: `to_iso`, `to_iso_long`, `to_european`, `to_american`, `to_rfc`
- `hour_12`, `is_am`, `is_pm` for 12-hour time
- `timezone_bias` in minutes for timezone offset
- Composition: P_DATE_TIME contains P_DATE and P_TIME (not inheritance)

### From ISO8601 Library (Thomas Beale) - Void-Safe!

**Repository:** https://github.com/eiffelhub/iso8601

**Key Patterns:**
1. **Validation before conversion** - `valid_iso8601_date()` before `iso8601_string_to_date()`
2. **Cached parser** - Avoid redundant parsing when validate + convert
3. **Central routines class** - ISO8601_ROUTINES as facade
4. **Bidirectional conversion** - String ↔ ISO8601 object ↔ Eiffel DATE/TIME

---

## Attribution Requirements

When implementing simple_datetime, give credit in class notes:

```eiffel
note
    description: "[
        Simple DateTime - High-level date/time API for Eiffel.

        Design influenced by:
        - Eiffel Loop (Finnian Reilly) - Localization patterns, multiple format outputs
        - Pylon library - Composition over inheritance, format variety (iso, rfc, etc.)
        - ISO8601 library (Thomas Beale) - Validation before conversion pattern
        - Gobo Time library (Eric Bezault) - DT_DATE epoch-based calculations
        - Luxon/date-fns (JavaScript) - Fluent API, immutability
        - Pendulum (Python) - Timezone-aware by default, humanize output
        ]"
```

---

## References

### Design Patterns
- [Oracle Java Date-Time Design Principles](https://docs.oracle.com/javase/tutorial//datetime/overview/design.html)
- [Baeldung: Fluent Interface vs Builder](https://www.baeldung.com/java-fluent-interface-vs-builder-pattern)
- [Stack Overflow: Fluent APIs - Return This or New?](https://stackoverflow.com/questions/19841569/fluent-apis-return-this-or-new)

### Library Comparisons
- [NPM Compare: Date Libraries](https://npm-compare.com/date-fns,dayjs,luxon,moment)
- [aboutsimon: Python DateTime Libraries](https://aboutsimon.com/blog/2016/08/04/datetime-vs-Arrow-vs-Pendulum-vs-Delorean-vs-udatetime.html)
- [Better Stack: Moment.js Alternatives](https://betterstack.com/community/guides/scaling-nodejs/momentjs-alternatives/)

### Pain Points
- [Jon Skeet: Common DateTime Mistakes](https://codeblog.jonskeet.uk/2015/05/05/common-mistakes-in-datetime-formatting-and-parsing/)
- [Falsehoods Programmers Believe About Time](https://gist.github.com/timvisee/fcda9bbdff88d45cc9061606b4b923ca)
- [Ten Python Datetime Pitfalls](https://dev.arie.bovenberg.net/blog/python-datetime-pitfalls/)

### Natural Language
- [Chrono: Natural Language Date Parser](https://github.com/wanasit/chrono)
- [Zyte: Dateparser](https://www.zyte.com/blog/parse-natural-language-dates-with-dateparser/)
- [PrettyTime NLP](https://www.ocpsoft.org/prettytime/nlp/)
