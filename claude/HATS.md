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

### Specification Hat (Vibe-Contracting)

**Focus**: Writing formal specifications (contracts) BEFORE implementation

**When to use**:
- Starting a new feature (write contracts first!)
- Formalizing requirements into contracts
- Defining interfaces between components
- When you want to think about WHAT before HOW

**Principles** (from Meyer's "probable to provable"):
- Specification IS requirements engineering
- Contracts define behavior precisely, enabling verification
- Write contracts first, then implement to satisfy them
- "Vibe-contracting complements vibe-coding"

**The Key Questions** (ask in order):
1. **What must be true BEFORE this can be called?** → Preconditions
2. **What will be true AFTER this completes?** → Postconditions
3. **What is ALWAYS true about this class?** → Class invariants
4. **What ELSE is guaranteed?** → Completeness check (critical!)

**Contract Completeness Check**:
After writing a postcondition, ask: "Is this TRUE but INCOMPLETE?"

```eiffel
-- INCOMPLETE: Only says item exists
ensure
  has_item: items.has (a_item)

-- COMPLETE: Says item added and count changed
ensure
  has_item: items.has (a_item)
  count_increased: items.count = old items.count + 1
```

**Workflow**:
1. Write feature signature (name, parameters, return type)
2. Write `require` clause - what callers must guarantee
3. Write `ensure` clause - what this feature guarantees
4. Review: Is postcondition COMPLETE, not just true?
5. Only THEN write implementation (or hand to Feature Hat)

**Checklist**:
- [ ] Preconditions capture all caller obligations
- [ ] Postconditions capture ALL guarantees (completeness!)
- [ ] Return value postconditions use `Result`
- [ ] State changes use `old` for comparison
- [ ] Class invariant updated if new state introduced

**Avoid**: Writing implementation before contracts, assuming contracts are "obvious"

**Reference**: See `verification_process.md` for full iterative workflow

---

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

**Focus**: Deep-dive code review with full codebase access and web research

**When to use**:
- Comprehensive code audit before release
- Security review of user-facing features
- Reviewing changes before commit
- Auditing existing code quality
- Comparing implementation against best practices

**Capabilities** (Claude has access to):
- **Full codebase**: Read all source files, tests, configs
- **Web research**: Search for best practices, OWASP guidelines, library docs
- **Cross-reference**: Compare against similar implementations online
- **Dependency analysis**: Check for known vulnerabilities in patterns used

**Principles**:
- Check for correctness first
- Look for potential bugs and security issues
- Consider edge cases and error handling
- Research best practices for the domain (web security, database, etc.)
- Note style issues separately from bugs
- Provide actionable recommendations with severity

**Review Workflow**:
1. **Scope**: Identify files/classes to review
2. **Read**: Deep-dive into the code
3. **Research**: Web search for relevant best practices, vulnerabilities, patterns
4. **Analyze**: Compare code against findings
5. **Report**: Categorized findings with severity and recommendations

**Checklist - Correctness**:
- [ ] Logic is correct
- [ ] Edge cases handled (empty, null, boundary values)
- [ ] Error conditions handled appropriately
- [ ] Contracts (preconditions/postconditions) are appropriate
- [ ] Invariants maintained

**Checklist - Security** (research OWASP if needed):
- [ ] Input validation/sanitization
- [ ] No injection vulnerabilities (SQL, XSS, command)
- [ ] Authentication/authorization correct
- [ ] Sensitive data protected
- [ ] No hardcoded secrets

**Checklist - Eiffel-Specific**:
- [ ] Void safety (proper use of `attached`, `detachable`)
- [ ] Design by Contract coverage
- [ ] Proper use of `once` functions for singletons
- [ ] ARRAYED_LIST.has vs value equality (`~`)
- [ ] STRING_8 vs STRING_32 handling

**Checklist - Quality**:
- [ ] Code is readable and well-named
- [ ] No unnecessary complexity
- [ ] DRY (no duplicated logic)
- [ ] Follows existing patterns in codebase

**Severity Levels**:
- **Critical**: Security vulnerability, data loss risk, crash
- **High**: Bug that affects functionality, contract violation
- **Medium**: Edge case not handled, missing validation
- **Low**: Style issue, minor improvement, documentation gap

**Output Format**:
```
## Code Review: [Class/Feature Name]

### Critical
- [Issue description] - [File:Line] - [Recommendation]

### High
- [Issue description] - [File:Line] - [Recommendation]

### Medium
- [Issue description] - [File:Line] - [Recommendation]

### Low
- [Issue description] - [File:Line] - [Recommendation]

### Research Notes
- [Relevant best practices found]
- [Links to documentation/guidelines consulted]
```

**Avoid**: Making changes during review (note for later), scope creep beyond review

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

### Code Smell Detector Hat

**Focus**: Identifying code smells and patterns that indicate design problems

**When to use**:
- Before major refactoring to identify targets
- During code review to spot structural issues
- When code "feels wrong" but you can't articulate why
- Periodic health checks on mature codebases

**Common Code Smells to Look For**:

| Smell | Symptom | Eiffel-Specific Signs |
|-------|---------|----------------------|
| **God Class** | Class doing too much | >1000 lines, many unrelated feature groups |
| **Long Feature** | Feature doing too much | >50 lines, multiple levels of nesting |
| **Feature Envy** | Feature uses another class more than its own | Many calls to one external object |
| **Duplicate Code** | Same logic in multiple places | Copy-pasted code with minor variations |
| **Data Clumps** | Same group of data passed around | Multiple features with same parameter sets |
| **Primitive Obsession** | Using primitives instead of objects | Strings/integers where domain types fit |
| **Long Parameter List** | Too many parameters | >4 parameters, especially if same types |
| **Shotgun Surgery** | Change requires edits in many places | Adding feature requires touching >3 classes |
| **Divergent Change** | Class changed for unrelated reasons | Multiple unrelated modification reasons |
| **Inappropriate Intimacy** | Classes too coupled | Direct attribute access, friend-like patterns |
| **Message Chains** | a.b.c.d.e chains | Long navigation through objects |
| **Dead Code** | Unreachable/unused code | Features never called, commented-out code |
| **Speculative Generality** | Unused flexibility | Abstract classes with one implementation |

**Workflow**:
1. **Survey**: Scan class structure, file sizes, feature counts
2. **Sniff**: Read through looking for patterns above
3. **Document**: Note each smell with location and severity
4. **Prioritize**: Rank by impact (high coupling > style issues)
5. **Plan**: Create refactoring backlog (use Refactoring Hat later)

**Output Format**:
```
## Code Smell Report: [Area/Class]

### High Impact
- [Smell]: [Location] - [Evidence] - [Suggested Fix]

### Medium Impact
- [Smell]: [Location] - [Evidence] - [Suggested Fix]

### Low Impact (Style)
- [Smell]: [Location] - [Evidence] - [Suggested Fix]
```

**Avoid**: Fixing smells during detection (note for Refactoring Hat), over-reporting minor issues

---

### Security Hat

**Focus**: Identifying security vulnerabilities, attack vectors, and hardening code against threats

**When to use**:
- Before deploying user-facing features
- After adding any user input handling (forms, URLs, file uploads)
- Reviewing authentication/authorization code
- Handling sensitive data (credentials, PII, tokens)
- Adding external integrations (APIs, email, file system)
- Periodic security audits of existing code

**Mindset**:
- Assume ALL input is malicious until validated
- Think like an attacker - what would I exploit?
- Trust nothing from the client/user
- Defense in depth - multiple layers of protection
- Fail secure - deny by default

**OWASP Top 10 Checklist** (web applications):
- [ ] **Injection** (SQL, command, XSS, path traversal)
  - All user input sanitized/escaped for context
  - Parameterized queries for database
  - No shell command construction from user data
- [ ] **Broken Authentication**
  - Session management secure
  - Passwords hashed (not encrypted or plain)
  - Rate limiting on login attempts
- [ ] **Sensitive Data Exposure**
  - Credentials not logged or in error messages
  - Secrets not in source code
  - HTTPS for sensitive data in transit
- [ ] **Broken Access Control**
  - Authorization checked on every request
  - No direct object references without auth check
  - Principle of least privilege
- [ ] **Security Misconfiguration**
  - Debug mode disabled in production
  - Default credentials changed
  - Unnecessary features/ports disabled
- [ ] **Cross-Site Scripting (XSS)**
  - All output HTML-encoded
  - Content-Security-Policy headers
  - No inline JavaScript from user data
- [ ] **Insecure Deserialization**
  - No deserializing untrusted data
  - Validate object types after deserialization
- [ ] **Using Components with Known Vulnerabilities**
  - Dependencies up to date
  - No deprecated/abandoned libraries
- [ ] **Insufficient Logging/Monitoring**
  - Security events logged (login, auth failures)
  - No sensitive data in logs
  - Log injection prevented

**Eiffel-Specific Security**:
- [ ] Void safety enforced (no null pointer attacks)
- [ ] Contracts validate input at trust boundaries
- [ ] `require` clauses guard against invalid input
- [ ] File paths validated (no `..` traversal)
- [ ] External process execution sanitized

**Web Application Checklist**:
- [ ] CSRF tokens on state-changing operations
- [ ] Rate limiting on sensitive endpoints
- [ ] Input length limits enforced
- [ ] File upload type/size validation
- [ ] Cookie security flags (HttpOnly, Secure, SameSite)
- [ ] Response headers set (X-Frame-Options, CSP, etc.)

**Severity Levels**:
- **Critical**: Remote code execution, auth bypass, data breach risk
- **High**: XSS, SQL injection, privilege escalation
- **Medium**: Information disclosure, weak crypto, missing rate limits
- **Low**: Missing headers, verbose errors, minor hardening gaps

**Workflow**:
1. **Map attack surface**: User inputs, external interfaces, data flows
2. **Review OWASP**: Check each category against the code
3. **Trace data flow**: Follow user input from entry to storage/output
4. **Test boundaries**: What happens with malformed/malicious input?
5. **Report findings**: Categorize by severity with reproduction steps

**Output Format**:
```
## Security Review: [Component/Feature]

### Attack Surface
- [Entry points, user inputs, external interfaces]

### Critical
- [Vulnerability] - [File:Line] - [Exploit scenario] - [Fix]

### High
- [Vulnerability] - [File:Line] - [Exploit scenario] - [Fix]

### Medium/Low
- [Issue] - [Location] - [Recommendation]

### Hardening Recommendations
- [Defense-in-depth improvements]
```

**Avoid**: Fixing issues during review (note for later), assuming "users won't do that"

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
| 2025-12-04 | Added Security Hat for vulnerability assessment and security hardening |
| 2025-12-03 | Added Specification Hat (vibe-contracting) based on Meyer's "probable to provable" |
| 2025-12-03 | Added Code Smell Detector hat for identifying design problems |
| 2025-12-02 | Expanded Code Review Hat with web research, security checklists, severity levels |
| 2025-12-02 | Profiler Hat: Segfaults usually mean corrupted EIFGENs - use clean compile |
| 2025-12-02 | Profiler Hat: Corrected ECF syntax (`profile="true"` on `<option>`, not `<setting>`) |
| 2025-12-02 | Added SCOOP hat for concurrency analysis and implementation |
| 2025-12-02 | Added Profiler hat for performance profiling work |
| 2025-12-02 | Removed Bug Hunting hat (covered by Testing + Contracting), added Mock Building hat |
| 2025-12-02 | Initial hats documentation created |
