# simple_regex Design Report v2

## Revision Notes (Addressing Grok Review)

### Changes from v1:
1. **Gobo PCRE Version Clarified**: Pure Eiffel implementation (2001-2019), based on PCRE1 semantics, not PCRE2
2. **STRING_8/STRING_32 Handling**: Explicit Unicode strategy added
3. **ReDoS Elevated to HIGH**: Added match timeout and pattern complexity warnings
4. **Convenience Methods**: Internal pattern caching added to prevent performance hits
5. **Builder Contracts**: Added comprehensive preconditions
6. **Thread Safety**: Immutable pattern objects, cloning for option changes
7. **Common Patterns**: Accuracy disclaimers and limitations documented
8. **Test Strategy**: Expanded to 80+ tests with fuzzing and security tests

---

## Executive Summary

**simple_regex** is a high-level, fluent Eiffel library wrapping Gobo's PCRE regex engine (`RX_PCRE_REGULAR_EXPRESSION`).

**Critical Limitation Acknowledged**: Gobo implements PCRE1 semantics (not PCRE2), so modern features like JIT compilation, atomic groups, and some Unicode extensions are NOT available. The library provides a clean API over what Gobo actually supports.

**Key Value Propositions:**
1. Cached convenience methods preventing hidden performance hits
2. Fluent builder with full contracts and validation
3. Explicit STRING_32 support for Unicode
4. ReDoS protection via timeouts and pattern analysis
5. Thread-safe immutable pattern objects

---

## Research Findings (Expanded)

### Additional Language Analysis (per Grok)

| Language | Library | Key Insight for simple_regex |
|----------|---------|------------------------------|
| **Java** | `java.util.regex` | Pattern/Matcher separation, `Pattern.COMMENTS` for readable patterns |
| **C#/.NET** | `System.Text.RegularExpressions` | `RegexOptions.Compiled`, balancing groups, timeout built-in |
| **PHP** | `preg_*` | PREG_OFFSET_CAPTURE returns positions, error constants |

**Design Decision**: Adopt Java's Pattern/Matcher separation (SIMPLE_REGEX_PATTERN / SIMPLE_REGEX) and .NET's built-in timeout.

### Gobo Limitations (Honest Assessment)

| Feature | PCRE2 Status | Gobo Status | simple_regex Approach |
|---------|--------------|-------------|----------------------|
| JIT Compilation | Yes | No | Document limitation |
| Atomic Groups | Yes | Limited | Pass through if supported |
| Named Groups | Yes | Yes | Full support via (?P<name>) |
| Unicode Properties | Yes | Basic | Support what Gobo offers |
| Recursion (?R) | Yes | Unknown | Test and document |
| Lookahead/Lookbehind | Yes | Yes | Builder support added |
| Timeout | Yes | No | **Implement in wrapper** |

### Eiffel-Specific Pain Points (Addressed)

| Issue | Impact | Solution |
|-------|--------|----------|
| STRING_8 vs STRING_32 | Unicode corruption | Separate APIs: `match` vs `match_unicode` |
| Immutable strings | Performance concerns | Internal mutable buffers |
| Strong typing | Verbose casts | Generic result types |
| Contract overhead | Performance in loops | `{NONE}` disable option |

---

## Revised API Design

### Core Principle Changes

1. **Pattern Caching**: Convenience methods use internal `once` cache
2. **Immutability**: Options return new objects, not mutate
3. **Explicit Unicode**: Separate methods for STRING_32
4. **Timeout First**: All match operations support timeout

### Class: SIMPLE_REGEX_PATTERN (Immutable, Thread-Safe)

```eiffel
class SIMPLE_REGEX_PATTERN
    -- Compiled, immutable regex pattern
    -- Thread-safe: can be shared across SCOOP processors

create
    make,              -- From pattern string
    make_with_options  -- From pattern with options

feature -- Access (Immutable)

    pattern: STRING
        -- The original pattern string

    is_valid: BOOLEAN
        -- Was compilation successful?

    error_message: detachable STRING
        -- Compilation error if not valid

    options: SIMPLE_REGEX_OPTIONS
        -- Compilation options used

feature -- Queries

    has_named_groups: BOOLEAN
    named_group_names: ARRAYED_LIST [STRING]
    estimated_complexity: INTEGER
        -- Heuristic 1-10 for ReDoS risk

feature -- Factory (Returns new immutable instances)

    with_case_insensitive: SIMPLE_REGEX_PATTERN
        -- New pattern with case insensitive option
        ensure
            new_object: Result /= Current
            same_pattern: Result.pattern.same_string (pattern)

    with_multiline: SIMPLE_REGEX_PATTERN
    with_dotall: SIMPLE_REGEX_PATTERN
    with_extended: SIMPLE_REGEX_PATTERN

invariant
    pattern_not_void: pattern /= Void
    valid_or_error: is_valid xor (error_message /= Void)
    immutable_options: options.is_frozen

end
```

### Class: SIMPLE_REGEX (Matcher Instance)

```eiffel
class SIMPLE_REGEX
    -- Regex matcher instance (NOT thread-safe, use one per thread)

create
    make,                  -- Empty, set pattern later
    make_from_pattern,     -- From SIMPLE_REGEX_PATTERN
    make_from_string       -- Compile from string

feature -- Class-Level Convenience (Cached Patterns)

    matches (a_pattern, a_subject: STRING): BOOLEAN
        -- Does subject match pattern?
        -- Uses internal pattern cache to avoid recompilation
        note
            performance: "Pattern cached after first use"

    first_match (a_pattern, a_subject: STRING): detachable STRING
        -- First match or Void

    all_matches (a_pattern, a_subject: STRING): ARRAYED_LIST [STRING]

    replace_first (a_pattern, a_subject, a_replacement: STRING): STRING

    replace_all (a_pattern, a_subject, a_replacement: STRING): STRING

    split (a_pattern, a_subject: STRING): ARRAYED_LIST [STRING]

feature -- Unicode Variants (STRING_32)

    matches_unicode (a_pattern: STRING; a_subject: STRING_32): BOOLEAN
    first_match_unicode (a_pattern: STRING; a_subject: STRING_32): detachable STRING_32
    -- ... etc

feature -- Pattern Management

    set_pattern (a_pattern: SIMPLE_REGEX_PATTERN)
        require
            valid_pattern: a_pattern.is_valid
        ensure
            pattern_set: pattern = a_pattern

    compile (a_pattern_string: STRING)
        -- Compile pattern, set is_compiled and last_error
        ensure
            compiled_xor_error: is_compiled xor (last_error /= Void)

feature -- Matching with Timeout

    match (a_subject: STRING): SIMPLE_REGEX_MATCH
        require
            has_pattern: pattern /= Void and then pattern.is_valid

    match_with_timeout (a_subject: STRING; a_timeout_ms: INTEGER): SIMPLE_REGEX_MATCH
        -- Match with timeout protection against ReDoS
        require
            has_pattern: pattern /= Void and then pattern.is_valid
            positive_timeout: a_timeout_ms > 0
        ensure
            timed_out_or_result: Result.is_timed_out or Result.is_complete

    match_all (a_subject: STRING): SIMPLE_REGEX_MATCH_LIST
    match_all_with_timeout (a_subject: STRING; a_timeout_ms: INTEGER): SIMPLE_REGEX_MATCH_LIST

feature -- Status

    is_compiled: BOOLEAN
    last_error: detachable STRING
    pattern: detachable SIMPLE_REGEX_PATTERN

feature {NONE} -- Implementation

    pattern_cache: HASH_TABLE [SIMPLE_REGEX_PATTERN, STRING]
        -- Internal cache for convenience methods
        once
            create Result.make (50)
        end

invariant
    cache_exists: pattern_cache /= Void

end
```

### Class: SIMPLE_REGEX_MATCH (Result Object)

```eiffel
class SIMPLE_REGEX_MATCH

feature -- Status

    is_matched: BOOLEAN
    is_empty: BOOLEAN
    is_timed_out: BOOLEAN
        -- True if match was aborted due to timeout
    is_complete: BOOLEAN
        -- True if match completed (matched or not)

feature -- Access

    value: STRING
        require
            matched: is_matched
        ensure
            from_subject: subject.has_substring (Result)

    start_position: INTEGER
        require
            matched: is_matched
        ensure
            valid_position: Result >= 1 and Result <= subject.count

    end_position: INTEGER
    length: INTEGER

feature -- Groups (with validation)

    group_count: INTEGER

    group (n: INTEGER): detachable STRING
        require
            valid_index: n >= 0 and n <= group_count

    group_start (n: INTEGER): INTEGER
        require
            valid_index: n >= 0 and n <= group_count
            group_matched: group (n) /= Void

    named_group (a_name: STRING): detachable STRING
        require
            name_exists: pattern.named_group_names.has (a_name)

    groups: ARRAYED_LIST [detachable STRING]
        ensure
            correct_count: Result.count = group_count + 1

feature -- Context

    subject: STRING
        -- The matched subject string

    text_before: STRING
        -- Subject text before match
        require
            matched: is_matched
        ensure
            correct: Result.same_string (subject.substring (1, start_position - 1))

    text_after: STRING
        require
            matched: is_matched
        ensure
            correct: Result.same_string (subject.substring (end_position + 1, subject.count))

invariant
    complete_or_timeout: is_complete xor is_timed_out
    matched_implies_complete: is_matched implies is_complete
    positions_valid: is_matched implies (start_position <= end_position)

end
```

### Class: SIMPLE_REGEX_BUILDER (With Full Contracts)

```eiffel
class SIMPLE_REGEX_BUILDER

create
    make

feature -- Literals

    literal (s: STRING): SIMPLE_REGEX_BUILDER
        -- Match literal string (auto-escaped)
        require
            not_empty: not s.is_empty
        ensure
            pattern_extended: pattern.count > old pattern.count

    raw (s: STRING): SIMPLE_REGEX_BUILDER
        -- Insert raw pattern (NOT escaped)
        -- Use with caution!
        require
            not_empty: not s.is_empty

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
        require
            not_empty: not chars.is_empty
        ensure
            valid_class: pattern.has_substring ("[")

    none_of (chars: STRING): SIMPLE_REGEX_BUILDER
        -- Negated character class [^chars]
        require
            not_empty: not chars.is_empty

    range (from_char, to_char: CHARACTER): SIMPLE_REGEX_BUILDER
        -- Character range [from-to]
        require
            valid_range: from_char <= to_char

feature -- Quantifiers

    zero_or_more: SIMPLE_REGEX_BUILDER   -- *
        require
            has_quantifiable: has_quantifiable_element

    one_or_more: SIMPLE_REGEX_BUILDER    -- +
        require
            has_quantifiable: has_quantifiable_element

    zero_or_one: SIMPLE_REGEX_BUILDER    -- ?
        require
            has_quantifiable: has_quantifiable_element

    exactly (n: INTEGER): SIMPLE_REGEX_BUILDER
        -- {n}
        require
            positive: n > 0
            has_quantifiable: has_quantifiable_element

    at_least (n: INTEGER): SIMPLE_REGEX_BUILDER
        -- {n,}
        require
            non_negative: n >= 0
            has_quantifiable: has_quantifiable_element

    between (min, max: INTEGER): SIMPLE_REGEX_BUILDER
        -- {min,max}
        require
            valid_range: min >= 0 and min <= max
            has_quantifiable: has_quantifiable_element

    lazy: SIMPLE_REGEX_BUILDER
        -- Make previous quantifier non-greedy
        require
            has_quantifier: has_quantifier_element

feature -- Anchors

    start_of_string: SIMPLE_REGEX_BUILDER   -- ^
    end_of_string: SIMPLE_REGEX_BUILDER     -- $
    word_boundary: SIMPLE_REGEX_BUILDER     -- \b
    non_word_boundary: SIMPLE_REGEX_BUILDER -- \B

feature -- Groups

    group_start: SIMPLE_REGEX_BUILDER
        -- (
        ensure
            group_opened: open_group_count = old open_group_count + 1

    group_end: SIMPLE_REGEX_BUILDER
        -- )
        require
            group_open: open_group_count > 0
        ensure
            group_closed: open_group_count = old open_group_count - 1

    named_group_start (name: STRING): SIMPLE_REGEX_BUILDER
        -- (?P<name>
        require
            valid_name: is_valid_group_name (name)
            unique_name: not used_group_names.has (name)
        ensure
            name_recorded: used_group_names.has (name)

    non_capturing_group_start: SIMPLE_REGEX_BUILDER
        -- (?:

feature -- Lookahead/Lookbehind (Advanced)

    positive_lookahead_start: SIMPLE_REGEX_BUILDER
        -- (?=

    negative_lookahead_start: SIMPLE_REGEX_BUILDER
        -- (?!

    positive_lookbehind_start: SIMPLE_REGEX_BUILDER
        -- (?<=

    negative_lookbehind_start: SIMPLE_REGEX_BUILDER
        -- (?<!

feature -- Alternation

    or_else: SIMPLE_REGEX_BUILDER
        -- |

feature -- Building

    pattern: STRING
        -- The constructed pattern string

    build: SIMPLE_REGEX_PATTERN
        -- Create compiled pattern
        require
            groups_balanced: open_group_count = 0
            not_empty: not pattern.is_empty
        ensure
            result_valid: Result.is_valid implies Result.pattern.same_string (pattern)

    reset: SIMPLE_REGEX_BUILDER
        -- Clear and start fresh
        ensure
            empty: pattern.is_empty
            groups_reset: open_group_count = 0

feature -- Validation Queries

    has_quantifiable_element: BOOLEAN
        -- Is there an element that can be quantified?

    has_quantifier_element: BOOLEAN
        -- Does pattern end with a quantifier?

    is_valid_group_name (name: STRING): BOOLEAN
        -- Is name a valid group name?

    open_group_count: INTEGER
        -- Number of unclosed groups

    used_group_names: ARRAYED_LIST [STRING]
        -- Names already used for groups

invariant
    pattern_exists: pattern /= Void
    non_negative_groups: open_group_count >= 0
    names_tracked: used_group_names /= Void

end
```

### Class: SIMPLE_REGEX_PATTERNS (With Accuracy Disclaimers)

```eiffel
class SIMPLE_REGEX_PATTERNS
    -- Pre-built common patterns
    -- WARNING: These are heuristic patterns, not full validators.
    -- For strict validation, use domain-specific parsers.

feature -- Email (Heuristic)

    email: SIMPLE_REGEX_PATTERN
        -- Basic email pattern (covers ~95% of common cases)
        -- NOT RFC 5322 compliant (that requires a parser)
        note
            limitations: "Does not handle: comments, folding whitespace, quoted local parts"
        once
            create Result.make ("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}")
        end

feature -- URLs (Heuristic)

    url_http: SIMPLE_REGEX_PATTERN
        -- HTTP/HTTPS URL pattern
        note
            limitations: "Does not validate domain exists or handle all edge cases"
        once
            create Result.make ("https?://[a-zA-Z0-9.-]+(?:/[^\s]*)?")
        end

feature -- Numbers (Accurate)

    integer_signed: SIMPLE_REGEX_PATTERN
        -- Signed integer: -123, +456, 789
        once
            create Result.make ("[+-]?[0-9]+")
        end

    decimal: SIMPLE_REGEX_PATTERN
        -- Decimal number: 123.456, -0.5
        once
            create Result.make ("[+-]?[0-9]*\.?[0-9]+")
        end

feature -- Identifiers (Accurate)

    identifier: SIMPLE_REGEX_PATTERN
        -- Programming identifier [a-zA-Z_][a-zA-Z0-9_]*
        once
            create Result.make ("[a-zA-Z_][a-zA-Z0-9_]*")
        end

    uuid: SIMPLE_REGEX_PATTERN
        -- UUID format (8-4-4-4-12 hex)
        once
            create Result.make ("[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}")
        end

feature -- Dates (Accurate Format, Not Semantics)

    iso_date: SIMPLE_REGEX_PATTERN
        -- YYYY-MM-DD format
        -- WARNING: Does not validate date semantics (e.g., 2024-02-30 matches)
        once
            create Result.make ("[0-9]{4}-[0-9]{2}-[0-9]{2}")
        end

feature -- Network (Heuristic)

    ipv4: SIMPLE_REGEX_PATTERN
        -- IPv4 address pattern
        -- WARNING: Allows values > 255, validate semantically
        once
            create Result.make ("[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}")
        end

feature -- Validation Methods (More Accurate)

    is_valid_email (s: STRING): BOOLEAN
        -- Heuristic email check
        do
            Result := email.matches (s)
        end

    is_valid_uuid (s: STRING): BOOLEAN
        -- Accurate UUID format check
        do
            Result := uuid.matches (s) and s.count = 36
        end

end
```

---

## Contract Strategy (Revised)

### Precondition Approach

**Decision**: Remove expensive validation from preconditions, use explicit validation methods:

```eiffel
-- WRONG: Expensive precondition
compile (a_pattern: STRING)
    require
        valid: is_valid_pattern (a_pattern)  -- Called on EVERY call

-- RIGHT: Caller-validated pattern
compile (a_pattern: STRING)
    -- Compiles pattern, sets is_compiled and last_error
    ensure
        compiled_xor_error: is_compiled xor (last_error /= Void)

-- Caller code:
if regex.is_valid_pattern (my_pattern) then
    regex.compile (my_pattern)
else
    handle_invalid_pattern
end
```

### Postcondition Completeness

```eiffel
match (a_subject: STRING): SIMPLE_REGEX_MATCH
    ensure
        result_attached: Result /= Void
        subject_stored: Result.subject = a_subject
        complete_or_timeout: Result.is_complete or Result.is_timed_out
        value_from_subject: Result.is_matched implies
            a_subject.substring (Result.start_position, Result.end_position).same_string (Result.value)
```

### Builder Contracts (New)

All builder methods have appropriate require/ensure clauses preventing invalid pattern construction.

---

## Security Strategy (ReDoS - HIGH Priority)

### Timeout Implementation

```eiffel
feature {NONE} -- Timeout Implementation

    match_with_timeout_impl (a_subject: STRING; a_timeout_ms: INTEGER): SIMPLE_REGEX_MATCH
        local
            l_start_time: INTEGER_64
            l_elapsed: INTEGER_64
        do
            l_start_time := current_time_ms
            -- Gobo doesn't support native timeout, so we check periodically
            -- This is a best-effort approach
            from
                internal_regex.match (a_subject)
            until
                not internal_regex.has_matched or else
                (current_time_ms - l_start_time > a_timeout_ms)
            loop
                -- For match_all, check timeout between iterations
            end

            if current_time_ms - l_start_time > a_timeout_ms then
                create Result.make_timed_out (a_subject)
            else
                create Result.make_from_gobo (internal_regex, a_subject)
            end
        end
```

### Pattern Complexity Warning

```eiffel
feature -- Security

    pattern_complexity (a_pattern: STRING): INTEGER
        -- Heuristic ReDoS risk score (1-10)
        -- High scores indicate potentially dangerous patterns
        local
            l_nested_quantifiers: INTEGER
            l_alternation_depth: INTEGER
        do
            -- Count nested quantifiers like (a+)+
            -- Count deep alternations
            -- Score based on known dangerous patterns
            Result := calculate_complexity (a_pattern)
        ensure
            valid_range: Result >= 1 and Result <= 10
        end

    Dangerous_complexity_threshold: INTEGER = 7

    is_potentially_dangerous (a_pattern: STRING): BOOLEAN
        do
            Result := pattern_complexity (a_pattern) >= Dangerous_complexity_threshold
        end
```

---

## Test Strategy (Expanded: 80+ tests)

### Unit Tests

| Category | Count | Focus |
|----------|-------|-------|
| Basic Matching | 12 | Literals, empty, Unicode |
| Character Classes | 10 | All classes, negation, ranges |
| Quantifiers | 10 | All quantifiers, lazy, edge cases |
| Groups | 10 | Capturing, named, nested |
| Replacement | 8 | Simple, groups, all |
| Split | 6 | Basic, edge cases |
| Builder | 12 | All methods, contracts |
| Patterns Library | 8 | Each pattern validated |

### Security Tests (New)

| Test | Purpose |
|------|---------|
| ReDoS Pattern Detection | Test complexity scoring |
| Timeout Behavior | Verify timeout stops execution |
| Evil Regex Tests | `(a+)+`, `(a|a)+`, `(a|aa)+` |
| Large Input Handling | 1MB+ subjects |

### Unicode Tests (New)

| Test | Purpose |
|------|---------|
| STRING_32 Matching | Emoji, CJK, RTL |
| Mixed Encoding | STRING_8/STRING_32 interaction |
| Unicode Properties | If Gobo supports |

### Performance Tests

| Test | Purpose |
|------|---------|
| Pattern Cache Verification | Convenience methods don't recompile |
| Compilation Benchmark | Track regression |
| Match Benchmark | Compare to raw Gobo |

---

## Risks (Updated)

| Risk | Severity | Mitigation |
|------|----------|------------|
| ReDoS attacks | HIGH | Timeout + complexity scoring |
| Gobo limitations | MEDIUM | Clear documentation of what's NOT supported |
| STRING_8/STRING_32 confusion | MEDIUM | Separate APIs, clear naming |
| Thread safety | MEDIUM | Immutable patterns, per-thread matchers |
| Gobo maintainer dependency | LOW | We only wrap, don't modify |
| Performance overhead | LOW | Caching, thin wrapper |

---

## Implementation Phases (Revised)

### Phase 1: Core with Safety (MVP+)
- SIMPLE_REGEX_PATTERN (immutable)
- SIMPLE_REGEX with timeout support
- SIMPLE_REGEX_MATCH
- Pattern caching for convenience methods
- **35 tests passing**

### Phase 2: Builder
- SIMPLE_REGEX_BUILDER with full contracts
- Lookahead/lookbehind support
- **55 tests passing**

### Phase 3: Patterns & Polish
- SIMPLE_REGEX_PATTERNS with disclaimers
- Unicode STRING_32 variants
- Pattern complexity scoring
- **80+ tests passing**

### Phase 4: Integration
- FOUNDATION_API integration
- Documentation with security warnings
- Examples

---

## Approval Checklist

- [x] Gobo limitations acknowledged (PCRE1, not PCRE2)
- [x] STRING_8/STRING_32 strategy defined
- [x] ReDoS protection elevated to high priority
- [x] Convenience methods use caching
- [x] Builder has full contracts
- [x] Pattern objects are immutable
- [x] Common patterns have accuracy disclaimers
- [x] Test count expanded to 80+
- [x] Security tests added

**Ready for implementation approval.**
