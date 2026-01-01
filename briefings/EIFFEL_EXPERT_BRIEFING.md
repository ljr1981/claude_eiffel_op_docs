## String Conversion Rules (NO OBSOLETE CODE)

**NEVER use implicit `as_string_8` conversions.** Always use explicit conversions:

| Situation | BAD (obsolete) | GOOD |
|-----------|----------------|------|
| substring on READABLE_STRING_8 | `l_str := readable.substring(1,5)` | `l_str := readable.substring(1,5).to_string_8` |
| Environment variable | `l_path := env.get("HOME")` | `l_path := env.get("HOME").to_string_8` |
| String concatenation | `err := "Error: " + msg` (if msg is STRING_32) | `err := "Error: " + msg.to_string_8` |

**Key types:**
- `READABLE_STRING_8` - parent of STRING_8, substring returns READABLE_STRING_8
- `STRING` = `STRING_8` - use this for local variables
- `STRING_32` - Unicode strings, need explicit `.to_string_8` for conversion

**Watch for these patterns (all cause obsolete warnings):**
1. Assigning `READABLE_STRING_8.substring()` to `STRING`
2. Concatenating `STRING_32` with `STRING_8` using `+`
3. Getting environment variables (often return STRING_32)
4. File path operations (PATH returns STRING_32)

## Common Eiffel Pitfalls (Lessons Learned)

### Operators

| Wrong | Right | Notes |
|-------|-------|-------|
| `n \ 2` | `n \ 2` | Integer modulo uses DOUBLE backslash |
| `x.sqrt` | `{DOUBLE_MATH}.sqrt (x)` | Math functions are not methods on REAL_64 |
| `x.log` | `{DOUBLE_MATH}.log (x)` | Same for log, sin, cos, etc. |

### Iteration Cursors (across loops)

When using `across collection as ic loop`, `ic` gives you the **item**, not the cursor.
To access cursor methods, use `@ic`:

```eiffel
-- WRONG: ic is the item (e.g., STRING), not the cursor
across my_list as ic loop
    if ic.cursor_index > 1 then ...  -- ERROR: STRING has no cursor_index
    if ic.is_last then ...           -- ERROR: STRING has no is_last
end

-- RIGHT: Use @ to access the actual cursor
across my_list as ic loop
    if @ic.cursor_index > 1 then ... -- OK: cursor has cursor_index
    if @ic.is_last then ...          -- OK: cursor has is_last
end

-- For HASH_TABLE, to get key:
across my_hash as ic loop
    key := @ic.key    -- @ needed to get key from cursor
    val := ic         -- ic is the value directly
end
```

### TUPLE Labels

TUPLE labels cannot shadow existing TUPLE features:

```eiffel
-- WRONG: 'count' is already a TUPLE feature
l_data: TUPLE [word: STRING; count: INTEGER]

-- RIGHT: Use different name
l_data: TUPLE [word: STRING; cnt: INTEGER]
```

### Class Names

- Class names are **globally unique** in Eiffel
- Cannot have `FACTORIAL` in both `tier2/` and `tier4/` directories
- Each Rosetta Code task = ONE solution class

### String Escape Sequences

In Eiffel strings:
- `%%` outputs `%`
- `%"` outputs `"` (does NOT close the string)
- `%N` newline
- `%T` tab

To output `%"` literally: use `%%%"` (outputs `%` then `"`)

### Preconditions

Features used in preconditions must be exported to all clients:

```eiffel
-- WRONG: helper is private but used in precondition
feature {NONE}
    helper (s: STRING): INTEGER do ... end

feature
    process (s: STRING)
        require
            valid: helper (s) > 0  -- ERROR: clients can't see helper
        do ...

-- RIGHT: Export helper or move precondition logic
feature
    helper (s: STRING): INTEGER do ... end
```

### Development Workflow

1. **Compile in small batches** - Do not write 100 files then compile
2. **Use simple_* libraries** - Not ISE stdlib (e.g., `SIMPLE_PROCESS` not `PROCESS`)
3. **Document dependencies** - When using simple_* in Rosetta solutions, add to notes clause:
   ```eiffel
   note
       description: "Solution using simple_process library"
       dependencies: "simple_process (https://github.com/simple-eiffel/simple_process)"
   ```