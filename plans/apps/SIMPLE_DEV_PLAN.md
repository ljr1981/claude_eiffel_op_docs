\# Project Plan: \*\*simple\_dev\*\* – AI-Orchestrated Eiffel Development



\## Background: The Simple Eiffel Ecosystem and AI-Assisted Workflow



\*Figure (context): The \*\*Simple Eiffel\*\* initiative is building a suite of lightweight Eiffel libraries (each named `simple\_\*`) to modernize the Eiffel ecosystem. These libraries, developed rapidly with AI assistance, aim to equip Eiffel with features on par with modern tech stacks, from web development to cryptography.\*



Over the past months, the Eiffel community (spearheaded by the Simple Eiffel project) has launched a \*\*“Christmas 2025 Challenge”\*\* – creating a suite of Eiffel libraries to make Eiffel competitive with today’s popular tech stacks. This effort has produced libraries like \*\*simple\_json\*\*, \*\*simple\_sql\*\*, \*\*simple\_web\*\*, \*\*simple\_htmx\*\*, \*\*simple\_alpine\*\*, \*\*simple\_uuid\*\*, \*\*simple\_hash\*\*, \*\*simple\_jwt\*\*, and more – all built in a matter of weeks using an AI-assisted development approach. Notably, an entire Eiffel-based web showcase site was recently developed in a single session by leveraging an AI (Anthropic’s Claude) alongside Eiffel’s design-by-contract checks. In that workflow, the human provided direction (“build a hero section”), the AI generated Eiffel code, and then the Eiffel compiler and contract system immediately caught errors or contract violations, which were fixed in short iterative cycles. This \*\*experiment in Eiffel+AI\*\* showed both the promise and the pitfalls of current AI coding assistants. It dramatically accelerated development, but also highlighted challenges that need to be addressed before AI-generated Eiffel code can be truly \*“real-world”\* in robustness and maintainability.



\## Current Limitations of LLM-Assisted Coding (Lay of the Land)



Despite the successes, using large language models (LLMs) like Claude or ChatGPT for coding still exhibits several \*\*serious limitations\*\*. Through experience, we’ve identified a few key pain points when using AI in the development of Eiffel libraries and applications:



\- \*\*Lack of Long-Term Memory:\*\* Modern LLMs do \*\*not\*\* retain conversation context beyond a single session or beyond their fixed context window. This means if you close a session or the conversation grows too long, the AI may “forget” important details discussed earlier. Every new session starts fresh, requiring re-feeding of information. Even within one session, if the conversation is lengthy, older details can be dropped once the context window is exceeded. Any \*\*“compaction”\*\* or summarization of the conversation to save tokens can further cause loss of critical detail. In short, without explicit memory aids, the AI cannot reliably remember past instructions, leading to repetition and inconsistency.



\- \*\*Eager to Code, Reluctant to Design:\*\* LLMs tend to jump straight into coding solutions without sufficient upfront design discussion. If given a feature request, the AI often outputs code immediately, treating the prompt as a spec – even when a higher-level architectural dialog is needed. This \*\*lack of patience for planning\*\* can result in suboptimal designs. Best practices for AI-assisted engineering emphasize writing a detailed specification with the AI and outlining the design \*\*before\*\* writing any code. Our current AI partner, however, doesn’t naturally do this – it needs to be guided to focus on design. Without that guidance, it may produce code that technically works but doesn’t necessarily follow human-preferred design choices or misses edge cases that a design discussion would catch.



\- \*\*Myopic Implementation Choices:\*\* Today’s AI often makes implementation decisions \*\*myopically\*\*, without considering downstream effects or the broader project context. It will modify a module or class to satisfy the immediate request, but may not realize that this change breaks other components that depend on it. This happens because an LLM works only on the snippet of context it’s given and lacks a holistic view of the entire codebase or dependency graph. In practice, this means the AI might introduce tight coupling, break APIs, or choose non-idiomatic designs that increase technical debt. It does not naturally perform impact analysis of a change. The burden falls on the human to detect these issues (via tests or code review) and then prompt the AI to fix inconsistencies across the project.



\- \*\*Lack of Persistent Specifications and Style Recall:\*\* The AI does not inherently \*remember\* the project’s established conventions, documentation style, or specifications from prior work – unless we remind it every time. For instance, each time we start a new `simple\_\*` library, we have to reiterate our standards (e.g. \*“create a README.md and docs folder following the style of earlier simple\_\* libraries”\*). The AI doesn’t on its own recall these past instructions or infer the style guide. If we forget to restate a crucial requirement (say, “include design-by-contract assertions” or “use the naming conventions we used before”), the AI’s output may omit or violate it. This leads to the human getting “tired of having to do that” repeatedly. Maintaining consistency in style and specs across sessions is tedious and error-prone under the current ad-hoc prompting approach.



In summary, while our AI coding companion has \*\*enormous strengths\*\* – speed, knowledge, ability to generate working Eiffel code – it also has clear \*\*weaknesses\*\* in context retention, design foresight, holistic reasoning, and consistency. These weaknesses are not due to “faults” in the AI per se, but inherent to how LLMs function and the limitations of prompt-based development. Using LLMs for programming is \*not\* push-button magic; getting great results requires learning new patterns. To harness the AI’s strengths for serious software development, we need a more structured approach that compensates for these weaknesses. This is where \*\*our vision for `simple\_dev` comes in.\*\*



\## Vision: A Guided AI Development Orchestrator (simple\_dev)



\*\*simple\_dev\*\* is envisioned as a new `simple\_\*` tool/library that will act as an \*\*AI development orchestrator\*\* for Eiffel projects. It’s not meant to replace the AI (we still leverage LLMs like Claude, ChatGPT, Gemini, or local models), but to \*use\* the AI in a far more controlled, efficient, and context-aware manner. In essence, `simple\_dev` will wrap around the AI, providing the “glue” of project knowledge, best practices, and workflow management so that the AI’s contributions are consistently correct and aligned with our goals. The driving idea is to \*\*put guardrails and a guiding framework around the prompt-response cycle\*\*, making AI a truly effective pair-programmer integrated into Eiffel’s software engineering methodology.



\### Goals for simple\_dev



\- \*\*Persistent Knowledge \& Context:\*\* \*Never lose important information given to or produced by the AI.\* `simple\_dev` will maintain project knowledge in a structured form (likely via SQLite databases and files) that can be reloaded into context as needed. This includes requirements, design decisions, specifications, coding guidelines, and even past conversation summaries.



\- \*\*Adaptive Conversation Flow Across the Dev Lifecycle:\*\* `simple\_dev` will manage interaction with the AI in \*\*phases\*\* that mirror a proper software development lifecycle. Rather than every prompt being an isolated Q\&A, the tool will guide the AI through stages such as:



&nbsp; 1. \*\*Ideation \& Requirements Gathering:\*\* Brainstorm and clarify scope/constraints before any code.

&nbsp; 2. \*\*Design \& Specification:\*\* Produce and refine a durable spec (`spec.md`) that governs later work.

&nbsp; 3. \*\*Task Planning:\*\* Break work into explicit tasks/milestones (“tickets”) for controlled iteration.

&nbsp; 4. \*\*Incremental Implementation:\*\* Prompt in focused chunks; implement one unit at a time.

&nbsp; 5. \*\*Testing \& Verification:\*\* Generate/maintain tests; use contracts as executable spec.

&nbsp; 6. \*\*Debugging \& Refinement:\*\* Feed compiler/test failures back to the AI in tight loops.

&nbsp; 7. \*\*Integration \& Regression:\*\* Rebuild dependent libs and run regressions to prevent breakage.

&nbsp; 8. \*\*Documentation \& Artifacts:\*\* Enforce README/docs/templates, examples, changelog updates.

&nbsp; 9. \*\*Release \& Delivery:\*\* Package binaries and installers (e.g., Inno Setup) with artifacts.



This interaction model scales in \*\*granularity\*\*: from a single line-of-code fix to system-level architectural review, but always within scoped context windows and governed by stored specifications.



\## Key Components and Features of simple\_dev



\- \*\*Unified AI Client with Multi-Provider Support:\*\* Use `simple\_ai\_client` as the backbone to support Claude/OpenAI/local Ollama models (e.g., Qwen). This allows choosing different models per task (cost/quality tradeoffs).



\- \*\*Persistent Project Database (Knowledge Base):\*\* SQLite-backed project memory for specs, decisions, logs, task state, and summaries. Enables seamless resume across sessions.



\- \*\*Vector Embedding Knowledge and Semantic Search:\*\* Store and retrieve errors/solutions and relevant spec/code snippets via embeddings to keep prompts small and relevant.



\- \*\*Structured Prompt Templates and Guidance:\*\* Prompt templates by stage (spec/design/code/debug/docs) that always inject Eiffel best practices and project conventions.



\- \*\*Enforcing Eiffel Design by Contract \& Standards:\*\* Require contracts (require/ensure/invariant), class notes, feature docs, examples, and consistent library scaffolding.



\- \*\*Integration with Eiffel Build and Test Tools:\*\* Automatically compile after changes; run tests; parse output (optionally JSON via CI tooling) and feed failures to the AI.



\- \*\*Token Efficiency and Cost Awareness:\*\* Retrieval-based context, caching, summarization, small task iteration, and selective model usage to control cost and latency.



\- \*\*Interactive CLI/TUI:\*\* CLI-first with optional TUI (e.g., `simple\_tui`) for menus, diffs, approvals, and readable reports.



\- \*\*Generality and Configurability:\*\* User-configurable AI backends, keys, endpoints, and template overrides so others can install and use `simple\_dev` immediately.



\## Embracing Eiffel’s Strengths: Design by Contract meets AI



\- \*\*Contracts as Automatic Validator:\*\* Contracts turn intent into executable checks, catching AI mistakes early and guiding corrections.

\- \*\*Compiler Feedback Loop:\*\* Compile early/often; use typed compilation errors as structured feedback for the AI.

\- \*\*Clear structure aids reasoning:\*\* Eiffel’s explicit types and disciplined OO structure help constrain and clarify AI-generated changes.

\- \*\*Future verification hooks:\*\* Potential future integration with formal tools (e.g., AutoProof) fits the same loop structure.



\## Inspiration from Similar Projects and Industry Trends



\- \*\*Spec-first workflows:\*\* Industry best practices increasingly emphasize spec/design before code for LLM-driven work.

\- \*\*CI-driven agent loops:\*\* Modern AI tools integrate with CI feedback; `simple\_dev` does this locally with full transparency.

\- \*\*RAG for context scaling:\*\* Indexing/retrieval avoids dumping whole repos into prompts, improving accuracy and cost.

\- \*\*Skill modules and reusable procedures:\*\* Encode repeatable “skills” like “Create new simple\_\* lib skeleton” as templates and automated steps.

\- \*\*Human-in-the-loop:\*\* The developer remains the owner of truth and quality; `simple\_dev` makes oversight easier, not optional.



\## Conclusion



`simple\_dev` is about \*\*turning AI from a chat-based code generator into a disciplined, lifecycle-aware engineering assistant\*\* for Eiffel. It introduces memory, structure, and guardrails grounded in strong software engineering practice. It scales from idea → spec → code → test → regression → docs → release, while leveraging Eiffel’s strengths (contracts, compilation, clarity) to keep quality high.



The tool’s success will come from:

\- \*\*durable specs and stored decisions\*\*

\- \*\*tight compile/test loops\*\*

\- \*\*regression awareness across simple\_\* libs\*\*

\- \*\*prompt templates that encode best practice\*\*

\- \*\*token-efficient context retrieval\*\*

\- \*\*a pragmatic CLI-first UX\*\*



---



\## Sources (High-level)



This report was informed by:

\- Simple Eiffel ecosystem discussions and project documentation (simple\_\* libraries, `simple\_ai\_client`, `simple\_ci`)

\- Industry best practices around spec-first LLM workflows, retrieval-based context, and CI-driven feedback loops

\- Observed pitfalls in LLM-assisted development: context loss, premature coding, myopic changes, and inconsistency across sessions



