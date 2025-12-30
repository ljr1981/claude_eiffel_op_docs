# Project Plan: simple_dev – AI-Orchestrated Eiffel Development

## Table of Contents

- [Background: The Simple Eiffel Ecosystem and AI-Assisted Workflow](#background-the-simple-eiffel-ecosystem-and-ai-assisted-workflow)
- [Current Limitations of LLM-Assisted Coding](#current-limitations-of-llm-assisted-coding-lay-of-the-land)
- [Vision: A Guided AI Development Orchestrator](#vision-a-guided-ai-development-orchestrator-simple_dev)
- [Key Components and Features of simple_dev](#key-components-and-features-of-simple_dev)
- [Embracing Eiffel's Strengths: Design by Contract meets AI](#embracing-eiffels-strengths-design-by-contract-meets-ai)
- [Inspiration from Similar Projects and Industry Trends](#inspiration-from-similar-projects-and-industry-trends)
- [Conclusion](#conclusion)
- [Sources](#sources)
- [Citations](#citations)

---

## Background: The Simple Eiffel Ecosystem and AI-Assisted Workflow

![simple_ai_client](https://github.com/simple-eiffel/simple_ai_client)

*Figure: The Simple Eiffel initiative is building a suite of lightweight Eiffel libraries (each named `simple_*`) to modernize the Eiffel ecosystem. These libraries, developed rapidly with AI assistance, aim to equip Eiffel with features on par with modern tech stacks, from web development to cryptography.*

Over the past months, the Eiffel community (spearheaded by the Simple Eiffel project) has launched a **"Christmas 2025 Challenge"** – creating a suite of Eiffel libraries to make Eiffel competitive with today's popular tech stacks[^1]. This effort has produced libraries like:

- `simple_json`
- `simple_sql`
- `simple_web`
- `simple_htmx`
- `simple_alpine`
- `simple_uuid`
- `simple_hash`
- `simple_jwt`
- ...and more

All were built in a matter of weeks using an AI-assisted development approach[^1].

Notably, an entire Eiffel-based web showcase site was recently developed in a single session by leveraging an AI (Anthropic's Claude) alongside Eiffel's design-by-contract checks[^2]. In that workflow:

1. The human provided direction ("build a hero section")
2. The AI generated Eiffel code
3. The Eiffel compiler and contract system immediately caught errors or contract violations
4. Errors were fixed in short iterative cycles[^3]

This experiment in Eiffel+AI showed both the **promise** and the **pitfalls** of current AI coding assistants. It dramatically accelerated development, but also highlighted challenges that need to be addressed before AI-generated Eiffel code can be truly "real-world" in robustness and maintainability.

---

## Current Limitations of LLM-Assisted Coding (Lay of the Land)

Despite the successes, using large language models (LLMs) like Claude or ChatGPT for coding still exhibits several serious limitations. Through experience, we've identified a few key pain points when using AI in the development of Eiffel libraries and applications:

### 1. Lack of Long-Term Memory

Modern LLMs do not retain conversation context beyond a single session or beyond their fixed context window. They essentially have **no true long-term memory**[^4]. This means:

- If you close a session or the conversation grows too long, the AI may "forget" important details discussed earlier
- Every new session starts fresh, requiring re-feeding of information
- Even within one session, if the conversation is lengthy, older details can be dropped once the context window is exceeded[^5][^6]

We've observed the AI often fails to recall decisions or information given just a day before. Any "compaction" or summarization of the conversation to save tokens can further cause loss of critical detail.

> **In short:** Without explicit memory aids, the AI cannot reliably remember past instructions, leading to repetition and inconsistency.

### 2. Eager to Code, Reluctant to Design

LLMs tend to jump straight into coding solutions without sufficient upfront design discussion. If given a feature request, the AI often outputs code immediately, treating the prompt as a spec – even when a higher-level architectural dialog is needed.

This lack of patience for planning can result in suboptimal designs. In practice, we had to constantly force the AI to slow down and engage in design brainstorming. This aligns with observations by others:

> "A common mistake is diving straight into code generation with a vague prompt instead of first defining the problem and planning a solution."[^7]

Best practices for AI-assisted engineering now emphasize writing a detailed specification with the AI and outlining the design before writing any code[^8]. Our current AI partner, however, doesn't naturally do this – it needs to be guided to focus on design.

### 3. Myopic Implementation Choices

Today's AI often makes implementation decisions **myopically**, without considering downstream effects or the broader project context. It will modify a module or class to satisfy the immediate request, but may not realize that this change breaks other components that depend on it.

For example, we saw it update classes in the `simple_sql` library to satisfy a new requirement, but failed to account for other libraries depending on `simple_sql`, leading to regression in those libraries.

This happens because an LLM works only on the snippet of context it's given (a "snapshot of input") and lacks a holistic view of the entire codebase or dependency graph[^9]:

> "LLMs operate on a snapshot of input… Without deep integration into the full codebase, they lack the context needed for consistent architecture. They often don't reason about scalability, modularity, or downstream impacts like a seasoned engineer would."[^10]

In practice, this means the AI might:
- Introduce tight coupling
- Break APIs
- Choose non-idiomatic designs that increase technical debt[^11]

It does not naturally perform impact analysis of a change. The burden falls on the human to detect these issues (via tests or code review) and then prompt the AI to fix inconsistencies across the project.

### 4. Lack of Persistent Specifications and Style Recall

The AI does not inherently remember the project's established conventions, documentation style, or specifications from prior work – unless we remind it every time.

For instance, each time we start a new `simple_*` library, we have to reiterate our standards (e.g., "create a README.md and docs folder following the style of earlier simple_* libraries"). The AI doesn't on its own recall these past instructions or infer the style guide.

This is a side effect of the first limitation (no long-term memory) and the fact that the model only knows what's in the prompt. Expert LLM users have found that you must explicitly provide context and guidance for each new task[^12][^13].

Tools like Claude's "Projects" mode or VS Code plugins attempt to include relevant files automatically, but the bottom line is:

> **LLMs are only as good as the context you provide – they will not reliably recall prior project knowledge on their own.**[^14]

If we forget to restate a crucial requirement (say, "include design-by-contract assertions" or "use the naming conventions we used before"), the AI's output may omit or violate it. This leads to the human getting "tired of having to do that" repeatedly.

---

### Summary of Limitations

While our AI coding companion (Claude) has enormous strengths – speed, knowledge, ability to generate working Eiffel code – it also has clear weaknesses in:

- Context retention
- Design foresight
- Holistic reasoning
- Consistency

These weaknesses are not due to "faults" in the AI per se, but inherent to how LLMs function and the limitations of prompt-based development. As one experienced engineer noted:

> "Using LLMs for programming is not a push-button magic experience – it's difficult and unintuitive, and getting great results requires learning new patterns."[^15]

We now realize that to harness the AI's strengths for serious software development, we need a more structured approach that compensates for these weaknesses. This is where our vision for **simple_dev** comes in.

---

## Vision: A Guided AI Development Orchestrator (simple_dev)

**simple_dev** is envisioned as a new `simple_*` tool/library that will act as an **AI development orchestrator** for Eiffel projects. It's not meant to replace the AI (we still leverage LLMs like Claude, ChatGPT, Gemini, or local models), but to use the AI in a far more controlled, efficient, and context-aware manner.

In essence, simple_dev will **wrap around the AI**, providing the "glue" of project knowledge, best practices, and workflow management so that the AI's contributions are consistently correct and aligned with our goals.

> **The driving idea:** Put guardrails and a guiding framework around the prompt-response cycle, making AI a truly effective pair-programmer integrated into Eiffel's software engineering methodology.

### Our Goals for simple_dev

#### Persistent Knowledge & Context

Never lose important information given to or produced by the AI. simple_dev will maintain project knowledge in a structured form (likely via SQLite databases and files) that can be reloaded into context as needed. This includes:

- Requirements
- Design decisions
- Specifications
- Coding guidelines
- Past conversation summaries

By having an internal "memory" of the project, the orchestrator can re-inject relevant context into prompts, mitigating the AI's forgetting problem.

**Example:** If a new session starts, simple_dev can automatically feed in the high-level spec and style guidelines for the project from its database. If the conversation gets too long, it can summarize and store older parts, and recall them on demand when relevant.

This long-term memory layer ensures continuity across sessions and days, so the AI doesn't act like a blank slate every morning.

#### Adaptive Conversation Flow Across the Dev Lifecycle

simple_dev will manage the interaction with the AI in **phases** that mirror a proper software development lifecycle. Rather than every prompt being an isolated Q&A, the tool will guide the AI through stages:

##### Phase 1: Ideation & Requirements Gathering

In this stage, the developer can brainstorm with the AI. The orchestrator will prompt the AI to ask clarifying questions about the project idea and ensure all requirements and constraints are fleshed out. The output is a clear problem definition.

This addresses the AI's tendency to skip straight to coding – instead, simple_dev forces a preliminary dialogue about *what* to build before *how* to build it. (As an analogy, experienced AI users now "begin by defining the problem and planning a solution"[^16], and we'll encode that behavior.)

##### Phase 2: Design & Specification

Next, simple_dev will lead the AI in producing a detailed specification document (e.g., a `spec.md` or design description). The AI will be prompted to outline:

- The architecture
- Key classes/modules
- Their responsibilities
- Interfaces
- Testing strategy

Based on the requirements, the orchestrator can ensure this spec includes important Eiffel-specific considerations (Design by Contract expectations, inheritance structures, etc.).

We'll likely prompt the AI to iteratively refine the design, possibly asking *"Does this design handle all requirements and edge cases?"* – effectively simulating a design review. This phase results in a comprehensive plan that both human and AI can reference.

In the words of one engineer, it's like doing "a waterfall in 15 minutes" – a rapid upfront planning that makes subsequent coding much smoother[^17]:

> "We compile this into a comprehensive spec.md – containing requirements, architecture decisions, data models, and even a testing strategy. This spec forms the foundation for development."[^18]

In simple_dev, this spec will be saved (and possibly versioned) in the project's database and files, to be used throughout the development.

##### Phase 3: Task Planning

With a high-level design in place, simple_dev will break down the implementation into logical tasks or milestones. Essentially, it will create a **to-do list** or ticket list for the AI (and human) to tackle one by one[^19].

The AI can assist in this breakdown as well – e.g., generating a list of steps:
1. Implement class X
2. Then add feature Y
3. Write test for Z
4. etc.

The orchestrator ensures the plan is coherent and complete before proceeding. Each task can be relatively small and self-contained.

##### Phase 4: Incremental Implementation (Coding Stage)

Here's where the AI actually writes Eiffel code, but under strict guidance. simple_dev will take one task at a time, gather the necessary context (relevant part of spec, relevant existing code, coding conventions), and prompt the AI to generate the code for that task.

Crucially, we will **limit the scope of each prompt** to a focused task – aligning with the idea that:

> "LLMs do best when given focused prompts: implement one function, fix one bug, add one feature at a time."[^20]

By iterating in small chunks, we reduce the chance of the model going off-track or producing a large incoherent blob of code[^21]. After each chunk, simple_dev can insert a validation step (compiling the code, running tests for that unit).

##### Phase 5: Testing & Verification

After or alongside implementation, simple_dev will ensure the new code is tested:

- Eiffel's design by contract will automatically provide some runtime verification (e.g., postconditions checking outcomes)
- We will also integrate any unit tests or use Eiffel's testing frameworks (like EUnit or manual tests)

As part of the spec phase, we plan to have a testing strategy – possibly even auto-generating some basic tests or at least identifying test cases. simple_dev can prompt the AI to generate unit tests for a module after implementing it, or use existing tests. Then it will compile and run these tests. Any failures or contract violations will be caught in this phase.

##### Phase 6: Debugging & Refinement

If tests or the compiler reveal issues, simple_dev will capture those error messages or failing assertions and **feed them back to the AI for debugging**. This creates a tight feedback loop where the AI can suggest a fix, apply it, and re-run the test – repeatedly until the issue is resolved.

Such rapid iteration is something AI excels at, as long as we provide the feedback. For instance, if a contract fails or an Eiffel compilation error arises, simple_dev will format that information and include it in the next prompt like:

> "The test for X failed with this output… or The compiler error is Y… please fix."

This mirrors workflows reported elsewhere, where an AI agent is given failing test results and can quickly suggest corrections[^22][^23].

Indeed, in our earlier manual process we often did this by copying compile errors into Claude; simple_dev will automate it. The goal is that the AI "won't consider the task done until all tests pass," effectively using the tests/contracts as a quality gate[^24].

##### Phase 7: Integration & Regression Checks

Because Eiffel projects often consist of multiple libraries (as in the Simple Eiffel ecosystem), a change in one library may affect others. simple_dev will be aware of project dependencies.

After making changes to (or adding) a library, it can run a **regression build** of the whole suite (using something like `simple_ci`). Thanks to the existing Simple CI tool, we can compile multiple Eiffel projects in one go and produce a machine-readable report[^25][^26].

simple_dev could invoke such a CI run and then parse the results. If, say, modifying `simple_sql` caused `simple_web` to no longer compile, the orchestrator will catch that and prompt the AI with those specific errors (rather than discovering it much later manually).

This ensures the AI considers system-wide consequences, not just the local code. It effectively forces the AI to "think with a holistic mindset" by confronting it with any integration issues it caused.

##### Phase 8: Documentation & Artifacts

At appropriate points, simple_dev will also prompt for non-code outputs that are part of best practices:

- Updating the README.md
- Generating API documentation comments
- Preparing deployment scripts

These often get neglected by AI unless asked. The orchestrator can enforce that when a new library or major feature is finalized, the AI generates the README and usage examples (using the templates and style we've established in the ecosystem).

It can also ensure things like version numbers are updated, changelogs written, etc., based on the patterns from earlier libraries. By automating these prompts, we won't have to manually remind the AI to do them each time.

##### Phase 9: Release & Delivery

Finally, simple_dev can assist in packaging the project – for Eiffel, this could include:

- Generating final C compilation (finalized build)
- Producing binaries (e.g., DLLs or executables)
- Creating installer scripts if needed (e.g., Inno Setup installers)

While these steps are more tooling-oriented, the AI could help write an Inno Setup script or an installer README if provided with context. The orchestrator ensures the AI has all necessary info (file paths, version numbers, etc.) to do this correctly.

---

### Scaling Across the Lifecycle

In essence, simple_dev will scale the AI's involvement across the **entire dev lifecycle** – from initial idea discussions to final deployment – while maintaining appropriate structure at each level.

It will know when to switch the AI's role from:
- Brainstorming partner → Code writer → Debugger → Documenter

...following a logical progression.

This should feel like a more **continuous interaction model** where the AI is always engaged but in different modes depending on the stage.

Equally important, simple_dev's interaction model **scales in granularity**: the AI will handle everything from single-line code fixes up to high-level architectural suggestions, but always within a scoped context.

The tool will smartly **zoom in or out**: it can ask the AI to focus on a tiny detail (like renaming a variable or fixing a single contract) or to consider the big picture (like reviewing whether the overall design meets the spec), by feeding it the appropriate amount of context for each.

Managing this scope is key to using tokens efficiently and keeping the AI on track.

---

## Key Components and Features of simple_dev

To realize the above vision, simple_dev will incorporate several components and techniques. Below we outline the major features and how they address the earlier limitations:

### 1. Unified AI Client with Multi-Provider Support

We will leverage the existing **simple_ai_client** library as the backbone for connecting to various AI models. This library already supports multiple AI providers – for example:

- Local models via **Ollama**
- Cloud models like **Anthropic Claude** and **OpenAI GPT**[^27]

simple_dev can use this to flexibly switch or use whichever AI the user configures (e.g., use Claude for code generation, or a local Qwen-7B for smaller tasks to save costs).

The user will provide API keys or local endpoints, and simple_dev will route requests accordingly through simple_ai_client's unified interface. This also means we can use **different models for different subtasks**: perhaps a big model for complex code generation and a smaller one for running quick embedding queries or summarization – abstracting the model specifics away.

### 2. Persistent Project Database (Knowledge Base)

A core piece will be an **SQLite-backed knowledge store** where specifications, design decisions, context embeddings, and past conversation summaries are recorded. This addresses the memory problem by giving the orchestrator its own memory.

**Example:** After the spec is created, it's saved in the DB (and possibly as a file `spec.md`). If the conversation is resumed later, simple_dev will fetch the spec and remind the AI of it.

We can store structured data too – perhaps:
- A table of requirements
- A table of key architecture decisions

...which can be selectively included in prompts.

The DB can also track what has been done (task status) so that if a session is interrupted, we know where to resume. Essentially, the **project DB becomes the source of truth** that persists beyond the ephemeral AI chat context.

### 3. Vector Embedding Knowledge and Semantic Search

Building on simple_ai_client's embedding capabilities, simple_dev can implement a **semantic memory** of things like error messages and code snippets.

In fact, simple_ai_client already provides an `AI_EMBEDDING_STORE` that uses SQLite to store error texts and their resolutions as vectors, enabling similarity search[^28][^29].

We will use this feature (and extend it) to allow the system to recall solutions to known problems. For example:
- If the Eiffel compiler throws a certain error (say a common one about a missing creation procedure or a type mismatch), and we solved it before, simple_dev can retrieve that from the store and either automatically apply the fix or prompt the AI with "This looks similar to a past error; the fix then was XYZ"

This reduces repeated work and helps the AI not re-invent fixes.

Additionally, we might embed other text, such as portions of the codebase or documentation, to enable semantic lookup ("find relevant context for this prompt"). Before asking the AI a question, simple_dev could vector-search the spec or code to pull in only the most relevant snippets, rather than always sending everything.

This technique keeps prompts concise (saving tokens) while still giving the AI what it needs.

### 4. Structured Prompt Templates and Guidance

Rather than freeform chat, simple_dev will use **prompt templates** for different situations – ensuring the AI always gets clear instructions and necessary context.

**Example templates:**

| Situation | Template Contents |
|-----------|-------------------|
| Implementing a new class | Brief of the class purpose from the spec, reminder of coding conventions, exact task description |
| Bugfix | Error message, relevant code snippet, request for corrected code diff |

These structured prompts act as **guardrails**, preventing the AI from going off-topic or ignoring important factors.

We will encode Eiffel best practices into the prompts. For example, at the top of prompts we might insert a system message like:

> "You are an Eiffel expert assistant. Always enforce Design by Contract and produce code consistent with Eiffel style. If designing a new class, ensure it has a creation procedure `make`, a full class-level comment, and relevant examples."

By doing this systematically, we relieve the human from repeatedly typing the same guidance. Moreover, as we accumulate more project knowledge, these prompt templates can evolve (perhaps even dynamically incorporating lessons learned from earlier tasks).

### 5. Enforcing Eiffel Design by Contract & Standards

Because Eiffel's philosophy is centered on correctness and clear contracts, simple_dev will **bake those standards into every step**. That means new code generated should have appropriate:

- Preconditions
- Postconditions
- Class invariants (where applicable)

If the AI omits them, the orchestrator can prompt it to add contracts (we can detect if a routine has no postcondition and ask for one).

Eiffel is also big on documentation; each class should have a header note, each feature a brief description. simple_dev can ensure the AI includes these by default.

This not only yields better code but also uses the contracts as **additional verification** (if the AI writes them, it clarifies the intended behavior, and then we can catch if they're violated).

Essentially, we want the AI to act in the spirit of **Eiffel methodology** – something it won't do unless explicitly told. By maintaining a store of Eiffel best practices (or even a static prompt segment with guidelines), we ensure consistent application of Eiffel's Method and style.

This addresses the inconsistency issue: the AI will be reminded of the "past experience" (like always create a README for a new lib, always include examples in docs, etc.).

Over time, we could even encode these as a library of **"Claude Skills"-like modules** (a concept where repeatable procedures and domain expertise are packaged for the AI[^30]). For instance, a "Create Eiffel Library Template" skill could automatically inject the procedure to set up the directory, README template, docs skeleton, etc., whenever a new library creation task is invoked.

By moving these from ad-hoc human reminders to programmed behavior, we get more reliable and repeatable results[^31].

### 6. Integration with Eiffel Build and Test Tools

simple_dev will deeply integrate with the Eiffel toolchain to provide the AI with **immediate feedback** from the compiler and tests. This is where existing tools like **simple_ci** come in.

simple_ci can compile multiple projects and generate a JSON report of success/failure[^32][^33]. In our orchestrator, after the AI writes or modifies code, we can automatically run `ec` (the Eiffel compiler) on the affected project(s). If compilation fails, we capture the error output.

We may use simple_ci's JSON output mode to get a structured list of errors and projects status[^34][^35]. That JSON can be directly fed to the AI (which is easier for it to parse than raw console text)[^36].

The same for test results – if we have a test suite, we run it and collect any failures.

By programmatically looping the AI's output through compile/test and then through AI analysis of errors, we achieve an **autonomous fix loop**:

```
Code -> Compile -> (if errors) Explain & Fix -> Code (updated) -> Compile again… until clean
```

This is analogous to what some advanced AI coding agents do with CI pipelines, turning bug-fixing into a collaborative loop[^37].

With simple_dev, a developer could hit "go" on a high-level task and watch the tool and AI iterate until the code compiles and all tests pass – all the while logging the conversation and actions for transparency.

### 7. Token Efficiency and Cost Awareness

A design priority for simple_dev is to **minimize token usage** (and by extension API costs and latency) during AI interactions. That means we won't just dump the entire codebase or spec into every prompt – we'll be selective.

**Techniques to achieve this:**

| Technique | Description |
|-----------|-------------|
| **Vector search** | Only include relevant context snippets (functions or classes likely related to the task at hand, or just the diff of code that changed) |
| **Summarization** | Summarizing or abstracting large context when detail isn't needed (e.g., "Module X provides A, B, C functionality" instead of full code) |
| **Caching** | Caching AI responses (if we ask the AI to summarize the spec, we store that summary for reuse) |
| **Smaller steps** | Breaking work into smaller pieces can be more token-efficient because each prompt+response focuses on one thing |
| **Local models** | Using local models for certain tasks (e.g., a lightweight local model for simple Q&A or boilerplate generation)[^38] |
| **Usage monitoring** | Monitoring token usage and providing tools for the user to configure limits (dry-run mode, cost caps) |

By being efficient, we allow longer sessions within context windows (some models have up to 100k token context which is huge, but cost scales with usage, so efficiency still matters).

### 8. Interactive CLI/TUI for the Developer

Initially, simple_dev will run as a **CLI (Command-Line Interface) application**, likely with a text-based UI to guide the developer through the process.

The idea is that a user can launch simple_dev in a terminal, and engage in a continuous interactive session where the tool prints information and questions, and the developer can provide input or confirmations.

We might incorporate a terminal-based UI (a Text UI, possibly leveraging a library like `simple_tui` if available) to present menus or formatted outputs.

**Examples of interaction:**
- After generating a spec, the tool could show a diff or allow the user to edit the spec if desired
- When multiple fixes are possible for a bug, it might list them and ask the user to choose

While initially a CLI/TUI is simpler to implement and sufficient for a single developer workflow, we foresee later possibly integrating with an IDE or GUI. (Notably, the Eiffel community is exploring VS Code integration; one could imagine simple_dev hooking into an Eiffel VS Code extension down the line.)

However, for now, focusing on a robust CLI logic is the priority, which can later be wrapped in other interfaces.

### 9. Generality and Configurability

Though we are tailoring simple_dev for Eiffel development (because we believe Eiffel's strong-contract paradigm pairs excellently with AI-assisted coding), we plan to keep the system **flexible** so it could, in theory, be used with other languages or workflows.

Users will be able to configure:
- Which AI model to use
- API keys
- Prompt templates or custom guidelines

The goal is to make the tool useful to any Eiffel developer (and maybe broader audience) – someone should be able to download the simple_dev tool, set their API key, and immediately use AI to generate and maintain Eiffel code with confidence.

It should:
- **Limit the cost** of using AI by optimizing prompts
- **Maximize utility** by encoding a lot of Eiffel expertise into the interactions (so that even a newcomer to Eiffel can benefit from built-in best practices)

---

## Embracing Eiffel's Strengths: Design by Contract meets AI

One of the motivations for building simple_dev is the observation that **Eiffel's philosophy and toolchain can significantly enhance AI-assisted development** – and vice versa. Eiffel is uniquely positioned to take advantage of AI-generated code because of several factors:

### Design by Contract as Automatic Validator

Eiffel's built-in contracts (`require`, `ensure`, invariants) act like **executable specifications**. When the AI writes code with contracts, those contracts immediately provide a way to validate the AI's output against the intended behavior.

In our Eiffel+AI experiments, we saw that postconditions could catch incorrect output that appeared plausible. For example, a postcondition checking that an HTML page contains a certain substring caught a missing element that the AI had overlooked[^39].

This tight feedback is something we will leverage heavily. simple_dev will always encourage the AI to specify contracts for important properties. This way, if the AI misunderstands something, it's often revealed by a contract failing at runtime.

It's much cheaper to catch an error through a contract or test and have the AI fix it, than to have silent logical bugs. As Addy Osmani noted:

> "Those who get the most out of coding agents tend to be those with strong testing practices… an agent like Claude can fly through a project with a good test suite as safety net."[^40]

In Eiffel, contracts are part of that safety net.

### Compiler Errors as Guidance, Not Obstacles

Eiffel is a compiled language with strict type checking. For an AI, hitting a compiler error is not a failure; it's **feedback for learning**.

The Simple Eiffel approach has been to compile early and often – essentially using the compiler to tell the AI what's wrong. For instance, if the AI calls a feature that doesn't exist, the compiler will throw an error that we can feed back. The AI then knows to implement or correct that feature.

This is analogous to test-driven development but at the compilation level. With simple_dev, after each code generation step, we'll compile immediately, capture any errors, and present them to the AI. The AI, having been trained on lots of code, can usually interpret compiler errors quite well and suggest fixes.

This automated compile-fix loop continues until no errors remain. In effect, the **Eiffel compiler (and any static analysis tools) become part of the AI's feedback loop**, reducing hallucinations and type mistakes.

It's worth noting that, unlike dynamic languages, Eiffel won't let many errors slip by – which is good when an AI that sometimes produces minor mistakes is writing the code. We anticipate that this approach will yield **correct-by-construction** code by the time the process is done.

### High-level Abstractions and Clear Syntax

Eiffel's language design (e.g., explicit type declarations, relatively verbose and self-documenting syntax) might actually help the AI stay on track.

The clarity of Eiffel code can make it easier for the model to reason about and modify it without confusion (compared to languages with lots of subtle gotchas).

Also, Eiffel encourages thinking in terms of contracts and classes first – exactly the kind of thinking we want the AI to adopt through prompting. We will instruct the AI to follow Eiffel idioms (like command-query separation, immutability of functions, etc.), which in turn produces code that is easier to verify.

> **In short:** Eiffel's strong structure complements the AI's need for explicit guidance.

### Rich Legacy of Specifications

Eiffel has a tradition of formal specifications and even tooling like **AutoProof** (for static verification). In the future, simple_dev could integrate such tools as well – for example, running AutoProof on critical classes to formally verify correctness, and then having the AI fix any proof failures.

While that may be beyond the initial scope, it aligns perfectly with the idea of **provable software built with AI assistance**. The more we can harness Eiffel's verification tools, the more we can trust the AI-produced code.

(A long-term vision might even compile Eiffel to WebAssembly for an online playground, or integrate other analysis to catch things contracts might not – simple_dev provides a framework to add these steps when ready.)

### Consistency and Maintainability

The Simple Eiffel libraries have placed emphasis on clean, uniform documentation and licensing (MIT License for all libs). By ensuring the AI always incorporates these elements (like adding the MIT license header, using the same documentation generator format), we maintain a **professional consistency** across the ecosystem.

This was already noted by Eiffel community reviewers – the README files and docs of `simple_*` libs look "very nice and professional"[^41].

We want to keep that standard high even as more code is generated. The orchestrator will thus not only generate code, but also the peripheral pieces (docs, examples, build scripts) that make a library truly usable and polished.

---

## Inspiration from Similar Projects and Industry Trends

Our approach with simple_dev is ambitious, but we're not working in isolation – we are inspired by and will improve upon patterns emerging in other AI-driven development tools:

### AI "Conductor" vs. Freeform Chat

Instead of treating the AI as an autonomous agent given an open-ended goal (as some projects like AutoGPT or GPT-Engineer have tried), we are essentially creating an **AI conductor** that coordinates the AI's efforts through a predetermined workflow.

This philosophy is echoed by experienced practitioners:

> "Treat the LLM as a powerful pair programmer that requires clear direction, context, and oversight rather than autonomous judgment."[^42]

In other words, the human remains in charge of the process (and ultimately accountable for the result[^43]), but the heavy lifting is delegated to the AI in a controlled manner.

This yields a better outcome than expecting the AI to magically handle high-level planning itself. Projects such as GPT-Engineer and MetaGPT have highlighted the need for multi-step planning (GPT-Engineer, for instance, generates a plan and then code) – we take a similar concept but tailor it to Eiffel and embed it in a continuous interactive loop rather than a one-shot generation.

The Addy Osmani workflow we cited is a real-world validation of this approach: spec first, break into tasks, iterative development, test frequently[^44][^45][^46]. We are essentially **codifying those best practices into software**.

### Continuous Integration Loops

Big tech is already experimenting with AI agents that integrate with CI pipelines. For example, Google's Jules or GitHub's Copilot Agent can clone a repo, run tests, make fixes, and even open pull requests autonomously[^47].

They operate asynchronously in the cloud, acting as an AI DevOps engineer. simple_dev is conceptually similar but currently aimed at a local developer's workflow.

We draw inspiration from those tools in how they use CI feedback: our use of simple_ci and feeding its report to the AI is akin to what those agents do with GitHub Actions results[^48].

The key difference is simple_dev will run locally (or on the user's machine) and give the developer a chance to intervene at each step. This provides a reassuring level of control (the developer can inspect each AI output, run additional tests, etc., before moving on).

We believe a **semi-automated, human-in-the-loop** approach is safer in the context of generating critical software like system libraries.

### Context Window Workarounds

Many tools are coming out to address the context limitation of LLMs. For instance, the Augment Code engine (as described in ByteByteGo) indexes entire codebases so the AI can retrieve relevant pieces on the fly[^49].

There are also utilities like `gitingest` or `repo2txt` which dump relevant code parts to feed into GPT[^50].

Our use of an embedding store and selective context inclusion is in line with these trends. Essentially, simple_dev serves as an **intelligent retrieval-augmented generation system**: it will retrieve the necessary code or docs for the AI instead of expecting the AI's own limited memory to suffice.

This is a proven approach to scaling AI to work on large projects without hitting the "lost in the middle" problem of long context[^51].

We'll continue to monitor advances here (for example, if Claude's 100k token context or tools like LangChain's agents can be leveraged, we might integrate those capabilities).

### Skill-Based AI Responses

The concept of AI "skills" or tool usage is gaining ground (e.g., Claude Skills, OpenAI function calling, etc.)[^52].

Our design for simple_dev inherently gives the AI certain "skills" (like the ability to call the compiler, run tests, etc., albeit mediated by the orchestrator). As AI platforms allow more programmatic interaction, we can imagine simple_dev instructing the AI to output not just code, but possibly structured commands that the tool can interpret to perform actions (like a mini agent loop).

For now, we'll likely keep things straightforward (the tool decides when to call compiler or tests), but in the future, a more agentive AI could be integrated, that knows how to invoke simple_dev's functions directly when needed.

This is an area of innovation we'll watch – for instance, packaging Eiffel-specific expertise as a skill module that can be reused across projects[^53].

The architecture of simple_dev should be amenable to updates as the AI APIs evolve (e.g., if an API lets the AI reference code files by name, we could utilize that instead of injecting code text).

### Human Oversight and UI/UX

A crucial insight from others is that **keeping a human in the loop is vital**. simple_dev will not remove the developer from the process – rather, it amplifies the developer's productivity while continuously requiring their guidance and review.

We anticipate the UX to involve the tool presenting AI outputs and asking for confirmation or adjustments. The developer might choose among options or edit the AI's code themselves at times.

This interactive way of working is echoed by many who caution:

> "AI will happily produce plausible-looking code, but you are responsible for quality – always review and test thoroughly."[^54]

Our system is designed to make that reviewing easier by breaking changes into small pieces and automating the testing.

The end goal is that using simple_dev, **a single developer can accomplish what would normally take a small team**, but without sacrificing quality.

All the traditional software engineering rigor – design discussions, version control, continuous testing, documentation – are not discarded; rather, they're baked into the workflow and often facilitated by the AI. As one commentary put it:

> "The classic practices – design before coding, write tests, use version control, maintain standards – not only still apply, but are even more important when an AI is writing half your code."[^55]

We fully embrace that mindset in simple_dev's design.

---

## Conclusion

In summary, **simple_dev** aims to be a comprehensive AI-assisted development partner tailored for Eiffel. It will address the major shortcomings of raw AI chat-based coding by introducing **memory, structure, and guardrails** grounded in sound software engineering principles.

By guiding the AI through a structured lifecycle – from brainstorming to design to implementation, testing, and deployment – we ensure that the resulting code is:

- ✅ Correct
- ✅ Maintainable
- ✅ Aligned with Eiffel's high standards of reliability

This project is both a natural extension of the Simple Eiffel ecosystem and an innovation that could significantly boost developer productivity. It takes the hard-earned lessons from our recent AI coding experiments (and industry best practices) and codifies them into a repeatable process.

The end result will be a tool that any Eiffel developer (or team) can use to collaborate with AI efficiently – harnessing the speed and creativity of LLMs, while the tool mitigates the forgetfulness, impatience, and narrow focus that LLMs often exhibit.

By prioritizing strong design, continuous verification, and knowledge preservation, simple_dev will allow Eiffel programmers to confidently generate new libraries and applications at a blazing pace without losing control of quality.

> **It's about making AI a true engineering assistant rather than a stochastic code generator.**

If successful, this could position Eiffel as a frontrunner in demonstrating how classic "provable software" ideals can merge with cutting-edge AI capabilities. Eiffel's 2026 renaissance could very well be powered by such human+AI co-development workflows – delivering software that is not only **built faster**, but **built right**.

### Next Steps

Proceeding from here, the next steps will involve:

1. Prototyping the components described (especially the conversation flow manager and integration with simple_ai_client and simple_ci)
2. Incrementally testing this approach on a real Eiffel project
3. Refining the prompts, databases, and algorithms that constitute simple_dev with each iteration

The vision laid out is broad, but by breaking our own work into manageable pieces (just as we will instruct users to do), we'll build up this capability step by step.

The excitement around this project is palpable – **"we are DOING THIS!!!"**[^56] – and with careful design, simple_dev could become an empowering tool for Eiffel developers worldwide, turning the dream of AI-augmented software creation into a practical reality.

---

## Sources

1. Liberty Lover (Larry) – Eiffel Users Mailing List, discussion on Simple Libraries challenge[^1]
2. Simple Showcase project notes – Eiffel + AI web development example[^2][^3]
3. ByteByteGo Newsletter – The Memory Problem: Why LLMs Sometimes Forget[^4]
4. Metabob Blog – Pitfalls of Using LLMs in Software Development[^9][^10][^11]
5. Addy Osmani – "My LLM coding workflow going into 2026", best practices for AI-assisted coding[^7][^8][^12][^13][^14][^15][^16][^17][^18][^19][^20][^21][^22][^23][^24][^30][^31][^37][^40][^42][^43][^44][^45][^46][^47][^48][^50][^52][^53][^54][^55]
6. Simple Eiffel GitHub – simple_ai_client documentation (AI providers, embeddings)[^27][^28][^29][^38]
7. Simple Eiffel GitHub – simple_ci documentation (CI tool and Claude integration)[^25][^26][^32][^33][^34][^35][^36]
8. Eiffel Users Mailing List – feedback from Eric Bezault on Simple libraries docs[^41]
9. ByteByteGo – Context window problems[^49][^51]

---

## Citations

[^1]: [Simple Lib's - First Two...](https://groups.google.com/g/eiffel-users/c/GalH2I0GY-A) - groups.google.com
[^2]: [Simple Showcase (SSC) Website Plan](https://groups.google.com/g/eiffel-users/c/bqsabMSygzM) - groups.google.com
[^3]: [Simple Showcase (SSC) Website Plan](https://groups.google.com/g/eiffel-users/c/bqsabMSygzM) - groups.google.com
[^4]: [The Memory Problem: Why LLMs Sometimes Forget Your Conversation](https://blog.bytebytego.com/p/the-memory-problem-why-llms-sometimes) - blog.bytebytego.com
[^5]: [The Memory Problem: Why LLMs Sometimes Forget Your Conversation](https://blog.bytebytego.com/p/the-memory-problem-why-llms-sometimes) - blog.bytebytego.com
[^6]: [The Memory Problem: Why LLMs Sometimes Forget Your Conversation](https://blog.bytebytego.com/p/the-memory-problem-why-llms-sometimes) - blog.bytebytego.com
[^7]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^8]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^9]: [The Hidden Pitfalls of Using LLMs in Software Development](https://metabob.com/blog-articles/the-hidden-pitfalls-of-using-llms-in-software-development---why-language-models-arent-the-silver-bullet-you-might-think.html) - metabob.com
[^10]: [The Hidden Pitfalls of Using LLMs in Software Development](https://metabob.com/blog-articles/the-hidden-pitfalls-of-using-llms-in-software-development---why-language-models-arent-the-silver-bullet-you-might-think.html) - metabob.com
[^11]: [The Hidden Pitfalls of Using LLMs in Software Development](https://metabob.com/blog-articles/the-hidden-pitfalls-of-using-llms-in-software-development---why-language-models-arent-the-silver-bullet-you-might-think.html) - metabob.com
[^12]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^13]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^14]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^15]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^16]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^17]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^18]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^19]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^20]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^21]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^22]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^23]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^24]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^25]: [GitHub - simple-eiffel/simple_ci](https://github.com/simple-eiffel/simple_ci) - github.com
[^26]: [GitHub - simple-eiffel/simple_ci](https://github.com/simple-eiffel/simple_ci) - github.com
[^27]: [GitHub - simple-eiffel/simple_ai_client](https://github.com/simple-eiffel/simple_ai_client) - github.com
[^28]: [GitHub - simple-eiffel/simple_ai_client](https://github.com/simple-eiffel/simple_ai_client) - github.com
[^29]: [GitHub - simple-eiffel/simple_ai_client](https://github.com/simple-eiffel/simple_ai_client) - github.com
[^30]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^31]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^32]: [GitHub - simple-eiffel/simple_ci](https://github.com/simple-eiffel/simple_ci) - github.com
[^33]: [GitHub - simple-eiffel/simple_ci](https://github.com/simple-eiffel/simple_ci) - github.com
[^34]: [GitHub - simple-eiffel/simple_ci](https://github.com/simple-eiffel/simple_ci) - github.com
[^35]: [GitHub - simple-eiffel/simple_ci](https://github.com/simple-eiffel/simple_ci) - github.com
[^36]: [GitHub - simple-eiffel/simple_ci](https://github.com/simple-eiffel/simple_ci) - github.com
[^37]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^38]: [GitHub - simple-eiffel/simple_ai_client](https://github.com/simple-eiffel/simple_ai_client) - github.com
[^39]: [Simple Showcase (SSC) Website Plan](https://groups.google.com/g/eiffel-users/c/bqsabMSygzM) - groups.google.com
[^40]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^41]: [Simple Lib's - First Two...](https://groups.google.com/g/eiffel-users/c/GalH2I0GY-A) - groups.google.com
[^42]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^43]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^44]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^45]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^46]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^47]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^48]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^49]: [The Memory Problem: Why LLMs Sometimes Forget Your Conversation](https://blog.bytebytego.com/p/the-memory-problem-why-llms-sometimes) - blog.bytebytego.com
[^50]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^51]: [The context window problem or why LLM forgets the middle of a...](https://bdtechtalks.com/2025/02/05/the-context-window-problem-or-why-llm-forgets-the-middle-of-a-long-file/) - bdtechtalks.com
[^52]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^53]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^54]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^55]: [My LLM coding workflow going into 2026](https://addyo.substack.com/p/my-llm-coding-workflow-going-into) - addyo.substack.com
[^56]: [Simple Lib's - First Two...](https://groups.google.com/g/eiffel-users/c/GalH2I0GY-A) - groups.google.com

---

### All Source Domains

- [groups.google.com](https://groups.google.com) - Eiffel Users Mailing List
- [blog.bytebytego.com](https://blog.bytebytego.com) - ByteByteGo Newsletter
- [addyo.substack.com](https://addyo.substack.com) - Addy Osmani's Substack
- [metabob.com](https://metabob.com) - Metabob Blog
- [github.com](https://github.com) - Simple Eiffel GitHub
- [bdtechtalks.com](https://bdtechtalks.com) - BD Tech Talks
