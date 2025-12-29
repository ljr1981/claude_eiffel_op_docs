# Speech Studio - Project Specification

**Date:** December 29, 2025
**Author:** Larry Rix + Claude
**Status:** Planning Phase
**Location:** simple_speech repository (new ECF target)

---

## Executive Summary

Build a Windows desktop application that provides a complete graphical front-end for the simple_speech library. This application will transform the command-line speech processing capabilities into an intuitive, feature-rich GUI experience.

**Historical note:** This project realizes the vision originally intended for simple_gui_designer (abandoned). The JSON specification approach and design-first methodology carry that spirit forward in a more focused, application-specific context.

---

## Part 1: JSON Specification Protocol

Before writing any code, develop a **JSON-based specification protocol** for defining GUI applications. This specification will describe:

- **Window definitions** - Main windows, dialogs, popups, and their properties
- **Widget layouts** - Component hierarchy, positioning, and relationships
- **Navigation flows** - How users move between screens and dialogs
- **Data bindings** - Connections between UI elements and underlying data models
- **Event mappings** - User actions and their corresponding system responses
- **State machine definitions** - Valid states, transitions, guards, and edge-case handling
- **Error flows** - What happens when things go wrong, validation failures, recovery paths

The specification must cover both **happy-path workflows** and **edge-case scenarios**, with explicit state-machine mappings that define all valid transitions and their preconditions.

---

## Part 2: Comprehensive Architecture Analysis

Conduct a thorough examination of the simple_speech library - its full depth and breadth - to understand every capability it offers. From this analysis, derive:

- **All possible user scenarios** - Every way a user might want to interact with speech processing
- **Feature inventory** - Complete catalog of simple_speech functionality to expose via GUI
- **Workflow patterns** - Common sequences of operations users will perform
- **Edge cases and error states** - Network failures, invalid files, processing interruptions, etc.

---

## Part 3: Multi-Tier Architecture Design

Create a detailed architectural plan with clear separation of concerns:

| Tier | Responsibility | Implementation |
|------|----------------|----------------|
| **GUI Tier** | User interface, visual presentation | simple_vision |
| **Business Logic Tier** | Workflow orchestration, validation, rules | New classes in simple_speech |
| **Data Abstraction Tier** | Database-agnostic persistence interface | **New library: simple_repository** |
| **Data Storage Tier** | Physical persistence | simple_sql (SQLite) |

### New Library: simple_repository

A dedicated simple_* library to provide the abstract persistence layer:

- Repository pattern implementation
- Unit-of-work support
- Query abstraction (not raw SQL)
- Pluggable backend adapters (simple_sql adapter ships first, others possible)
- Transaction management across repositories

This ensures the business logic tier never touches SQL directly and allows database backends to be swapped without modifying upper tiers.

---

## Part 4: Technology Stack

**Prioritization hierarchy:**
1. **simple_* libraries** - First choice for all functionality
2. **ISE EiffelBase/EiffelVision** - Only when no simple_* alternative exists
3. **Gobo libraries** - Last resort when neither of the above suffice

**Known stack components:**

| Component | Library |
|-----------|---------|
| GUI | simple_vision |
| Persistence abstraction | simple_repository (new) |
| Storage | simple_sql |
| Speech processing | simple_speech (core) |
| Configuration | simple_config |
| JSON handling | simple_json |
| File operations | simple_file |

---

## Part 5: Deliverables

All artifacts will reside within the simple_speech repository as a new ECF target (e.g., `speech_studio`):

### Markdown Planning Documents
- Architecture overview
- Database schema design
- Window/dialog specifications
- User workflow documentation
- State machine diagrams
- API contracts between tiers

### JSON Specification Files
- Window/dialog definitions
- Use-case scenarios (exhaustive - happy path AND edge cases)
- State machine definitions
- Test scripts for GUI validation

### Implementation Code
- The actual application classes

---

## Part 6: Design Philosophy

The application should be **impressive from a "what can I do with this?" perspective**. Users should immediately see the value and power of the speech processing capabilities:

- Intuitive workflows that guide users naturally
- Visual feedback that makes processing status clear
- Features that showcase the full capability of simple_speech
- Professional polish that inspires confidence
- Graceful handling of errors and edge cases

Innovation is encouraged wherever it adds genuine value to the user experience.

---

## Part 7: simple_vision Phase Completion Decision

### Productivity Baseline (December 28-29, 2025)

| Metric | Value |
|--------|-------|
| 2-day total | 30,614 LOC |
| Daily average | 15,307 LOC/day |
| Classes created | 83 classes in 2 days |
| Libraries completed | 2 (simple_vision, simple_speech) |

This establishes our planning baseline: **~15K LOC/day** or **~40 classes/day** at peak productivity.

### Current State: Phase 6.75 (Beta Complete)

What exists now:
- 52 widget classes, 7 demos
- Theming with dark mode
- TRUE input masking
- Data grid with sorting
- All core widgets functional

### Phase Completion Options

| Complete Through | Effort | Est. LOC | What You Gain |
|------------------|--------|----------|---------------|
| **Use as-is (6.75)** | 0 | 0 | Nothing new |
| **Phase 6.75 remaining** | 2-3 hours | ~2-3K | GUI test harness, SV_FORM validation |
| **Phase 7** | 4-6 hours | ~5-8K | Cairo graphics, gradients, shadows, waveforms |
| **Phase 8** | 6-8 hours | ~8-12K | WebView, Chart.js, Monaco editor |
| **Phase 9** | 2-3 hours | ~2-3K | Documentation, comprehensive tests |

### Value Proposition Analysis

#### Phase 6.75 Remaining (SV_FORM + GUI Test Harness)
```
Bang: ★★★☆☆  |  Investment: 2-3 hours
```
- SV_FORM/SV_FIELD cleans up Speech Studio's input forms
- GUI test harness automates UI testing with JSON use-cases
- **Directly feeds the JSON specification protocol**
- **Verdict:** High value, low cost - DO THIS

#### Phase 7 (Cairo/Graphics)
```
Bang: ★★★★☆  |  Investment: 4-6 hours
```
- **Waveform visualization** - Display audio waveforms during playback/editing
- **Gradients/shadows** - Modern, polished UI aesthetic
- **Custom drawing** - Progress indicators, visual feedback during transcription
- **Verdict:** Very relevant for a media application - RECOMMENDED

#### Phase 8 (WebView/Charts/Monaco)
```
Bang: ★★★★★  |  Investment: 6-8 hours
```
- **Chart.js** - Visualize speaker timelines, word frequency, sentiment
- **Monaco editor** - Professional transcript editing
- **Rich HTML** - Formatted transcript display with speaker colors
- **Verdict:** Maximum "wow factor" - DEFER TO POST-V1

#### Phase 9 (Docs/Tests)
```
Bang: ★★☆☆☆  |  Investment: 2-3 hours
```
- Production polish, not new features
- **Verdict:** Do after Speech Studio ships

### Recommended Path

**Complete through Phase 7, then build Speech Studio.**

```
Phase 6.75 remaining → Phase 7 → Speech Studio v1 → Phase 8 (future)
     2-3 hours         4-6 hours     6-10 hours
```

**Total investment before Speech Studio:** ~0.5 day
**Speech Studio v1 build time:** ~0.5 day
**Total to working application:** ~1 day

### Why Phase 7 Before Speech Studio?

1. **Waveform display** - A speech app without audio visualization feels incomplete
2. **Visual polish** - Gradients and shadows signal "professional tool"
3. **Progress feedback** - Cairo enables custom progress indicators during long transcriptions
4. **Proving ground** - Speech Studio validates simple_vision before Tier 3 features

### What Phase 8 Adds (Post-V1)

Save for Speech Studio v2:
- Chart.js for analytics dashboards
- Monaco for advanced transcript editing
- WebView for rich formatted output

---

## Part 8: Project Timeline

Based on 15K LOC/day productivity baseline:

| Phase | Task | Duration | Cumulative |
|-------|------|----------|------------|
| 1 | Complete simple_vision 6.75 remaining | 2-3 hours | 3 hours |
| 2 | Complete simple_vision Phase 7 | 4-6 hours | ~0.5 day |
| 3 | Create simple_repository library | 3-4 hours | ~0.75 day |
| 4 | JSON spec protocol + use-cases | 2-3 hours | ~1 day |
| 5 | Speech Studio implementation | 6-8 hours | ~1.5 days |
| 6 | Testing and polish | 2-3 hours | ~1.5 days |

**Conservative estimate:** 1.5-2 days to working Speech Studio v1

---

## Appendix: December 28-29 Productivity Data

For calibration purposes, here is the actual output from the 48-hour sprint that produced simple_vision and simple_speech:

### Projects Touched: 3

| Project | Commits | Status |
|---------|---------|--------|
| simple_vision | 1 | NEW - Initial commit |
| simple_speech | 7 | Phases 0-7 complete |
| simple_sql | 1 | Enhancement |

### Classes Created: 83

| Project | Classes |
|---------|---------|
| simple_vision | 60 (.e files: 52 src + 7 demos + 1 test) |
| simple_speech | 23 |
| **Total** | **83 classes** |

### Lines of Code (GitHub insertions)

| Project | Added | Deleted | Net |
|---------|-------|---------|-----|
| simple_vision | +16,121 | 0 | +16,121 |
| simple_speech | +13,707 | -665 | +13,042 |
| simple_sql | +1,460 | -9 | +1,451 |
| **Total** | **+31,288** | -674 | **+30,614** |

### Performance Metrics

| Metric | Value |
|--------|-------|
| 2-day total | 30,614 LOC |
| Daily average | 15,307 LOC/day |
| Normal rate | 5,000 LOC/day |
| **Performance** | **3.06x normal** |

---

*This specification supersedes all previous notes and serves as the authoritative project definition for Speech Studio.*
