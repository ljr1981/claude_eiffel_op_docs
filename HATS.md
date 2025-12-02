# Working Hats

**Purpose**: Define focused work modes ("hats") for specific tasks. When you want Claude to focus on a particular type of work, tell Claude which hat to wear.

---

## How to Use

Simply say: **"Put on your [hat name] hat"** or **"Let's do some [hat name] work"**

When wearing a hat, Claude will:
1. Focus exclusively on that type of work
2. Apply the specific principles and checks for that hat
3. Avoid scope creep into other areas
4. Use the relevant checklists and patterns

---

## Available Hats

### Contracting Hat (Design-by-Contract)

**Focus**: Adding, reviewing, and strengthening contracts (preconditions, postconditions, invariants)

**When to use**:
- Hardening existing code with proper contracts
- Reviewing contract coverage
- Finding missing or weak contracts
- Ensuring defensive programming

**Principles**:
- Every public feature should have appropriate contracts
- Preconditions define what callers must guarantee
- Postconditions define what the feature guarantees
- Class invariants define what's always true about objects

**Priority Order** (work through in this sequence):
1. **Class invariants** (`invariant`) - What's always true about objects of this class?
2. **Loop invariants** (`invariant`) and **variants** - Loop correctness and termination
3. **Check assertions** (`check`) - Internal sanity checks within routines
4. **Preconditions/Postconditions** (`require`/`ensure`, `require else`/`ensure then`)

**Checklist**:
- [ ] Class invariants - fundamental truths about the class
- [ ] Loop invariants and variants for all loops
- [ ] Check assertions for non-obvious internal states
- [ ] Preconditions - what callers must guarantee
- [ ] Postconditions - what the feature guarantees (use `old` where needed)

**Avoid**: Adding features, refactoring, changing logic - only add/improve contracts

---

### Refactoring Hat

**Focus**: Improving code structure without changing behavior

**When to use**:
- Cleaning up code smells
- Extracting methods or classes
- Improving naming
- Reducing duplication
- Simplifying complex logic

**Principles**:
- Behavior must remain identical
- Tests should pass before and after
- Small, incremental changes
- Each refactoring should be independently valid

**Checklist**:
- [ ] Tests pass before starting
- [ ] Identify specific smell/issue to address
- [ ] Make minimal change to fix it
- [ ] Tests still pass
- [ ] Repeat

**Avoid**: Adding features, fixing bugs, changing contracts (unless they're wrong)

---

### Testing Hat

**Focus**: Writing and improving tests

**When to use**:
- Adding test coverage
- Writing tests for new features
- Improving test quality
- Finding edge cases

**Principles**:
- Test behavior, not implementation
- Cover happy path, edge cases, and error conditions
- Tests should be independent and repeatable
- Clear test names describing what's being tested

**Checklist**:
- [ ] Happy path covered
- [ ] Edge cases (empty, null, boundary values)
- [ ] Error conditions
- [ ] Contract violations (if applicable)
- [ ] Test names are descriptive

**Avoid**: Fixing production code (note issues for later), refactoring production code

---

### Mock Building Hat

**Focus**: Creating mock consumers/applications that exercise the library

**When to use**:
- Validating library API usability
- Stress-testing library features in realistic scenarios
- Discovering edge cases through real-world usage patterns
- Demonstrating library capabilities

**Principles**:
- Mocks should represent realistic use cases
- Exercise the library's public API as a real consumer would
- Bugs found belong to the library, not the mock (mocks are disposable)
- Mocks can be messy - they're tools for discovery, not production code
- Document discoveries (bugs, awkward APIs, missing features) for library improvement

**Checklist**:
- [ ] Define the mock's purpose/scenario
- [ ] Use only public library API (no reaching into internals)
- [ ] Exercise relevant library features
- [ ] Note any friction or issues discovered
- [ ] Feed findings back to library backlog

**Avoid**: Over-engineering the mock, treating mock bugs as important (fix the library instead)

---

### Documentation Hat

**Focus**: Writing and improving documentation

**When to use**:
- Documenting APIs
- Writing README content
- Adding code comments for complex logic
- Creating usage examples

**Principles**:
- Document "why" not "what" (code shows what)
- Keep docs close to code
- Examples are worth 1000 words
- Update docs when code changes

**Checklist**:
- [ ] Class-level note clause with description
- [ ] Complex features have explanatory comments
- [ ] Public API is documented
- [ ] Examples provided where helpful
- [ ] EIS links where appropriate

**Avoid**: Changing code behavior, refactoring

---

### Performance Hat

**Focus**: Optimizing performance

**When to use**:
- Addressing known performance issues
- Optimizing hot paths
- Reducing memory usage
- Improving query efficiency

**Principles**:
- Measure before optimizing
- Focus on actual bottlenecks
- Don't sacrifice clarity without good reason
- Document performance-critical code

**Checklist**:
- [ ] Identify actual bottleneck (not assumed)
- [ ] Measure current performance
- [ ] Implement optimization
- [ ] Measure improvement
- [ ] Document if code is less clear

**Avoid**: Premature optimization, refactoring for style

---

### Code Review Hat

**Focus**: Reviewing code for issues and improvements

**When to use**:
- Reviewing changes before commit
- Auditing existing code quality
- Looking for potential issues

**Principles**:
- Check for correctness first
- Look for potential bugs
- Consider edge cases
- Note style issues separately from bugs

**Checklist**:
- [ ] Logic is correct
- [ ] Edge cases handled
- [ ] Contracts are appropriate
- [ ] No obvious bugs
- [ ] Code is readable
- [ ] No security issues

**Output**: List of findings categorized by severity

---

### Feature Hat

**Focus**: Implementing new functionality

**When to use**:
- Adding new features
- Extending existing functionality
- Implementing user stories

**Principles**:
- Understand requirements before coding
- Start simple, iterate
- Include tests with the feature
- Follow existing patterns

**Checklist**:
- [ ] Requirements understood
- [ ] Design fits existing architecture
- [ ] Implementation complete
- [ ] Tests written
- [ ] Contracts added
- [ ] Documentation updated (if public API)

---

### Profiler Hat

**Focus**: Performance profiling and identifying bottlenecks

**When to use**:
- Investigating performance problems
- Identifying hot spots before optimization
- Validating that optimizations worked
- Understanding execution flow and call patterns

**Principles**:
- Profile before optimizing (never guess)
- Focus on biggest bottlenecks first (Pareto principle)
- Measure improvement with data, not intuition
- Finalized builds give more accurate timing than workbench

**Recommended Setup**: Create a dedicated `<app>_profile` target with:
- `profile="true"` on `<option>`
- A profile harness app (exercises code paths, non-interactive)
- Inlining disabled: `<setting name="inlining" value="false"/>`
- See `profiler.md` for detailed pattern and `wms_profile_app.e` for example

**Workflow** (requires EiffelStudio GUI for full analysis):
1. Create dedicated profile target with harness app
2. Finalize compile (frozen mode may not generate profinfo via batch)
3. Run the instrumented executable (generates `profinfo_XX` in F_CODE)
4. Use EiffelStudio Profiler Wizard to convert `profinfo` to `.pfi`
5. Analyze: calls, self-time, descendant-time, percentages
6. Identify bottlenecks and apply Performance Hat for optimization
7. Re-profile to verify improvement

**Output Metrics**:
- **Calls**: Number of times each feature was invoked
- **Self-time**: Time spent in the feature itself (excluding callees)
- **Descendant-time**: Time spent in called features
- **Percentage**: Proportion of total execution time

**Files Generated**:
- `profinfo_XX` - Raw profiling data (binary, suffix varies: _AC, _C4, etc.)
- `profinfo.pfi` - Converted execution profile for analysis
- Located in: `EIFGENs/<target>/F_CODE/` (finalized)

**Limitations**:
- Full profiler analysis requires EiffelStudio GUI (Profiler Wizard)
- AutoTest does NOT generate useful profinfo - use dedicated harness app
- **Shallow call tree issue (ES 25.02)**: Profiler may only show root class features, not called classes - under investigation
- Profiling adds overhead; use finalized builds for accurate relative timing
- **Segfaults on startup**: Usually indicates corrupted EIFGENs - do a clean compile (`-clean` flag)

**Checklist**:
- [ ] Profiling enabled in ECF (`profile="true"` on `<option>` element)
- [ ] System recompiled with profiling
- [ ] Representative workload executed
- [ ] `profinfo` file generated
- [ ] Profile converted and analyzed (in EiffelStudio)
- [ ] Top bottlenecks identified with percentages
- [ ] Optimization targets prioritized

**Avoid**: Optimizing without profiling data, micro-optimizations before fixing macro issues

**Note**: This hat often transitions to the **Performance Hat** or **SCOOP Hat** once bottlenecks are identified.

---

### SCOOP Hat (Concurrency)

**Focus**: Identifying and implementing concurrency opportunities using SCOOP

**When to use**:
- After profiling reveals CPU-bound bottlenecks that could parallelize
- When operations are independent and could run concurrently
- Examining I/O-bound code that could benefit from async patterns
- Reviewing existing code for concurrency opportunities

**Principles**:
- Concurrency adds complexity - only use where benefit is clear
- Profile first to identify actual bottlenecks (use Profiler Hat)
- SCOOP makes concurrency safer but not free
- Separate objects communicate asynchronously by default
- Queries on separate objects are synchronous (wait for result)

**SCOOP Key Concepts**:
- `separate` keyword declares objects that may run on different processors
- Locking via routine arguments (separate objects passed as args are locked)
- Commands are asynchronous, queries are synchronous
- Wait conditions via preconditions on separate arguments

**Good Candidates for SCOOP**:
- Independent batch operations (processing multiple items)
- I/O operations (file, network, database) that can overlap
- Background tasks (logging, monitoring, cleanup)
- Producer-consumer patterns
- Parallel computations on independent data

**Poor Candidates**:
- Tightly coupled operations with frequent synchronization
- Operations with shared mutable state
- Very short operations (overhead exceeds benefit)
- Sequential algorithms with dependencies

**Checklist**:
- [ ] Profile data shows parallelizable bottleneck
- [ ] Operations are sufficiently independent
- [ ] Identify which objects should be `separate`
- [ ] Design locking strategy (routine arguments)
- [ ] Consider wait conditions (preconditions)
- [ ] Test for correctness under concurrency
- [ ] Re-profile to verify improvement

**Workflow** (often follows Profiler Hat):
1. Review profiler results for CPU-bound hot spots
2. Analyze data dependencies - what can run in parallel?
3. Identify natural boundaries for separate processors
4. Design the SCOOP architecture
5. Implement with `separate` objects
6. Test thoroughly (concurrency bugs are subtle)
7. Re-profile to measure improvement

**Avoid**: Adding concurrency without profiling data, over-parallelizing, ignoring synchronization costs

**Reference**: See `scoop.md` for SCOOP patterns and gotchas

---

### Cleanup Hat

**Focus**: General code hygiene and maintenance

**When to use**:
- Removing dead code
- Fixing warnings
- Updating obsolete patterns
- General tidying

**Principles**:
- Remove rather than comment out
- Fix warnings properly
- Update to modern idioms
- Keep changes focused

**Checklist**:
- [ ] Dead code removed
- [ ] Warnings addressed
- [ ] Obsolete patterns updated
- [ ] Code compiles clean

---

## Combining Hats

Sometimes you need multiple hats in sequence:

**Example workflow for adding a feature**:
1. **Feature Hat** - Implement the functionality
2. **Testing Hat** - Add comprehensive tests
3. **Contracting Hat** - Add/verify contracts
4. **Documentation Hat** - Document if public API

**Example workflow for library hardening** (mature library):
1. **Mock Building Hat** - Create realistic consumer to exercise API
2. **Testing Hat** - Add tests for any gaps discovered
3. **Contracting Hat** - Strengthen contracts based on findings

**Example workflow for reported bug**:
1. **Testing Hat** - Write failing test that reproduces the bug
2. **Feature Hat** - Fix the issue (minimal change)
3. **Contracting Hat** - Add contract to prevent recurrence

**Example workflow for performance optimization**:
1. **Profiler Hat** - Identify bottlenecks with data
2. **SCOOP Hat** - Evaluate if concurrency would help (for CPU-bound parallel work)
3. **Performance Hat** - Apply optimizations (algorithmic or concurrent)
4. **Profiler Hat** - Verify improvement with new profile

---

## Custom Hats

If you need a focused work mode not listed here, just describe it:

*"Let's put on a security review hat - I want you to look at this code specifically for security vulnerabilities"*

Claude will adapt the focused approach to your specific need.

---

## Update Log

| Date | Change |
|------|--------|
| 2025-12-02 | Profiler Hat: Segfaults usually mean corrupted EIFGENs - use clean compile |
| 2025-12-02 | Profiler Hat: Corrected ECF syntax (`profile="true"` on `<option>`, not `<setting>`) |
| 2025-12-02 | Added SCOOP hat for concurrency analysis and implementation |
| 2025-12-02 | Added Profiler hat for performance profiling work |
| 2025-12-02 | Removed Bug Hunting hat (covered by Testing + Contracting), added Mock Building hat |
| 2025-12-02 | Initial hats documentation created |
