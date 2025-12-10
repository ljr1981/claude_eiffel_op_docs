# Jocelyn Fiat's Feedback on AI-Generated Code

**Date:** December 10, 2025
**Context:** Response to simple_* ecosystem and simple_lsp announcement

## Key Points

### AI as a Tool, Not an Oracle

> I think this is a good reminder that AI services are just tools, powerful and advanced tools, but in the end it has to be double checked by a human and expert in the given domain.

This is absolutely correct. The AI:
- Can generate plausible-looking code quickly
- May hallucinate details (as shown in his image example)
- Should always be verified by human experts
- Benefits enormously from Larry's domain expertise guiding the process

### Verification Burden

> I just fear that checking all the generated libraries may take time. Of course the associated spec docs, and test cases help.

Valid concern. Our approach:
1. Design by Contract catches many issues at compile time
2. Test suites for each library
3. Larry's expert review of all generated code
4. Incremental development with immediate testing

### Platform Support Transparency

> About the generated "simple_" libraries, I was surprised to see many of them are targeted only to Windows, it is clear from the dependencies, but maybe it would be good to list, for each, the supported platforms.

**Action items:**
1. Add platform support badges to each README
2. Create ecosystem-wide platform compatibility matrix
3. Document which libraries are portable vs platform-specific
4. Plan for cross-platform support where feasible

### The AI Hallucination Example

Jocelyn demonstrated how AI can confidently generate false information when asked about "John Smith Eiffel expert" - the AI made up details. This is a critical reminder:
- AI can fabricate plausible-sounding facts
- Always request sources
- Always verify with authoritative documentation
- Domain expert involvement is essential

### Larry's Role

> I must say that thanks to Larry's experience, Larry is able to use those tools efficiently and get amazing results quickly.

The human-AI collaboration model:
- AI brings speed and broad knowledge
- Human brings domain expertise and verification
- Larry's 20+ years of Eiffel experience guides quality
- The combination produces results neither could alone

## Our Response

1. Creating platform compatibility documentation
2. Adding platform badges to READMEs
3. Acknowledging Windows-first approach with cross-platform roadmap
4. Emphasizing that all code is human-reviewed

---

**From:**
Jocelyn Fiat
EiffelSoftware
