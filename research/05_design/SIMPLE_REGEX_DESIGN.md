# simple_regex Design Report

## Executive Summary

**simple_regex** is a high-level, fluent Eiffel library wrapping Gobo's PCRE regex engine (`RX_PCRE_REGULAR_EXPRESSION`). It aims to make regex operations intuitive, safe, and enjoyable for Eiffel developers by addressing common pain points and providing a modern API that rivals the best libraries in other languages.

**Key Value Propositions:**
1. One-liner convenience methods for 80% of use cases
2. Fluent builder for readable pattern construction
3. Contract-driven safety catching errors at compile time
4. Full integration with simple_* ecosystem (FOUNDATION_API, SERVICE_API)

---

## Research Findings Summary

### 1. Specifications Reviewed

| Specification | Key Insights |
|--------------|--------------|
| **POSIX.1-2024** | BRE vs ERE, longest match semantics, portable syntax |
| **PCRE2 10.47** | Perl-compatible, Unicode support, JIT compilation, UTS#18 set operations |
| **Unicode UTS#18** | Three conformance levels, Unicode properties, grapheme clusters |
| **ECMAScript 2024** | New `/v` flag for unicodeSets, set operations |

### 2. Tech-Stack Library Analysis

| Language | Library | Key Innovation |
|----------|---------|----------------|
| **Python** | `regex` module | Unicode properties, concurrent matching, GIL release |
| **Python** | `re2` | Linear time guarantee, ReDoS protection |
| **Rust** | `regex` crate | Linear time, modular architecture, `regex-lite` for minimal builds |
| **Go** | `regexp` | RE2-based, linear time, but limited (no backreferences) |
| **Go** | `regexp2` | .NET-compatible, full features but no time guarantee |
| **JS** | `/v` flag | Set operations in character classes |

**Design Patterns Observed:**
- Rust: Separation of simple API (`regex`) vs power API (`regex-automata`)
- Python: Drop-in replacement pattern (`import re2 as re`)
- Go: `MustCompile` for panic-on-error compile
- All: Pre-compile patterns for performance

### 3. Gobo Eiffel Analysis

**Current API (RX_PCRE_REGULAR_EXPRESSION):**
```eiffel
create regex.make
regex.compile ("pattern")
if regex.is_compiled then
    regex.match (subject)
    if regex.has_matched then
        result := regex.captured_substring (0)
    end
end
```

**Pain Points:**
- Verbose: 6+ lines for simple match
- Manual compilation step
- Manual error checking
- No fluent interface
- No convenience methods

**Strengths to Preserve:**
- Full PCRE support
- Unicode handling
- Mature, well-tested
- Eric Bezault's quality

### 4. Developer Pain Points Documented

| Pain Point | Frequency | simple_regex Solution |
|-----------|-----------|----------------------|
| Cryptic syntax | Very High | Fluent builder with named methods |
| Can't debug regex | High | `explain` feature showing pattern breakdown |
| Escape confusion | High | Auto-escaping of literals |
| Empty match loops | Medium | Handled internally like Gobo |
| ReDoS attacks | Medium | Timeout option, pattern analysis |
| Verbose APIs | High | One-liner convenience methods |
| Compile vs match confusion | High | Auto-compile on first use |

### 5. Innovation Opportunities

1. **Fluent Pattern Builder**: `regex.digit.one_or_more.then_literal("@").word.one_or_more`
2. **Named Pattern Library**: Common patterns built-in (email, URL, phone, etc.)
3. **Smart Validation**: `is_valid_email(s)` using pre-compiled patterns
4. **Result Objects**: Rich match results with named groups, positions, context
5. **Chainable Operations**: `text.regex_replace(...).regex_split(...)`
6. **Explain Mode**: Human-readable description of what pattern does
7. **Contract Integration**: Preconditions validate patterns at design time

---

## API Design

### Core Classes

```
SIMPLE_REGEX                    -- Main facade class
SIMPLE_REGEX_PATTERN           -- Compiled pattern (wraps RX_PCRE_REGULAR_EXPRESSION)
SIMPLE_REGEX_MATCH             -- Single match result
SIMPLE_REGEX_MATCH_LIST        -- Collection of all matches
SIMPLE_REGEX_BUILDER           -- Fluent pattern builder
SIMPLE_REGEX_PATTERNS          -- Library of common patterns
```

### Class: SIMPLE_REGEX

The main entry point providing static-like convenience and instance methods.

```eiffel
class SIMPLE_REGEX

create
    make,                       -- Empty, use fluent builder
    make_from_pattern           -- From pattern string

feature -- Quick Operations (class-level convenience)

    matches (a_pattern, a_subject: STRING): BOOLEAN
        -- Does subject contain a match for pattern?

    first_match (a_pattern, a_subject: STRING): detachable STRING
        -- First matching substring, or Void

    all_matches (a_pattern, a_subject: STRING): ARRAYED_LIST [STRING]
        -- All matching substrings

    replace_first (a_pattern, a_subject, a_replacement: STRING): STRING
        -- Replace first match

    replace_all (a_pattern, a_subject, a_replacement: STRING): STRING
        -- Replace all matches

    split (a_pattern, a_subject: STRING): ARRAYED_LIST [STRING]
        -- Split subject by pattern

feature -- Pattern Compilation

    compile (a_pattern: STRING)
        -- Compile pattern for repeated use
        require
            valid_pattern: is_valid_pattern (a_pattern)
        ensure
            compiled: is_compiled

    is_compiled: BOOLEAN

    is_valid_pattern (a_pattern: STRING): BOOLEAN
        -- Is pattern syntactically valid?

feature -- Instance Matching

    match (a_subject: STRING): SIMPLE_REGEX_MATCH
        -- Get match result object
        require
            compiled: is_compiled

    match_all (a_subject: STRING): SIMPLE_REGEX_MATCH_LIST
        -- Get all matches
        require
            compiled: is_compiled

feature -- Instance Replacement

    replace (a_subject, a_replacement: STRING): STRING
        -- Replace first match in subject

    replace_with (a_subject: STRING; a_replacer: FUNCTION [SIMPLE_REGEX_MATCH, STRING]): STRING
        -- Replace using function for dynamic replacement

feature -- Fluent Options

    case_insensitive: SIMPLE_REGEX
        -- Return self with case-insensitive matching

    multiline: SIMPLE_REGEX
        -- Return self with multiline mode (^ and $ match line boundaries)

    dotall: SIMPLE_REGEX
        -- Return self with dot matching newlines

    extended: SIMPLE_REGEX
        -- Return self with extended mode (whitespace/comments in pattern)

feature -- Diagnostics

    explain: STRING
        -- Human-readable explanation of the pattern

    last_error: detachable STRING
        -- Last compilation error message

end
```

### Class: SIMPLE_REGEX_MATCH

Rich match result object.

```eiffel
class SIMPLE_REGEX_MATCH

feature -- Status

    is_matched: BOOLEAN
        -- Did the match succeed?

    is_empty: BOOLEAN
        -- Is the matched string empty?

feature -- Access

    value: STRING
        -- The matched substring
        require
            matched: is_matched

    start_position: INTEGER
        -- Start position in subject (1-based)

    end_position: INTEGER
        -- End position in subject

    length: INTEGER
        -- Length of match

feature -- Groups

    group_count: INTEGER
        -- Number of captured groups (not including group 0)

    group (n: INTEGER): detachable STRING
        -- The n-th captured group (0 = whole match)
        require
            valid_group: n >= 0 and n <= group_count

    named_group (a_name: STRING): detachable STRING
        -- Captured group by name (if pattern uses named groups)

    groups: ARRAYED_LIST [detachable STRING]
        -- All captured groups as list

feature -- Context

    text_before: STRING
        -- Subject text before the match

    text_after: STRING
        -- Subject text after the match

    context (n: INTEGER): STRING
        -- n characters before and after match for context

end
```

### Class: SIMPLE_REGEX_BUILDER

Fluent builder for constructing patterns programmatically.

```eiffel
class SIMPLE_REGEX_BUILDER

create
    make

feature -- Literals

    literal (s: STRING): SIMPLE_REGEX_BUILDER
        -- Match literal string (auto-escaped)

    any_char: SIMPLE_REGEX_BUILDER
        -- Match any character (.)

feature -- Character Classes

    digit: SIMPLE_REGEX_BUILDER          -- \d
    non_digit: SIMPLE_REGEX_BUILDER      -- \D
    word_char: SIMPLE_REGEX_BUILDER      -- \w
    non_word_char: SIMPLE_REGEX_BUILDER  -- \W
    whitespace: SIMPLE_REGEX_BUILDER     -- \s
    non_whitespace: SIMPLE_REGEX_BUILDER -- \S

    one_of (chars: STRING): SIMPLE_REGEX_BUILDER
        -- Character class [chars]

    none_of (chars: STRING): SIMPLE_REGEX_BUILDER
        -- Negated character class [^chars]

    range (from_char, to_char: CHARACTER): SIMPLE_REGEX_BUILDER
        -- Character range [from-to]

feature -- Quantifiers

    zero_or_more: SIMPLE_REGEX_BUILDER   -- *
    one_or_more: SIMPLE_REGEX_BUILDER    -- +
    zero_or_one: SIMPLE_REGEX_BUILDER    -- ?
    exactly (n: INTEGER): SIMPLE_REGEX_BUILDER        -- {n}
    at_least (n: INTEGER): SIMPLE_REGEX_BUILDER       -- {n,}
    between (min, max: INTEGER): SIMPLE_REGEX_BUILDER -- {min,max}

    lazy: SIMPLE_REGEX_BUILDER
        -- Make previous quantifier non-greedy

feature -- Anchors

    start_of_string: SIMPLE_REGEX_BUILDER   -- ^
    end_of_string: SIMPLE_REGEX_BUILDER     -- $
    word_boundary: SIMPLE_REGEX_BUILDER     -- \b
    non_word_boundary: SIMPLE_REGEX_BUILDER -- \B

feature -- Groups

    group_start: SIMPLE_REGEX_BUILDER       -- (
    group_end: SIMPLE_REGEX_BUILDER         -- )
    named_group (name: STRING): SIMPLE_REGEX_BUILDER  -- (?P<name>
    non_capturing_group: SIMPLE_REGEX_BUILDER         -- (?:

feature -- Alternation

    or_else: SIMPLE_REGEX_BUILDER           -- |

feature -- Building

    pattern: STRING
        -- The constructed pattern string

    build: SIMPLE_REGEX
        -- Create compiled SIMPLE_REGEX from pattern

end
```

### Class: SIMPLE_REGEX_PATTERNS

Pre-built common patterns.

```eiffel
class SIMPLE_REGEX_PATTERNS

feature -- Email

    email: SIMPLE_REGEX
        -- Basic email pattern

    email_strict: SIMPLE_REGEX
        -- RFC 5322 compliant email

feature -- URLs

    url: SIMPLE_REGEX
        -- HTTP/HTTPS URL

    url_with_protocol: SIMPLE_REGEX
        -- URL requiring protocol

feature -- Numbers

    integer: SIMPLE_REGEX
        -- Integer with optional sign

    decimal: SIMPLE_REGEX
        -- Decimal number

    scientific: SIMPLE_REGEX
        -- Scientific notation

feature -- Identifiers

    identifier: SIMPLE_REGEX
        -- Programming identifier [a-zA-Z_][a-zA-Z0-9_]*

    uuid: SIMPLE_REGEX
        -- UUID format

feature -- Dates/Times

    iso_date: SIMPLE_REGEX
        -- YYYY-MM-DD

    iso_datetime: SIMPLE_REGEX
        -- ISO 8601 datetime

    time_24h: SIMPLE_REGEX
        -- HH:MM:SS

feature -- Phone Numbers

    phone_us: SIMPLE_REGEX
        -- US phone format

    phone_international: SIMPLE_REGEX
        -- International format with country code

feature -- Network

    ipv4: SIMPLE_REGEX
        -- IPv4 address

    ipv6: SIMPLE_REGEX
        -- IPv6 address

    mac_address: SIMPLE_REGEX
        -- MAC address

feature -- File Paths

    windows_path: SIMPLE_REGEX
        -- Windows file path

    unix_path: SIMPLE_REGEX
        -- Unix file path

    file_extension (ext: STRING): SIMPLE_REGEX
        -- Files with specific extension

end
```

---

## Contract Strategy

### Preconditions

```eiffel
-- Pattern validity
compile (a_pattern: STRING)
    require
        pattern_not_empty: not a_pattern.is_empty
        -- Runtime check: is_valid_pattern(a_pattern) -- can't be precondition since expensive

-- Compiled state
match (a_subject: STRING): SIMPLE_REGEX_MATCH
    require
        compiled: is_compiled
        subject_not_void: a_subject /= Void

-- Group access
group (n: INTEGER): detachable STRING
    require
        matched: is_matched
        valid_group: n >= 0 and n <= group_count
```

### Postconditions

```eiffel
-- Compilation
compile (a_pattern: STRING)
    ensure
        compiled_or_error: is_compiled or last_error /= Void
        pattern_stored: pattern.same_string (a_pattern)

-- Matching
match (a_subject: STRING): SIMPLE_REGEX_MATCH
    ensure
        result_attached: Result /= Void
        subject_stored: Result.subject = a_subject
```

### Class Invariants

```eiffel
invariant
    compiled_has_pattern: is_compiled implies pattern /= Void
    error_implies_not_compiled: last_error /= Void implies not is_compiled
    match_position_valid: is_matched implies (start_position >= 1 and end_position <= subject.count)
```

---

## Integration Plan

### FOUNDATION_API Integration

```eiffel
feature -- Regex Operations

    new_regex (a_pattern: STRING): SIMPLE_REGEX
        -- Create regex from pattern

    regex_matches (a_pattern, a_subject: STRING): BOOLEAN
        -- Quick match check

    regex_replace (a_pattern, a_subject, a_replacement: STRING): STRING
        -- Quick replace

    regex_split (a_pattern, a_subject: STRING): ARRAYED_LIST [STRING]
        -- Quick split

    regex_patterns: SIMPLE_REGEX_PATTERNS
        -- Access to common patterns
```

### SERVICE_API Integration

Uses FOUNDATION_API's regex features directly.

---

## Test Strategy

### Unit Tests (Target: 50+ tests)

1. **Basic Matching** (10 tests)
   - Simple literal match
   - No match returns proper state
   - Empty pattern handling
   - Empty subject handling
   - Unicode subject matching

2. **Character Classes** (8 tests)
   - \d, \D, \w, \W, \s, \S
   - Custom character classes
   - Negated classes
   - Ranges

3. **Quantifiers** (8 tests)
   - *, +, ?, {n}, {n,}, {n,m}
   - Greedy vs lazy
   - Edge cases (0 matches, many matches)

4. **Groups** (8 tests)
   - Capturing groups
   - Named groups
   - Non-capturing groups
   - Nested groups

5. **Replacement** (8 tests)
   - Simple replacement
   - Group references in replacement
   - Replace all
   - Empty replacement

6. **Split** (5 tests)
   - Basic split
   - Split with groups
   - Edge cases

7. **Builder** (8 tests)
   - Fluent building
   - Complex patterns
   - Builder reset

8. **Common Patterns** (10 tests)
   - Email validation
   - URL validation
   - Phone numbers
   - Dates

---

## Implementation Roadmap

### Phase 1: Core (MVP)
- SIMPLE_REGEX with convenience methods
- SIMPLE_REGEX_MATCH
- Basic compile/match/replace/split
- 25 tests passing

### Phase 2: Fluent API
- SIMPLE_REGEX_BUILDER
- SIMPLE_REGEX_MATCH_LIST
- Option chaining
- 40 tests passing

### Phase 3: Patterns Library
- SIMPLE_REGEX_PATTERNS
- Common patterns
- Explain feature
- 50+ tests passing

### Phase 4: Integration
- FOUNDATION_API integration
- Documentation
- Examples

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Gobo API changes | Pin to known working version, thin wrapper |
| Performance overhead | Cache compiled patterns, benchmark |
| Pattern compatibility | Test against PCRE test suite |
| Unicode edge cases | Use Gobo's Unicode handling |

---

## Sources

### Specifications
- [POSIX.1-2024 Base Specifications](https://pubs.opengroup.org/onlinepubs/9799919799.2024edition/mindex.html)
- [PCRE2 Documentation](https://pcre2project.github.io/pcre2/)
- [Unicode UTS #18](https://www.unicode.org/reports/tr18/)
- [ECMAScript 2024](https://tc39.es/ecma262/2024/)

### Library Documentation
- [Python regex module](https://pypi.org/project/regex/)
- [Python re2](https://github.com/google/re2)
- [Rust regex crate](https://docs.rs/regex/latest/regex/)
- [Go regexp package](https://pkg.go.dev/regexp)

### Fluent Builders
- [FluentRegex .NET](https://github.com/bcwood/FluentRegex)
- [regex-builder Java](https://github.com/sgreben/regex-builder)
- [fluent-regex TypeScript](https://github.com/gillyb/fluent-regex)

### Pain Points Research
- [Regular Expressions: Now You Have Two Problems](https://blog.codinghorror.com/regular-expressions-now-you-have-two-problems/)
- [What do regular expression bugs look like?](https://chuniversiteit.nl/papers/regular-expression-bugs)
- [SonarSource: Regular expressions present challenges](https://www.sonarsource.com/blog/regular-expressions-present-challenges/)
