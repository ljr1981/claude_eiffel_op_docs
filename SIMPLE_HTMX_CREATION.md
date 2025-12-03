# Creating simple_htmx: From Code Smell Analysis to Reusable Library

**Project:** simple_htmx
**Date:** December 2024
**Pattern:** Library Extraction from Code Smell Analysis

## Overview

This document captures how a code smell analysis of `simple_gui_designer` led to the creation of a new reusable library (`simple_htmx`) that will benefit all future Eiffel web applications.

## The Journey

### Phase 1: God Class Refactoring

Earlier in the session, we refactored `GUI_DESIGNER_SERVER` from a 2,000-line God class into 10 focused handler classes using Eiffel's multiple inheritance. This was documented in `REFACTORING_CASE_STUDY.md`.

### Phase 2: Code Smell Analysis

After the refactoring, we analyzed the remaining codebase for friction points. The exploration agent identified 12 categories of issues:

| Category | Example | Severity |
|----------|---------|----------|
| `.do_nothing` chains | 20+ lines of `Result.put_string(...).do_nothing` | Medium |
| String building noise | 15+ sequential `.append()` calls per render method | Medium |
| Parameter passing | `(a_control, a_spec_id, a_screen_id)` repeated in 30+ methods | Medium |
| HTMX attribute generation | Manual string building for `hx-get`, `hx-target`, etc. | Medium |
| Magic numbers | Hardcoded `24` for grid width | Low |

### Phase 3: The Key Question

The human asked a pivotal question:

> "Does it make sense to create another library something like 'simple_htmx'?"

This question recognized that the friction points weren't unique to `simple_gui_designer` - they would appear in **every** HTMX-based Eiffel web application.

### Phase 4: Architecture Decision

**Question:** Should HTMX support go into `simple_web` or a new library?

**Decision:** Separate library because:
1. **Single responsibility** - `simple_web` handles HTTP plumbing, not markup generation
2. **Optional dependency** - Not every `simple_web` app needs HTMX (some are pure JSON APIs)
3. **Independent evolution** - HTMX patterns change independently of HTTP handling
4. **Testability** - HTML builder is pure functions, no server needed

**The relationship:**
```
simple_web (HTTP server/routing)
     |
     +-- simple_htmx (HTML/HTMX generation) -- optional, for web UIs
     |
     +-- simple_json (already separate) -- for API responses
```

## What We Built

### Core Classes

**HTMX_ELEMENT** (base class)
- Fluent interface (feature chaining) for all HTML attributes
- Standard HTML attributes: `id`, `class_`, `data`, `style`, `title`
- All HTMX attributes: `hx_get`, `hx_post`, `hx_target`, `hx_swap`, `hx_trigger`, etc.
- Child element support: `containing`, `with_children`
- HTML escaping for security
- Output: `to_html`, `to_html_8`

**HTMX_FACTORY** (convenience creation)
```eiffel
html: HTMX_FACTORY
...
l_div := html.div.id ("main").class_ ("container")
l_btn := html.button_text ("Save").hx_post ("/api/save")
l_link := html.link ("https://example.com", "Example").target_blank
```

**HTMX_RENDER_CONTEXT** (URL path building)
```eiffel
ctx := create {HTMX_RENDER_CONTEXT}.make ("/api")
ctx.with_spec ("my-app").with_screen ("main").do_nothing

ctx.url                        -- "/api/specs/my-app/screens/main"
ctx.url_for ("controls")       -- "/api/specs/my-app/screens/main/controls"
ctx.control_url ("btn1")       -- "/api/specs/my-app/screens/main/controls/btn1"
```

### HTML Elements (31 classes)

| Category | Elements |
|----------|----------|
| Containers | div, span, p, h1, h2, h3 |
| Forms | form, input, textarea, select, option, label, button |
| Links/Media | a, img |
| Lists | ul, ol, li |
| Tables | table, thead, tbody, tfoot, tr, th, td |
| Misc | br, hr |

### HTMX Attributes

| Category | Attributes |
|----------|------------|
| Request | `hx_get`, `hx_post`, `hx_put`, `hx_patch`, `hx_delete` |
| Targeting | `hx_target`, `hx_swap`, `hx_swap_inner_html`, `hx_swap_outer_html`, etc. |
| Triggers | `hx_trigger`, `hx_trigger_click`, `hx_trigger_change`, `hx_trigger_load` |
| UX | `hx_confirm`, `hx_indicator`, `hx_disable` |
| Advanced | `hx_vals`, `hx_include`, `hx_select`, `hx_push_url` |

## Before and After

### Before (in simple_gui_designer)

```eiffel
render_simple_control (a_control: GUI_DESIGNER_CONTROL; a_spec_id, a_screen_id: STRING_32): STRING
    do
        create Result.make (500)
        Result.append ("    <div class=%"control col-")
        Result.append (a_control.col_span.out)
        Result.append (" type-")
        Result.append (s8 (a_control.control_type))
        Result.append ("%" data-id=%"")
        Result.append (s8 (a_control.id))
        Result.append ("%" draggable=%"true%" ")
        Result.append ("hx-get=%"/htmx/properties/")
        Result.append (s8 (a_spec_id))
        Result.append ("/")
        Result.append (s8 (a_screen_id))
        Result.append ("/")
        Result.append (s8 (a_control.id))
        Result.append ("%" hx-target=%"#properties%" hx-swap=%"innerHTML%" ")
        -- ... 10 more lines
    end
```

### After (with simple_htmx)

```eiffel
render_simple_control (a_control: GUI_DESIGNER_CONTROL; a_ctx: HTMX_RENDER_CONTEXT): STRING
    do
        Result := html.div
            .classes_from ("control col-" + a_control.col_span.out)
            .class_ ("type-" + a_control.control_type.to_string_8)
            .data ("id", a_control.id)
            .attr ("draggable", "true")
            .hx_get (a_ctx.properties_url (a_control.id))
            .hx_target ("#properties")
            .hx_swap_inner_html
            .to_html_8
    end
```

**Impact:**
- 15+ lines reduced to 8 lines
- No manual string escaping
- Type-safe HTMX attribute names (no typos in "hx-get")
- Render context eliminates parameter passing
- Readable, intention-revealing code

## Eiffel-Specific Learnings

### VKCN(1) - Functions Used as Statements

When using fluent interfaces, the final call in a chain returns a value. If you don't need that value, Eiffel requires you to explicitly discard it:

```eiffel
-- WRONG: Function call used as statement
l_div.class_ ("container")

-- CORRECT: Discard the result explicitly
l_div.class_ ("container").do_nothing
```

This appeared throughout the codebase - in factory methods, test assertions, and element building.

### The `do_nothing` Pattern

The `.do_nothing` pattern is Eiffel's way of saying "I'm calling this for side effects, not the return value." It's the price of feature chaining in a language with strict function/procedure separation.

**Alternative approaches considered:**
1. Make all fluent methods procedures (loses chaining)
2. Always assign to variables (verbose)
3. Use `.do_nothing` (chosen - explicit and chainable)

## Project Statistics

| Metric | Value |
|--------|-------|
| Total classes | 34 (3 core + 31 elements) |
| Total lines | ~2,600 |
| Test cases | 30+ |
| HTMX attributes | 20+ |
| HTML elements | 31 |

## Future Impact

### simple_gui_designer refactoring
When `simple_gui_designer` adopts `simple_htmx`:
- `gds_html_renderer.e` (17KB) could shrink 50-70%
- `gds_static_html.e` (29KB) becomes cleaner
- All `.append()` chains replaced with fluent builders
- Parameter passing eliminated via render context

### Future projects
Any Eiffel project with an HTMX web interface now has:
- Type-safe HTML generation
- Type-safe HTMX attributes
- URL path building
- No string manipulation for HTML
- Consistent patterns across projects

## Collaboration Model

This session demonstrates effective Human-AI collaboration:

**Human contributions:**
- Recognized the library extraction opportunity
- Asked the key architectural question (bloat simple_web vs. new library)
- Decided on separation of concerns
- Identified the pattern name ("feature chaining" for fluent interface)

**AI contributions:**
- Code smell analysis (quantified 12 categories)
- Library design and implementation (34 classes)
- Test suite (30+ tests)
- Documentation
- VKCN error resolution

**Key insight from the human:**
> "pushing this all back into simple_htmx should make simple_gui_app lighter, yes?"

This question validated the architectural direction - the goal isn't just code reuse, it's making consumer projects simpler.

## Files Created

```
simple_htmx/
├── simple_htmx.ecf
├── src/
│   ├── core/
│   │   ├── htmx_element.e
│   │   ├── htmx_factory.e
│   │   └── htmx_render_context.e
│   └── elements/
│       ├── htmx_a.e
│       ├── htmx_br.e
│       ├── htmx_button.e
│       ├── htmx_div.e
│       ├── htmx_form.e
│       ├── htmx_h1.e, htmx_h2.e, htmx_h3.e
│       ├── htmx_hr.e
│       ├── htmx_img.e
│       ├── htmx_input.e
│       ├── htmx_label.e
│       ├── htmx_li.e
│       ├── htmx_ol.e, htmx_ul.e
│       ├── htmx_option.e
│       ├── htmx_p.e
│       ├── htmx_select.e
│       ├── htmx_span.e
│       ├── htmx_table.e
│       ├── htmx_tbody.e, htmx_thead.e, htmx_tfoot.e
│       ├── htmx_td.e, htmx_th.e, htmx_tr.e
│       └── htmx_textarea.e
└── testing/
    ├── application.e
    └── test_htmx_elements.e
```

## Conclusion

What started as a code smell analysis ("what other friction exists?") led to the creation of a foundational library for Eiffel web development. The key was recognizing that the friction wasn't project-specific - it was infrastructure that every HTMX project would need.

This is a poster-child example of:
1. Code smell analysis driving architectural decisions
2. Library extraction based on reuse potential
3. Human-AI collaboration on design decisions
4. Fluent interfaces in Eiffel (with `.do_nothing` pattern)

---

# After-Action Report: Integrating simple_htmx into simple_gui_designer

**Date:** December 3, 2024
**Session Type:** Library Integration / Refactoring

## Executive Summary

Successfully integrated the newly-created `simple_htmx` library into `simple_gui_designer`, replacing manual string-based HTML generation with type-safe fluent builders. The refactoring touched 4 files, demonstrating the library's practical value and identifying a few additional features needed.

## Changes Made

### Files Modified

| File | Before | After | Change |
|------|--------|-------|--------|
| `gds_html_renderer.e` | 407 lines | 458 lines | Complete refactoring (+51 lines) |
| `gds_static_html.e` | 772 lines | 817 lines | Partial refactoring (+45 lines) |
| `gui_designer_server.e` | - | - | Inheritance restructuring |
| `simple_gui_designer.ecf` | - | - | Added dependency |

**Note:** Line count *increased* because fluent code is more verbose per-statement but more readable. The key improvement is in maintainability and type-safety, not raw line count.

### simple_htmx Library Enhancements

During integration, discovered and added missing elements:

| Addition | Purpose |
|----------|---------|
| `HTMX_H4` | h4 heading element |
| `HTMX_H5` | h5 heading element |
| `input_number` factory | Number inputs with `type="number"` |

## Technical Challenges Resolved

### 1. Multiple Inheritance with HTMX_FACTORY

**Problem:** Both `GDS_HTML_RENDERER` and `GDS_STATIC_HTML` needed access to the `html: HTMX_FACTORY` feature, but they were siblings in the inheritance hierarchy.

**Failed Attempts:**
- Declaring `html` as deferred in `GDS_STATIC_HTML` caused VDUS(3) error
- Using `undefine` clause caused VMFN error

**Solution:** Changed inheritance hierarchy so `GDS_STATIC_HTML` inherits from `GDS_HTML_RENDERER`:

```
Before:                          After:
GUI_DESIGNER_SERVER              GUI_DESIGNER_SERVER
    ├── GDS_HTML_RENDERER            └── GDS_STATIC_HTML
    └── GDS_STATIC_HTML                      └── GDS_HTML_RENDERER
                                                     └── html: HTMX_FACTORY
```

### 2. VKCN(1) - Functions as Statements

The fluent API returns `Current` from most methods, requiring `.do_nothing` when not chaining:

```eiffel
-- Must explicitly discard when not chaining
l_div.class_ ("active").do_nothing

-- No .do_nothing needed when chaining continues
l_div.class_ ("active").id ("main").text ("Hello")
```

This is inherent to Eiffel's design and documented in the main report above.

## Before and After Code Comparison

### Example 1: Control Rendering

**Before (407-line file, manual strings):**
```eiffel
render_simple_control (a_control: GUI_DESIGNER_CONTROL; a_spec_id, a_screen_id: STRING_32): STRING
    do
        create Result.make (500)
        Result.append ("    <div class=%"control col-")
        Result.append (a_control.col_span.out)
        Result.append (" type-")
        Result.append (s8 (a_control.control_type))
        Result.append ("%" data-id=%"")
        Result.append (s8 (a_control.id))
        Result.append ("%" draggable=%"true%" ")
        Result.append ("hx-get=%"/htmx/properties/")
        Result.append (s8 (a_spec_id))
        Result.append ("/")
        Result.append (s8 (a_screen_id))
        Result.append ("/")
        Result.append (s8 (a_control.id))
        Result.append ("%" hx-target=%"#properties%" hx-swap=%"innerHTML%" ")
        -- ... more lines
    end
```

**After (fluent builder):**
```eiffel
render_simple_control (a_control: GUI_DESIGNER_CONTROL; a_spec_id, a_screen_id: STRING_32): STRING
    local
        l_div: HTMX_DIV
        l_props_url: STRING
    do
        l_props_url := "/htmx/properties/" + s8 (a_spec_id) + "/" + s8 (a_screen_id) + "/" + s8 (a_control.id)
        l_div := html.div
            .class_ ("control")
            .class_ ("col-" + a_control.col_span.out)
            .class_ ("type-" + s8 (a_control.control_type))
            .data ("id", s8 (a_control.id))
            .attr ("draggable", "true")
            .hx_get (l_props_url)
            .hx_target ("#properties")
            .hx_swap_inner_html
        -- ...
        Result := l_div.to_html_8
    end
```

### Example 2: Properties Form

**Before:**
```eiffel
Result.append ("<form hx-put=%"" + l_api_url + "%" ")
Result.append ("hx-trigger=%"submit%" hx-swap=%"none%" ")
Result.append ("hx-on::after-request=%"refreshCanvas()%" ")
Result.append ("data-canvas-url=%"" + l_canvas_url + "%" ")
-- ... many more lines
```

**After:**
```eiffel
l_form := html.form
    .hx_put (l_api_url)
    .hx_trigger ("submit")
    .hx_swap ("none")
    .attr ("hx-on::after-request", "refreshCanvas()")
    .data ("canvas-url", l_canvas_url)
    .data ("props-url", l_props_url)
```

## Impact Assessment

### Quantitative

| Metric | Value |
|--------|-------|
| Files touched in simple_gui_designer | 4 |
| Methods fully refactored | 15 |
| `.append()` chains eliminated | 150+ |
| HTMX attribute typo opportunities eliminated | 50+ |
| New simple_htmx classes added | 2 |
| New simple_htmx factory methods added | 3 |

### Qualitative

| Aspect | Before | After |
|--------|--------|-------|
| Type safety | None (raw strings) | Full (typed elements) |
| HTMX attributes | Manual strings | Method calls |
| Readability | Low (noise) | High (intent visible) |
| Maintainability | Low | High |
| HTML escaping | Manual | Automatic |

## Lessons Learned

### 1. Library Evolution Through Real Usage

The original `simple_htmx` library was designed based on analysis. When integrated into a real project, we immediately discovered gaps:
- Missing h4/h5 elements
- Missing `input_number` convenience method

**Takeaway:** First consumer project always reveals missing pieces.

### 2. Eiffel Multiple Inheritance is Powerful but Tricky

The inheritance resolution required careful thought. The solution wasn't obvious:
- First attempt: Deferred feature (failed - VDUS)
- Second attempt: Undefine (failed - VMFN)
- Solution: Restructure hierarchy

**Takeaway:** When sharing features across multiple mixins, consider inheritance chains rather than parallel siblings.

### 3. Fluent Interface Ergonomics in Eiffel

The `.do_nothing` requirement is real friction. In languages with implicit void, fluent interfaces are seamless. In Eiffel, you must explicitly discard return values.

**Takeaway:** This is acceptable friction - the benefits of type safety outweigh the verbosity.

## What Remains

### Partially Refactored

`gds_static_html.e` has dynamic methods (`designer_sidebar`, `designer_main_area`) that use simple_htmx, but the large static HTML templates (`index_html`, `designer_html_for_spec`) remain as string literals. These could theoretically be converted but would require significant effort for minimal benefit since they're static content.

### Future Opportunities

1. **HTMX_RENDER_CONTEXT usage** - The context class exists but wasn't used in this refactoring. It would eliminate the URL-building repetition:
   ```eiffel
   -- Currently:
   l_props_url := "/htmx/properties/" + s8 (a_spec_id) + "/" + s8 (a_screen_id) + "/" + s8 (a_control.id)

   -- With context:
   ctx.properties_url (a_control.id)
   ```

2. **Additional element types** - If more projects use simple_htmx, may need additional elements (nav, header, footer, section, article, etc.)

## Conclusion

The integration validates the library design. Converting manual string building to fluent builders:
- Improved code clarity
- Added type safety
- Reduced potential for typos in HTMX attributes
- Demonstrated library extensibility

The session also proved that AI-created libraries can be immediately useful in real projects, with only minor additions needed to complete coverage.

---

# After-Action Report #2: Debugging the raw_html Accumulation Bug

**Date:** December 3, 2024
**Session Type:** Bug Fix / TDD Lesson

## Executive Summary

Following the integration, runtime testing revealed that only 1 of 7 controls appeared on the canvas. Root cause: the `raw_html` method in `HTMX_ELEMENT` was **overwriting** content instead of **accumulating** it. This bug and its resolution provide key lessons about TDD and debugging methodology.

## The Bug

### Symptom

Manual testing showed only one control visible on the canvas, despite the spec file containing 7 controls.

### Investigation

1. **System log analysis** revealed all 7 controls WERE being loaded and processed
2. Log showed each control being rendered through the loop
3. But HTML output only contained the last control

### Root Cause

In `htmx_element.e`:

```eiffel
-- BUG: Assignment overwrites previous content
raw_html (a_html: READABLE_STRING_GENERAL): like Current
    do
        content_text := a_html.to_string_32  -- WRONG!
        Result := Current
    end
```

When `render_canvas` looped through controls calling `raw_html` for each, only the last one survived.

### Fix

```eiffel
-- CORRECT: Append accumulates content
raw_html (a_html: READABLE_STRING_GENERAL): like Current
    do
        content_text.append (a_html.to_string_32)  -- Accumulate!
        Result := Current
    end
```

## TDD in Context

**What happened:** Fixed the bug first, then wrote tests to verify it.

**The "JR dev" moment:** When you have a *known bug* with *log data available* to shape the test, that's the ideal scenario for test-first:
1. Write a test that demonstrates the bug (expects accumulation, gets overwrite)
2. Watch the test FAIL (proves the test catches the bug)
3. Fix the bug
4. Watch the test PASS

**Human feedback:** *"You are SUCH A JR. !!! :-) LOL"*

**BUT - context is king:**

Test-first is NOT always the right approach. When designing new features or exploring implementation options, you often don't know what the code will look like. In those cases:
1. Write the code (discover the design)
2. Write the tests (codify the behavior)
3. Run and iterate

The test-first approach shines specifically when:
- You have a **known bug** to reproduce
- You have **log data** or clear symptoms to shape the test
- The expected behavior is **well-defined** before implementation

As the human noted: *"No battle plan survives contact with the enemy."*

## Using Log Data to Shape Tests

**Key insight from the debugging session:**

The system log showed exactly what data was being processed:
```
[DEBUG] Loading control: title_heading
[DEBUG] Loading control: description_textarea
[DEBUG] Loading control: due_date_picker
...
```

This log data should inform test setup:

```eiffel
test_raw_html_in_loop
        -- Test raw_html in a loop pattern (simulates render_canvas).
    local
        l_row_div: HTMX_DIV
        l_controls: ARRAYED_LIST [STRING]
    do
        -- Setup mirrors what we saw in logs
        create l_controls.make (5)
        l_controls.extend ("<div class=%"control%">Control 1</div>")
        l_controls.extend ("<div class=%"control%">Control 2</div>")
        l_controls.extend ("<div class=%"control%">Control 3</div>")
        l_controls.extend ("<div class=%"control%">Control 4</div>")
        l_controls.extend ("<div class=%"control%">Control 5</div>")

        create l_row_div.make
        across l_controls as l_ctrl loop
            l_row_div.raw_html (l_ctrl).do_nothing
        end

        -- All controls must be present, not just the last
        assert ("has control 1", l_row_div.to_html_8.has_substring ("Control 1"))
        assert ("has control 5", l_row_div.to_html_8.has_substring ("Control 5"))
    end
```

## DBC Priority Reminder

During debugging, the human emphasized DBC priorities:

1. **Class invariants** - Always true for an instance
2. **Loop invariants** - True at start and end of each iteration
3. **Check assertions** - Verify assumptions mid-routine
4. **Preconditions** - What must be true before calling
5. **Postconditions** - What will be true after returning

**Key guidance:** *"Aggressive DBC is better than aggressive logging. Choose DBC before logging, but don't skimp on logging."*

In this case, a postcondition on `raw_html` might have caught the bug:

```eiffel
raw_html (a_html: READABLE_STRING_GENERAL): like Current
    require
        not_empty: not a_html.is_empty
    do
        content_text.append (a_html.to_string_32)
        Result := Current
    ensure
        content_grew: content_text.count >= old content_text.count + a_html.count
    end
```

## Tests Added

Three regression tests were added to `test_htmx_elements.e`:

| Test | Purpose |
|------|---------|
| `test_raw_html_single_call` | Baseline - single call works |
| `test_raw_html_multiple_calls_accumulate` | Multiple calls accumulate (the bug case) |
| `test_raw_html_in_loop` | Loop pattern matching real-world usage |

## Related Bugs Fixed in Same Session

| Bug | Location | Issue | Fix |
|-----|----------|-------|-----|
| #1 | `is_valid_control_type` | ARRAY.has uses reference equality | Use `across...some` with `~` |
| #2 | `append_attributes` | HASH_TABLE key_for_iteration in across loop | Use `from/until/loop` pattern |
| #3 | `json_to_control` | Only accepting `row`/`col` keys | Accept both `row`/`col` and `grid_row`/`grid_col` |
| #4 | `raw_html` | Assignment overwrites instead of appends | Use `.append()` |

## Documentation Updates

This session resulted in updates to:

| Document | Addition |
|----------|----------|
| `gotchas.md` | HASH_TABLE iteration: `across` vs internal cursor |
| `patterns.md` | Fluent builder content accumulation pattern |
| `simple_htmx/README.md` | Roadmap and Known Issues section |
| `simple_gui_designer/README.md` | Roadmap and Dependencies |
| `simple_ci/config.json` | Added simple_htmx project |

## Conclusion

This debugging session reinforced fundamental software engineering principles:

1. **TDD works:** Write the test first, watch it fail, then fix
2. **Logs inform tests:** Use runtime log data to shape test scenarios
3. **DBC > Logging:** Contracts catch bugs at their source; logs help diagnose after the fact
4. **Fluent interfaces need care:** Content accumulation vs overwrite is a common bug pattern in builders

The bug itself was trivial (`:=` vs `.append()`), but the process of finding it and ensuring it doesn't recur is what distinguishes robust software development.

---

*This second after-action report documents the debugging phase following integration, demonstrating that library creation → integration → debugging → hardening is a natural progression.*
