# simple_codegen Claude Action Logging Specification

**Purpose:** Track all Claude actions during code generation for gap analysis and after-action reports.

## The Problem

When Claude operates under simple_codegen's direction, there's no record of:
1. What Claude did vs. what it was instructed to do
2. Gap-fills (actions Claude took because instructions were incomplete)
3. Rebellions (actions Claude took contrary to instructions)
4. Error fixes (actions Claude took to fix mistakes)

This data is critical for improving simple_codegen's prompts and workflow.

## Proposed Solution

Add a `log` command to simple_codegen CLI that Claude calls BEFORE performing any action:

```bash
simple_codegen log --session <name> --category <CAT> --action "<description>"
```

### Categories

| Category | Description |
|----------|-------------|
| `INSTRUCTED` | Following explicit simple_codegen instruction |
| `GAP_FILL` | Doing something necessary but not specified |
| `REBEL` | Going against or modifying instructions |
| `ERROR_FIX` | Correcting mistakes from prior steps |
| `DECISION` | Made a choice between alternatives |

### Examples

```bash
# Claude follows a prompt to generate CHART_CONFIG
simple_codegen log --session simple_chart --category INSTRUCTED --action "Generated CHART_CONFIG class per prompt 003"

# Claude notices the prompt lacks detail and adds it
simple_codegen log --session simple_chart --category GAP_FILL --action "Enhanced prompt 001 with chart type requirements - original too sparse"

# Claude changes approach because generated code won't work
simple_codegen log --session simple_chart --category REBEL --action "Using index-based iteration instead of across - SIMPLE_JSON_ARRAY not ITERABLE"

# Claude fixes compilation error
simple_codegen log --session simple_chart --category ERROR_FIX --action "Changed as_real_64 to as_real - method name mismatch"
```

## Implementation

### New Class: SCG_ACTION_LOGGER

```eiffel
class SCG_ACTION_LOGGER

create
    make

feature {NONE} -- Initialization

    make (a_session_path: STRING)
            -- Create logger for session.
        local
            l_log_path: STRING
        do
            l_log_path := a_session_path + "/claude_actions.log"
            create logger.make_to_file (l_log_path)
            logger.set_json_output (True)
            logger.with_field ("session", a_session_path)
        end

feature -- Logging

    log_action (a_category, a_action: STRING)
            -- Log a Claude action.
        require
            valid_category: is_valid_category (a_category)
            action_not_empty: not a_action.is_empty
        do
            logger.info_fields (a_action, <<
                ["category", a_category],
                ["timestamp", (create {SIMPLE_DATE_TIME}.make_now).to_iso8601]
            >>)
        end

feature -- Categories

    Category_instructed: STRING = "INSTRUCTED"
    Category_gap_fill: STRING = "GAP_FILL"
    Category_rebel: STRING = "REBEL"
    Category_error_fix: STRING = "ERROR_FIX"
    Category_decision: STRING = "DECISION"

    is_valid_category (a_cat: STRING): BOOLEAN
        do
            Result := a_cat.same_string (Category_instructed) or
                      a_cat.same_string (Category_gap_fill) or
                      a_cat.same_string (Category_rebel) or
                      a_cat.same_string (Category_error_fix) or
                      a_cat.same_string (Category_decision)
        end

feature {NONE} -- Implementation

    logger: SIMPLE_LOGGER

end
```

### CLI Command Handler in SCG_CLI_APP

```eiffel
process_log_command
        -- Handle: simple_codegen log --session X --category Y --action "Z"
    local
        l_session, l_category, l_action: STRING
        l_logger: SCG_ACTION_LOGGER
    do
        l_session := get_option_value ("--session")
        l_category := get_option_value ("--category")
        l_action := get_option_value ("--action")

        create l_logger.make (session_path (l_session))
        l_logger.log_action (l_category, l_action)

        io.put_string ("Logged: [" + l_category + "] " + l_action + "%N")
    end
```

## Output Format

JSON log file at `sessions/<name>/claude_actions.log`:

```json
{"timestamp":"2026-01-16T17:00:00Z","level":"info","message":"Generated CHART_CONFIG class per prompt 003","category":"INSTRUCTED","session":"simple_chart"}
{"timestamp":"2026-01-16T17:01:00Z","level":"info","message":"Enhanced prompt 001 with chart type requirements","category":"GAP_FILL","session":"simple_chart"}
{"timestamp":"2026-01-16T17:02:00Z","level":"info","message":"Using index-based iteration instead of across","category":"REBEL","session":"simple_chart"}
```

## After-Action Report Generation

Add command to generate summary:

```bash
simple_codegen after-action --session simple_chart
```

Output:
```
=== After-Action Report: simple_chart ===

Actions by Category:
  INSTRUCTED: 12 (57%)
  GAP_FILL:    4 (19%)
  ERROR_FIX:   4 (19%)
  REBEL:       1 (5%)
  DECISION:    0 (0%)

Gap Analysis:
  - Prompt 001: Enhanced with chart type requirements (too sparse)
  - Prompt 003: Added example usage (missing context)
  - Assemble: Fixed output path (bug, not instruction issue)

Error Patterns:
  - SIMPLE_CSV API: 3 errors (iteration, creation, validation)
  - SIMPLE_JSON API: 2 errors (iteration, method names)

Recommendations:
  1. Add simple_* API docs to class generation prompts
  2. Fix assemble path construction bug
  3. Add iteration pattern examples for each library
```

## Claude Discipline

When operating under simple_codegen:

1. **BEFORE** any action, call `simple_codegen log`
2. **ALWAYS** categorize honestly (don't hide gap-fills or rebellions)
3. **BE SPECIFIC** in action descriptions (reference prompt numbers, class names)

Example Claude workflow:
```
1. Read prompt 003_class_chart_config.txt
2. Log: simple_codegen log --session simple_chart --category INSTRUCTED --action "Reading prompt 003"
3. Generate CHART_CONFIG class
4. Log: simple_codegen log --session simple_chart --category INSTRUCTED --action "Generated CHART_CONFIG per prompt 003"
5. Notice prompt didn't specify default values
6. Log: simple_codegen log --session simple_chart --category GAP_FILL --action "Added default width=80 height=24 - not specified in prompt"
7. Create class with defaults
```

## Integration with SCG_AUDIT_DB

The action log complements SCG_AUDIT_DB:
- **SCG_AUDIT_DB**: Stores artifacts (prompts, responses, code)
- **SCG_ACTION_LOGGER**: Stores Claude's reasoning and decisions

Both are needed for complete visibility into the generation process.
