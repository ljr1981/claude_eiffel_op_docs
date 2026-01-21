# Eiffel Library Ecosystem Reference

**Created:** December 7, 2025
**Purpose:** Document the historical and current Eiffel library landscape for reference when designing simple_* wrappers

---

## Core Libraries

### EiffelBase (ISE)
The principal contribution of Eiffel - a library of fundamental structures and algorithms covering the basics of computing science.

**Includes:**
- Core data structures (lists, arrays, hash tables, etc.)
- Fundamental algorithms
- Kernel classes (STRING, INTEGER, etc.)

**Status:** Active, maintained by Eiffel Software
**Location:** `$ISE_LIBRARY\library\base\`

### EiffelTime (ISE)
Date and time handling library.

**Key Classes:**
- `DATE` - Calendar dates
- `TIME` - Time of day
- `DATE_TIME` - Combined date and time
- `DATE_DURATION`, `TIME_DURATION`, `DATE_TIME_DURATION`
- `INTERVAL` - Time intervals

**Location:** `$ISE_LIBRARY\library\time\`

---

## Third-Party Libraries (Void-Safe!)

### ISO8601 Library (eiffelhub)

**GitHub:** https://github.com/eiffelhub/iso8601
**Author:** Thomas Beale (wolandscat@gmail.com)
**License:** Apache-2.0
**Status:** Void-safe, in production use on openEHR.org

**This is a candidate for use in simple_datetime!**

**Key Classes:**
- `ISO8601_ROUTINES` - Central conversion interface
- `ISO8601_DATE` - Date with component access
- `ISO8601_TIME` - Time with component access
- `ISO8601_DATE_TIME` - Combined datetime
- `ISO8601_DURATION` - Duration representation

**Key Features:**
- String ↔ ISO8601 object ↔ Eiffel DATE/TIME conversion
- Validation before conversion
- Cached parser for performance (no redundant parsing)
- Multiple format support:
  - Dates: YYYY, YYYY-MM, YYYY-MM-DD
  - Times: with optional timezone
  - Durations: "PNNDTNNhNNmNNs" format

**API Patterns:**
```eiffel
-- Validation
valid_iso8601_date (s: STRING): BOOLEAN
valid_iso8601_date_time (s: STRING): BOOLEAN
valid_iso8601_duration (s: STRING): BOOLEAN

-- Conversion
date_time_to_iso8601_string (dt: DATE_TIME): STRING
iso8601_string_to_date (s: STRING): DATE
iso8601_string_to_time (s: STRING): TIME
duration_to_iso8601_string (d: DURATION): STRING
```

**Integration Opportunity:**
Consider using this library as a dependency for simple_datetime's ISO8601 parsing/formatting.

---

## Gobo Libraries

**Created by:** Eric Bezault
**Website:** https://www.gobosoft.com/
**Status:** De-facto standard for Eiffel libraries

### Gobo Time (DT_* classes)
**Location:** `$ISE_LIBRARY\contrib\library\gobo\library\time\`

**Key Classes:**
- `DT_DATE` - Gregorian calendar dates
- `DT_TIME` - Time of day
- `DT_DATE_TIME` - Combined
- `DT_DURATION`, `DT_DATE_DURATION`, `DT_TIME_DURATION`
- `DT_WEEK_DAY` - Day of week handling
- `DT_GREGORIAN_CALENDAR` - Calendar calculations

**Notable Features:**
- Week number calculation (ISO 8601)
- Day count from epoch (1970-01-01)
- Canonical duration between dates

### Gobo XML (XM_* classes)
**Location:** `$ISE_LIBRARY\contrib\library\gobo\library\xml\`

**Already wrapped by:** simple_xml

### Gobo Structure
Data structure library.

### Gobo String
String manipulation and formatting, including date/time formatting.

**Location:** `$ISE_LIBRARY\contrib\library\gobo\library\string\src\date\`
- `ST_DATE_TIME_FORMAT`
- `ST_DATE_TIME_PARSER`
- `ST_DATE_TIME_ROUTINES`
- `ST_XSD_DATE_TIME_FORMAT` (XML Schema formats)
- `ST_XSLT_FORMAT_DATE_TIME`

---

## Historical Libraries

### Pylon (Unmaintained - Ideas Source)

**Website:** https://www.nenie.org/eiffel/pylon/
**Last Release:** 0.82 beta (January 1998)
**Status:** Unmaintained, use Gobo instead
**Value:** Design ideas for simple_* wrappers

**Pylon Date/Time Classes:**

#### P_DATE
- `year`, `month`, `day` with setters
- `is_valid` - validates actual calendar date
- `is_valid_date (y, m, d)` - static validation
- `is_leap_year`
- `day_of_week`, `day_of_year`
- **String formats:**
  - `to_iso` → "18870726"
  - `to_iso_long` → "1887-07-26"
  - `to_european` → European format
  - `to_american` → American format
  - `to_rfc` → RFC-822 Internet format

#### P_TIME
- `hour`, `minute`, `second`
- `hour_12` - 12-hour format (1-12)
- `is_am`, `is_pm`
- **String formats:**
  - `to_iso` → "233055"
  - `to_rfc` → "23:30:55"

#### P_DATE_TIME
- **Composition:** Contains `date: P_DATE` and `time: P_TIME`
- `set_date`, `set_time`
- **Timezone:**
  - `is_local` - no timezone specified
  - `timezone_bias` - minutes offset from UTC
  - `set_timezone_bias`
  - `is_utc` - zero offset
- Combined string formats with timezone

**Design Note:** Pylon explicitly avoids "current time" functionality, leaving it to OS-specific code.

#### Other Pylon Features (Ideas for future simple_* libs)
- **Data Structures:** P_LIST, P_LINKED_LIST, P_SEQUENCE_LIST, P_STACK, P_QUEUE, P_SET, P_HASH_TABLE
- **Serialization:** P_TEXT_OBJECT for text serialization
- **Iterators:** External iterators combined with cursor-based navigation
- **String Parsing:** Number formatting and parsing utilities

### Eiffel Loop (NOT Void-Safe - Ideas Source)

**Website:** http://eiffel-loop.com/
**GitHub:** https://github.com/finnianr/Eiffel-Loop
**Author:** Finnian Reilly (developing since 2002)
**Status:** 4100+ classes, but NOT void-safe and unlikely to be migrated
**Problem:** Massive library with good ideas, but unusable in void-safe projects

**Notable Features for Mining Ideas:**
- **String Library (46+ classes):**
  - Split cursors for string iteration
  - Substring interval editing
  - Character escape tables
  - Reflection-based object initialization from strings
  - Foreign naming convention adapters (camelCase, kebab-case)
  - Automatic type conversion from strings

- **Other Clusters:**
  - Testing with CRC-32 checksum regression testing
  - Extensive reflection and serialization
  - XML processing
  - Encryption/hashing
  - OS integration

**Design Mining Opportunity:**
Review Eiffel Loop's patterns for:
- String manipulation conveniences
- Reflection/serialization approaches
- Testing patterns
- Any datetime utilities

**Caveat:** Cannot use directly - must reimplement ideas in void-safe code.

### SmallEiffel / SmartEiffel / GNU Eiffel
**Status:** Mostly stopped development (stable at v2.3)
**Notable:** Had TIME_IN_GERMAN for German date/time locale

### ELKS (Eiffel Library Kernel Standard)
**Purpose:** Standards for Eiffel libraries across compilers
**Versions:** ELKS '95, ELKS 2000, ELKS 2001
**Status:** Influenced Gobo and EiffelBase design

---

## Design Lessons from Historical Libraries

### From Pylon:
1. **Validation as query:** `is_valid_date(y, m, d)` for pre-validation
2. **Multiple string formats:** ISO, European, American, RFC
3. **12-hour time:** `hour_12`, `is_am`, `is_pm`
4. **Timezone as bias:** Simple minutes offset approach
5. **Composition over inheritance:** P_DATE_TIME contains P_DATE and P_TIME
6. **Comparable + Hashable:** Date/time objects should support both

### From Gobo:
1. **Epoch-based calculations:** `day_count` from 1970-01-01
2. **Week handling:** ISO 8601 week numbers
3. **Canonical durations:** Normalize year/month/day
4. **Handler pattern:** Separate handler classes for operations

### Common Pain Points (addressed by simple_datetime):
1. **Verbose creation:** Multiple make_* procedures → single `make` or factory methods
2. **Format strings:** Complex patterns → named methods (`to_iso8601`, `to_human`)
3. **No relative dates:** → Add `next_monday`, `last_friday`
4. **No business days:** → Add `plus_business_days`
5. **No human output:** → Add `time_ago`, `time_until`
6. **No age calculation:** → Add `SIMPLE_AGE` class

---

## Library Relationships

```
EiffelBase (ISE)
    └── Used by everything

EiffelTime (ISE)           Gobo Time
    └── DATE                   └── DT_DATE
    └── TIME                   └── DT_TIME
    └── DATE_TIME              └── DT_DATE_TIME
         │                          │
         └──────────┬───────────────┘
                    ▼
            simple_datetime (Our wrapper)
                    │
                    ▼
            FOUNDATION_API
```

---

## References

- [Pylon Library](https://www.nenie.org/eiffel/pylon/)
- [Pylon User's Guide](https://www.nenie.org/eiffel/pylon/pylon1.html)
- [Gobo Eiffel Project](https://www.gobosoft.com/)
- [Gobo Time Library](http://www.gobosoft.com/eiffel/gobo/time/index.html)
- [EiffelBase Documentation](https://www.eiffel.org/doc/solutions/EiffelBase)
- [Eiffel Libraries](https://www.eiffel.org/resources/libraries)
- [ELKS History](https://en.wikibooks.org/wiki/Eiffel_Programming/Versions)
