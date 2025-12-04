# simple_showcase: Complete Content & Visual Script
## The Full Blueprint for Building SSC

**Created:** December 4, 2025
**Authors:** Larry Rix and Claude (Anthropic)
**Purpose:** Complete content, copy, and visual direction for simple_showcase website

---

## Table of Contents

1. [Design Philosophy](#design-philosophy)
2. [Visual System](#visual-system)
3. [Landing Page: The Journey](#landing-page-the-journey)
   - [Hero Section](#hero-section)
   - [Section 1: Recognition](#section-1-recognition-the-before)
   - [Section 2: The Shift](#section-2-the-shift-ai-arrived)
   - [Section 3: The Problem](#section-3-the-problem-ai-alone-isnt-enough)
   - [Section 4: The Unlock](#section-4-the-unlock-contracts-verify-ai)
   - [Section 5: The Evidence](#section-5-the-evidence-look-what-got-built)
   - [Section 6: The Revelation](#section-6-the-revelation-eiffel-was-built-for-this)
   - [Section 7: The Workflow](#section-7-the-workflow-human--ai-partnership)
   - [Section 8: The Invitation](#section-8-the-invitation-come-build)
4. [Sub-Pages](#sub-pages)
   - [Get Started](#sub-page-get-started)
   - [The Project Portfolio](#sub-page-the-project-portfolio)
   - [How Design by Contract Works](#sub-page-how-design-by-contract-works)
   - [The Human-AI Workflow](#sub-page-the-human-ai-workflow)
   - [The Competitive Analysis](#sub-page-the-competitive-analysis)
   - [The Business Case](#sub-page-the-business-case)
   - [Why Eiffel?](#sub-page-why-eiffel)
   - [From Probable to Provable](#sub-page-from-probable-to-provable)
   - [The Real Cost of the Old Way](#sub-page-the-real-cost-of-the-old-way)
   - [What AI Actually Changes](#sub-page-what-ai-actually-changes)
5. [Technical Implementation Notes](#technical-implementation-notes)
6. [Build Priority](#build-priority)

---

## Design Philosophy

### Core Principles

1. **Evidence over argument** — Show, don't tell. Let artifacts speak.
2. **Invitation, not attack** — The reader is a potential ally, not an enemy.
3. **Progressive revelation** — AI → DBC → Eiffel, in that order.
4. **Premium but not corporate** — Sophisticated, confident, human.
5. **Self-demonstrating** — The site itself is proof of the paradigm.

### The Emotional Journey

```
HERO         → "Wait, what? How is that possible?"
RECOGNITION  → "That's my life, that frustration"
SHIFT        → "Yes, AI is exciting, I feel that too"
PROBLEM      → "I've sensed that unease about AI"
UNLOCK       → "Oh, that's clever — the code verifies itself"
EVIDENCE     → "This is real, not theoretical"
REVELATION   → "This technology exists? For 40 years?"
WORKFLOW     → "I could do this, I could be the pilot"
INVITATION   → "I want to try"
```

### The Tribe Message

**Old tribe (implicit, not attacked):** Debating, waiting, managing dependencies, hoping code works.

**New tribe (explicit invitation):** Building, shipping, verifying, confidence.

> "While others were debating, we were building."

---

## Visual System

### Color Palette

| Role | Color | Hex | Usage |
|------|-------|-----|-------|
| **Primary Dark** | Deep Charcoal | `#0c0b0b` | Main background |
| **Primary Light** | Off-White | `#fdfcfa` | Primary text |
| **Accent Calm** | Deep Teal | `#0d4f4f` | Unlock/solution sections |
| **Accent Warning** | Deep Amber | `#4f2d0d` | Problem sections |
| **Accent Energy** | Deep Purple | `#2d0d4f` | Shift/excitement sections |
| **Accent Evidence** | Deep Navy | `#0d1a4f` | Evidence/proof sections |
| **Code Background** | Near-Black | `#1a1a1a` | Code blocks |
| **Code Text** | Soft Green | `#8fdf8f` | Code syntax |

### Typography

| Element | Treatment |
|---------|-----------|
| **Hero Headline** | 8-10vw, light weight (300), letter-spacing -0.02em |
| **Section Headlines** | 5-6vw, medium weight (500) |
| **Body Text** | 1.2-1.5rem, line-height 1.7, weight 400 |
| **Section Labels** | 0.75rem, uppercase, letter-spacing 0.2em, muted color |
| **Code** | Monospace, 0.9rem, line-height 1.5 |
| **Stats/Numbers** | Tabular figures, bold weight |

### Animation Principles

| Type | Timing | Easing |
|------|--------|--------|
| **Text reveals** | 0.6-0.8s | cubic-bezier(0.25, 0.46, 0.45, 0.94) |
| **Card entrances** | 0.4-0.6s | ease-out |
| **Number counting** | 1.5-2s | ease-out |
| **Background transitions** | 1.2s | linear |
| **Hover states** | 0.2s | ease |
| **Scroll progress** | Tied to scroll position | linear |

### Texture & Depth

- **Noise overlay:** 2-3% opacity, animated at 0.2s loop for subtle movement
- **Gradient layers:** Radial gradients for spotlight effects
- **Parallax:** 3 layers — background (slow), content (normal), foreground elements (faster)
- **Shadows:** Minimal, soft, used only for floating cards

### Scroll Behavior

- **Scroll-snap:** `scroll-snap-type: y mandatory` on main container
- **Sections:** `scroll-snap-align: start`, full viewport height
- **Smooth scroll:** Lenis or similar for premium feel
- **Progress indicator:** Optional — subtle line showing journey progress

---

## Landing Page: The Journey

---

### HERO SECTION

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│                                                                            │
│                                                                            │
│                    While others satisficed,                                │
│                       we were building.                                    │
│                                                                            │
│                                                                            │
│           11 libraries. 900+ tests. 10 days. One person.                   │
│           AI-assisted. Contract-verified. Production-ready.                │
│                                                                            │
│                                                                            │
│                                                                            │
│                          Scroll to see how                                 │
│                               ↓                                            │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
FADE IN:

[0.0s] Screen is pure deep charcoal (#0c0b0b)
       Subtle noise texture visible, slowly animating

[0.3s] First word "While" fades in from 0 opacity, drifts up 20px

[0.5s] "others" follows

[0.7s] "satisficed," appears

[1.0s] Second line begins: "we were building."
       This line is slightly brighter, more prominent

[1.5s] Pause. Let it land.

[2.0s] Subhead fades in as a single unit
       Numbers have subtle glow effect

[2.5s] Stats could have a very subtle pulse
       "11" "900+" "10" "One" — the human elements

[3.0s] Scroll indicator appears
       Gentle bounce animation, infinite loop

[IDLE] Subtle parallax on mouse movement
       Headline layers shift slightly with cursor position
       Noise texture continues its micro-animation

[ON SCROLL]
       Content begins to drift up and fade
       Next section background begins to appear
       Scroll indicator fades out
```

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| Headline | None (too large) | None |
| Subhead | Subtle brightness increase | None |
| Scroll indicator | Speeds up animation | Smooth-scrolls to Section 1 |

#### Technical Notes

```css
/* Hero section */
.hero {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  scroll-snap-align: start;
  position: relative;
  overflow: hidden;
}

/* Parallax container */
.hero-content {
  transform: translateY(calc(var(--scroll-progress) * -50px));
  opacity: calc(1 - var(--scroll-progress) * 2);
}
```

---

### SECTION 1: RECOGNITION (The Before)

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE OLD WAY                                                               │
│                                                                            │
│                                                                            │
│                       We've all been here.                                 │
│                                                                            │
│                                                                            │
│         The dependency update that broke production.                       │
│                                                                            │
│         The framework that changed everything. Again.                      │
│                                                                            │
│         The meeting about which library to use.                            │
│                                                                            │
│         The bug that "shouldn't have happened."                            │
│                                                                            │
│                                                                            │
│         Weeks spent managing tools instead of building.                    │
│         Months lost to decisions that didn't matter.                       │
│                                                                            │
│         This was just how software worked.                                 │
│                                                                            │
│                                                                            │
│                               → The real cost of the old way               │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
[SCROLL ENTER - 0%]
       Background transitions from pure charcoal to slightly warmer dark
       (#0c0b0b → #0f0c0a) — subtle shift suggesting age, rust, friction

[SCROLL 10%]
       Section label "THE OLD WAY" fades in
       Small, muted, uppercase — sets context without drama

[SCROLL 20%]
       "We've all been here." appears
       Large, centered, empathetic
       This is the invitation: "I see you. I've felt this too."

[SCROLL 30%]
       First pain point fades in from left:
       "The dependency update that broke production."
       Slight red/amber tint on "broke"

[SCROLL 40%]
       Second pain point:
       "The framework that changed everything. Again."
       "Again." has emphasis — slight pause before it appears

[SCROLL 50%]
       Third pain point:
       "The meeting about which library to use."

[SCROLL 60%]
       Fourth pain point:
       "The bug that 'shouldn't have happened.'"
       Quotes could flicker slightly — irony

[SCROLL 70%]
       Summary lines fade in together:
       "Weeks spent managing tools instead of building."
       "Months lost to decisions that didn't matter."

[SCROLL 80%]
       "This was just how software worked."
       Resignation in the typography — slightly dimmer

[SCROLL 90%]
       Link appears: "→ The real cost of the old way"
       Subtle arrow animation on idle

[SCROLL EXIT]
       All content drifts up and fades
       Background begins transitioning to next section's color
```

#### Background Treatment

Abstract visual elements suggesting friction/tangled complexity:
- Thin, tangled lines that slowly drift (like cables or dependencies)
- Or: A fragmented grid pattern, slightly broken
- These fade out as you scroll through, representing leaving the old way

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| Pain point lines | Subtle brightness increase | None |
| Link | Arrow extends right, underline appears | Navigate to sub-page |

---

### SECTION 2: THE SHIFT (AI Arrived)

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE SHIFT                                                                 │
│                                                                            │
│                                                                            │
│                   Then AI changed everything.                              │
│                                                                            │
│                                                                            │
│                Suddenly, code appeared in seconds.                         │
│                     Boilerplate vanished.                                  │
│                 Prototypes materialized overnight.                         │
│                                                                            │
│                                                                            │
│                 The productivity explosion was real.                       │
│                Everyone felt it. Everyone invested.                        │
│                                                                            │
│                                                                            │
│                          ┌─────────────────┐                               │
│                          │  $200 billion   │                               │
│                          │  AI funding     │                               │
│                          │  in 2024        │                               │
│                          └─────────────────┘                               │
│                                                                            │
│                          ┌─────────────────┐                               │
│                          │     84%         │                               │
│                          │  of developers  │                               │
│                          │  use AI daily   │                               │
│                          └─────────────────┘                               │
│                                                                            │
│                                                                            │
│                     The excitement was justified.                          │
│                                                                            │
│                       But something was wrong.                             │
│                                                                            │
│                                                                            │
│                              → What AI actually changes                    │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
[SCROLL ENTER - 0%]
       Background shifts to energetic dark purple (#1a0d2a)
       Subtle particle effect begins — dots of light drifting upward
       Energy. Excitement. Something new.

[SCROLL 10%]
       "THE SHIFT" label appears

[SCROLL 15%]
       "Then AI changed everything."
       Large, bright, possibly with subtle glow
       This is the turning point in the story

[SCROLL 25%]
       Rapid sequence of positive statements:
       "Suddenly, code appeared in seconds." — appears
       [0.3s pause]
       "Boilerplate vanished." — appears
       [0.3s pause]
       "Prototypes materialized overnight." — appears

       These come faster than Section 1 — momentum building

[SCROLL 40%]
       "The productivity explosion was real."
       "Everyone felt it. Everyone invested."

       Background particles increase slightly

[SCROLL 50%]
       First stat card slides in from left:
       "$200 billion" — number counts up from 0
       "AI funding in 2024"

       Card has subtle glow, glass-morphism effect

[SCROLL 60%]
       Second stat card slides in from right:
       "84%" — number counts up from 0
       "of developers use AI daily"

[SCROLL 70%]
       "The excitement was justified."
       Bright, affirming

[SCROLL 80%]
       Pause. Particles slow.

       "But something was wrong."

       This line appears slower.
       Slightly dimmer. The turn.
       Background begins subtle shift toward warning colors.
       Particles begin to fade.

[SCROLL 90%]
       Link appears: "→ What AI actually changes"

[SCROLL EXIT]
       The energy drains from the section
       Transition to the weight of Section 3
```

#### Background Treatment

- Particle field: Small bright dots drifting upward (like ideas, possibilities)
- Gradient: Deep purple → brighter purple at peak excitement → dims at "something was wrong"
- The particles could spell out subtle code symbols or just be abstract light

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| Stat cards | Lift slightly (transform: translateY(-4px)), glow increases | None |
| Link | Arrow extends, underline | Navigate to sub-page |

---

### SECTION 3: THE PROBLEM (AI Alone Isn't Enough)

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE PROBLEM                                                               │
│                                                                            │
│                                                                            │
│                     AI alone isn't enough.                                 │
│                                                                            │
│                                                                            │
│                 The code was fast. But was it correct?                     │
│                                                                            │
│                        The research is clear:                              │
│                                                                            │
│                                                                            │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                                                                      │  │
│  │  "Copilot users introduced 41% more bugs."                           │  │
│  │                                           — Uplevel, 2024            │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                            │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                                                                      │  │
│  │  "45% of LLM-generated code contained security flaws."               │  │
│  │                                           — Veracode, 2024           │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                            │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                                                                      │  │
│  │  "30-50% of AI code samples had critical vulnerabilities."           │  │
│  │                                           — Adnan et al., 2025       │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                            │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                                                                      │  │
│  │  "Developer trust in AI accuracy: 43% → 33%"                         │  │
│  │                                           — Stack Overflow, 2025     │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                            │
│                                                                            │
│              AI generates probable code. Fast, but not proven.             │
│                   At scale, probable isn't good enough.                    │
│                                                                            │
│                                                                            │
│                    ┌─────────────────────────────────┐                     │
│                    │                                 │                     │
│                    │  99.9% correct                  │                     │
│                    │       ×                         │                     │
│                    │  1,000 modules                  │                     │
│                    │       =                         │                     │
│                    │  37% system reliability         │                     │
│                    │                                 │                     │
│                    └─────────────────────────────────┘                     │
│                                                                            │
│                                                                            │
│                  The industry is shipping faster than ever.                │
│                     And breaking more than ever.                           │
│                                                                            │
│                                                                            │
│                            → From probable to provable                     │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
[SCROLL ENTER - 0%]
       Background shifts to deep warning tone
       Not red (too aggressive) — deep amber-black (#1a0f05)
       Particles from Section 2 are gone
       Mood: serious, sobering

[SCROLL 5%]
       "THE PROBLEM" label appears

[SCROLL 10%]
       "AI alone isn't enough."
       Large, stark, undeniable
       No animation flourish — just the statement

[SCROLL 18%]
       "The code was fast. But was it correct?"
       The question hangs.

[SCROLL 22%]
       "The research is clear:"
       This introduces the evidence

[SCROLL 28%]
       First evidence card slides in:
       "Copilot users introduced 41% more bugs."
       — Uplevel, 2024

       Card has left border accent (amber)
       Source is clearly attributed

[SCROLL 36%]
       Second evidence card:
       "45% of LLM-generated code contained security flaws."
       — Veracode, 2024

[SCROLL 44%]
       Third evidence card:
       "30-50% of AI code samples had critical vulnerabilities."
       — Adnan et al., 2025

[SCROLL 52%]
       Fourth evidence card:
       "Developer trust in AI accuracy: 43% → 33%"
       — Stack Overflow, 2025

       The arrow animates: 43 counts down to 33
       Trust is falling.

[SCROLL 62%]
       "AI generates probable code. Fast, but not proven."
       "At scale, probable isn't good enough."

[SCROLL 72%]
       The math visualization appears:

       "99.9% correct" appears
       [pause]
       "×" appears
       "1,000 modules" appears
       [pause]
       "=" appears
       [longer pause for impact]
       "37% system reliability" appears — this is the gut punch

       Numbers could animate/calculate
       37% should feel shockingly low

[SCROLL 85%]
       "The industry is shipping faster than ever."
       "And breaking more than ever."

       Parallel structure. The irony lands.

[SCROLL 92%]
       Link: "→ From probable to provable"

[SCROLL EXIT]
       The weight lifts
       Background begins transition to solution (teal)
       A sense of "but there's an answer..."
```

#### Background Treatment

- Deep amber-black gradient, darker at edges
- No particles — stillness, weight
- Subtle vignette effect drawing focus to center
- Could have very faint "broken code" pattern in background (illegible, abstract)

#### Evidence Cards Design

```
┌────────────────────────────────────────────────────────┐
│ ▌                                                      │
│ ▌  "Quote text here."                                  │
│ ▌                                                      │
│ ▌                           — Source, Year             │
│ ▌                                                      │
└────────────────────────────────────────────────────────┘

- Left border: 3px solid amber accent
- Background: slightly lighter than section background
- Subtle shadow for lift
- Attribution right-aligned, smaller, muted
```

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| Evidence cards | Slight lift, border brightens | Could link to source study |
| Math visualization | Numbers could re-animate on hover | None |
| Link | Standard arrow extend, underline | Navigate to sub-page |

---

### SECTION 4: THE UNLOCK (Contracts Verify AI)

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE UNLOCK                                                                │
│                                                                            │
│                                                                            │
│                  What if the code verified itself?                         │
│                                                                            │
│                                                                            │
│                Every feature states what must be true.                     │
│                Before it runs. After it runs. Always.                      │
│                                                                            │
│                       The runtime enforces it.                             │
│                      AI writes. Contracts verify.                          │
│                   Errors caught instantly. Automatically.                  │
│                                                                            │
│                                                                            │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                                                                      │  │
│  │  add_item (a_item: ITEM)                                             │  │
│  │      require                                                         │  │
│  │          item_exists: a_item /= Void                                 │  │
│  │      do                                                              │  │
│  │          items.extend (a_item)                                       │  │
│  │      ensure                                                          │  │
│  │          item_added: items.has (a_item)                              │  │
│  │          count_increased: items.count = old items.count + 1          │  │
│  │      end                                                             │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                            │
│               ▲                    ▲                    ▲                  │
│               │                    │                    │                  │
│          PRECONDITION         IMPLEMENTATION       POSTCONDITION           │
│          "What must be        "What happens"       "What is now            │
│           true before"                              guaranteed"            │
│                                                                            │
│                                                                            │
│                       Not "hope it works."                                 │
│                Not "the tests probably cover it."                          │
│                                                                            │
│              The code proves itself correct. Every time.                   │
│                                                                            │
│                                                                            │
│                      This is Design by Contract.                           │
│             And it changes everything about AI-assisted development.       │
│                                                                            │
│                                                                            │
│                          → How Design by Contract works                    │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
[SCROLL ENTER - 0%]
       Background transitions to solution color
       Deep teal (#0a2a2a) — calm, confident, clarity
       The mood lifts. Relief begins.

       Subtle animated gradient: darker at edges, lighter at center
       Like a light turning on, illuminating the solution

[SCROLL 5%]
       "THE UNLOCK" label appears

[SCROLL 10%]
       "What if the code verified itself?"
       This is the pivot question
       Slightly brighter than surrounding text
       Could have subtle glow or highlight

[SCROLL 20%]
       Explanatory lines appear in sequence:
       "Every feature states what must be true."
       [beat]
       "Before it runs. After it runs. Always."

[SCROLL 30%]
       "The runtime enforces it."
       "AI writes. Contracts verify."
       "Errors caught instantly. Automatically."

       These stack, building the picture

[SCROLL 42%]
       The code block begins to appear

       First: the container fades in (the frame)
       Then: code appears line by line

       Line 1: "add_item (a_item: ITEM)" — normal
       Line 2: "    require" — highlighted, label "PRECONDITION" appears
       Line 3: "        item_exists: a_item /= Void"
       Line 4: "    do" — normal
       Line 5: "        items.extend (a_item)" — labeled "IMPLEMENTATION"
       Line 6: "    ensure" — highlighted, label "POSTCONDITION" appears
       Line 7: "        item_added: items.has (a_item)"
       Line 8: "        count_increased: items.count = old items.count + 1"
       Line 9: "    end"

       Labels appear below the code block with connector lines

[SCROLL 65%]
       Code block is fully visible
       Labels are visible

       Pause to let it sink in

[SCROLL 72%]
       "Not 'hope it works.'"
       "Not 'the tests probably cover it.'"

       These dismiss the old way

[SCROLL 80%]
       "The code proves itself correct. Every time."

       This is the payoff. Slightly larger. Confident.

[SCROLL 88%]
       "This is Design by Contract."
       "And it changes everything about AI-assisted development."

[SCROLL 94%]
       Link: "→ How Design by Contract works"

[SCROLL EXIT]
       Sense of clarity maintained
       Transition to evidence section
```

#### Code Block Design

```css
.code-block {
  background: #1a1a1a;
  border: 1px solid #333;
  border-radius: 8px;
  padding: 24px;
  font-family: 'JetBrains Mono', 'Fira Code', monospace;
  font-size: 0.95rem;
  line-height: 1.6;
  overflow-x: auto;
}

.code-keyword { color: #c792ea; }     /* require, do, ensure, end */
.code-identifier { color: #82aaff; }  /* add_item, a_item, items */
.code-comment { color: #7a8599; }     /* item_exists:, item_added:, etc. */
.code-operator { color: #89ddff; }    /* /=, = */
.code-type { color: #ffcb6b; }        /* ITEM */
```

#### Code Animation Sequence

The code block could animate to show the three parts:

1. **State 1:** Only precondition highlighted
   - "This must be true BEFORE we start"

2. **State 2:** Only implementation highlighted
   - "This is what actually happens"

3. **State 3:** Only postcondition highlighted
   - "This is GUARANTEED after"

4. **State 4:** All visible
   - "Together, the code verifies itself"

This could be scroll-driven or auto-animated.

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| Code block | Subtle border glow | Could toggle highlighting states |
| Labels | Highlight corresponding code section | None |
| Link | Standard treatment | Navigate to sub-page |

---

### SECTION 5: THE EVIDENCE (Look What Got Built)

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE EVIDENCE                                                              │
│                                                                            │
│                                                                            │
│                  We didn't just theorize. We built.                        │
│                                                                            │
│                                                                            │
│          One person. 10 days. AI-assisted. Contract-verified.              │
│                                                                            │
│                                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                                                                     │   │
│  │  simple_json                                                        │   │
│  │  JSON parsing and serialization                          215 tests  │   │
│  │                                                                     │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │                                                                     │   │
│  │  simple_sql                                                         │   │
│  │  Type-safe SQL query building                            339 tests  │   │
│  │                                                                     │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │                                                                     │   │
│  │  simple_web                                                         │   │
│  │  HTTP server and client with middleware                   95 tests  │   │
│  │                                                                     │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │                                                                     │   │
│  │  simple_htmx                                                        │   │
│  │  Fluent HTML/HTMX attribute builder                       40 tests  │   │
│  │                                                                     │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │                                                                     │   │
│  │  simple_alpine                                                      │   │
│  │  Alpine.js directive builder                             103 tests  │   │
│  │                                                                     │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │                                                                     │   │
│  │  + 6 more libraries                                                 │   │
│  │  simple_ci, simple_process, simple_randomizer...                    │   │
│  │                                                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                            │
│                                                                            │
│                       Every library on GitHub.                             │
│                        Every test passing.                                 │
│                       Run them yourself.                                   │
│                                                                            │
│                                                                            │
│               ┌─────────────────────────────────────────┐                  │
│               │                                         │                  │
│               │   This site was built the same way.     │                  │
│               │                                         │                  │
│               └─────────────────────────────────────────┘                  │
│                                                                            │
│                                                                            │
│                            → The project portfolio                         │
│                            → The competitive analysis                      │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
[SCROLL ENTER - 0%]
       Background: confident dark navy (#0d1428)
       The mood is "receipts on the table"
       Subtle grid pattern in background (suggesting structure, organization)

[SCROLL 5%]
       "THE EVIDENCE" label appears

[SCROLL 12%]
       "We didn't just theorize. We built."
       Direct, confident, undeniable

[SCROLL 20%]
       "One person. 10 days. AI-assisted. Contract-verified."
       The parameters of the proof

[SCROLL 28%]
       Portfolio list begins to appear
       Each item slides in from bottom, staggered 0.15s apart

       Card 1: simple_json
       - Name fades in
       - Description fades in
       - "215" counts up from 0, then "tests" appears

[SCROLL 35%]
       Card 2: simple_sql
       - Same animation pattern
       - "339" counts up

[SCROLL 42%]
       Card 3: simple_web
       - "95" counts up

[SCROLL 49%]
       Card 4: simple_htmx
       - "40" counts up

[SCROLL 56%]
       Card 5: simple_alpine
       - "103" counts up

[SCROLL 63%]
       Card 6: "+ 6 more libraries"
       - Different treatment — expandable hint

[SCROLL 72%]
       Summary statements:
       "Every library on GitHub."
       "Every test passing."
       "Run them yourself."

       "Run them yourself" is the invitation — slightly emphasized

[SCROLL 82%]
       Self-reference callout:
       "This site was built the same way."

       This is in a distinct container — subtle border, slight glow
       The proof is the thing they're looking at

[SCROLL 90%]
       Two links appear:
       "→ The project portfolio"
       "→ The competitive analysis"

[SCROLL EXIT]
       Cards could drift up slightly as we leave
       Transition to revelation section
```

#### Portfolio Card Design

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  simple_json                                  215 tests │
│  JSON parsing and serialization                      → │
│                                                         │
└─────────────────────────────────────────────────────────┘

- Full width or card grid (responsive)
- Name: bold, left-aligned
- Description: smaller, muted
- Test count: right-aligned, numbers prominent
- Arrow: indicates clickable, appears on hover
- Hover: slight lift, border glow, arrow extends
```

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| Portfolio cards | Lift, glow, arrow appears | Navigate to GitHub repo |
| "This site" callout | Border pulse | Navigate to site's own repo |
| Links | Standard treatment | Navigate to sub-pages |

#### Total Animation

When fully scrolled through, a subtle "total" could appear:

```
                    ────────────────────
                    900+ tests passing
                    11 libraries shipped
                    ────────────────────
```

---

### SECTION 6: THE REVELATION (Eiffel Was Built For This)

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE TOOL                                                                  │
│                                                                            │
│                                                                            │
│                  One language was built for this.                          │
│                                                                            │
│                                                                            │
│                    Design by Contract isn't a library.                     │
│                       It isn't a plugin or a pattern.                      │
│                                                                            │
│                   It's built into the language itself.                     │
│                      For 40 years. Refined. Proven.                        │
│                                                                            │
│                                                                            │
│                                                                            │
│                              ┌───────────┐                                 │
│                              │           │                                 │
│                              │  Eiffel   │                                 │
│                              │           │                                 │
│                              └───────────┘                                 │
│                                                                            │
│                                                                            │
│                                                                            │
│                        Created by Bertrand Meyer.                          │
│             The language where contracts are first-class citizens.         │
│                  Where correctness isn't an afterthought.                  │
│                                                                            │
│                                                                            │
│           Now paired with AI, the combination unlocks something new:       │
│                                                                            │
│                         Speed without sacrifice.                           │
│                         Productivity with proof.                           │
│                        Building with confidence.                           │
│                                                                            │
│                                                                            │
│                                → Why Eiffel?                               │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
[SCROLL ENTER - 0%]
       Background: prestigious dark navy/charcoal (#0d1020)
       This is the reveal — the moment of naming
       Subtle radial gradient: lighter at center, drawing focus

[SCROLL 5%]
       "THE TOOL" label appears

[SCROLL 12%]
       "One language was built for this."
       The setup for the reveal

[SCROLL 22%]
       Clarifying statements:
       "Design by Contract isn't a library."
       "It isn't a plugin or a pattern."

       Dismissing what they might assume

[SCROLL 32%]
       "It's built into the language itself."
       "For 40 years. Refined. Proven."

       "40 years" could have emphasis — this isn't new, it's mature

[SCROLL 45%]
       THE REVEAL:

       Background dims slightly around the edges
       Center spotlight effect intensifies

       The word "Eiffel" appears.

       Just the word. Centered. Large.

       It sits alone for a moment.

       This is the name. The answer. The tool.

       Optional: Very subtle reference to Eiffel Tower shape
       in abstract behind the word (architectural, elegant)

[SCROLL 55%]
       Context appears below:
       "Created by Bertrand Meyer."

       Attribution matters. This has pedigree.

[SCROLL 62%]
       "The language where contracts are first-class citizens."
       "Where correctness isn't an afterthought."

[SCROLL 72%]
       "Now paired with AI, the combination unlocks something new:"

       Transition line — past meets present

[SCROLL 80%]
       Three payoffs appear, staggered:
       "Speed without sacrifice."
       [beat]
       "Productivity with proof."
       [beat]
       "Building with confidence."

       These are the outcomes. The benefits.

[SCROLL 92%]
       Link: "→ Why Eiffel?"

[SCROLL EXIT]
       The revelation complete
       Transition to workflow section
```

#### "Eiffel" Reveal Design

The word "Eiffel" is the visual centerpiece of this section:

- Typography: Large (6-8vw), elegant weight (not too bold)
- Could use a serif or distinctive font for this word only
- Subtle text-shadow or glow
- Background: Radial gradient spotlight effect
- Animation: Fade in with slight scale (0.95 → 1.0)

Optional abstract Eiffel Tower reference:
- Very subtle line drawing behind text
- Or: Triangular/architectural shapes
- Should NOT be literal or cheesy — abstract, sophisticated

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| "Eiffel" word | Very subtle glow increase | Could link to eiffel.org |
| "Bertrand Meyer" | Underline | Could link to his site |
| Link | Standard treatment | Navigate to sub-page |

---

### SECTION 7: THE WORKFLOW (Human + AI Partnership)

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE WORKFLOW                                                              │
│                                                                            │
│                                                                            │
│                 You're the pilot. AI is your co-pilot.                     │
│                                                                            │
│                                                                            │
│                      Sometimes it's "my airplane."                         │
│                      Sometimes it's "your airplane."                       │
│                        You're always in command.                           │
│                                                                            │
│                                                                            │
│        ┌─────────────────────────────────────────────────────────┐         │
│        │                                                         │         │
│        │                    THE COLLABORATION                    │         │
│        │                                                         │         │
│        │  ┌─────────┐                           ┌─────────┐      │         │
│        │  │  HUMAN  │                           │   AI    │      │         │
│        │  └────┬────┘                           └────┬────┘      │         │
│        │       │                                     │           │         │
│        │       │  1. "Build an HTTP server          │           │         │
│        │       │      with middleware."             │           │         │
│        │       │ ─────────────────────────────────► │           │         │
│        │       │                                     │           │         │
│        │       │  2. "I'll use agent-based          │           │         │
│        │       │      routing with pipelines."      │           │         │
│        │       │ ◄───────────────────────────────── │           │         │
│        │       │                                     │           │         │
│        │       │  3. "Use our testing base          │           │         │
│        │       │      class, not the default."      │           │         │
│        │       │ ─────────────────────────────────► │           │         │
│        │       │                                     │           │         │
│        │       │  4. Code, tests, documentation     │           │         │
│        │       │ ◄───────────────────────────────── │           │         │
│        │       │                                     │           │         │
│        │       │  5. Contracts verify               │           │         │
│        │       │     automatically                  │           │         │
│        │       │          ✓                         │           │         │
│        │       │                                     │           │         │
│        │       │  6. Ship with confidence           │           │         │
│        │       │                                     │           │         │
│        └───────┴─────────────────────────────────────┴───────────┘         │
│                                                                            │
│                                                                            │
│                     The AI learns your patterns.                           │
│                   The contracts catch its mistakes.                        │
│             You build faster than ever — with proof it works.              │
│                                                                            │
│                                                                            │
│                        → The human-AI workflow in detail                   │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
[SCROLL ENTER - 0%]
       Background: balanced, collaborative tone
       Two-tone gradient: left side slightly different from right
       Suggesting partnership, two parties working together
       Color: deep slate blue (#0f1520)

[SCROLL 5%]
       "THE WORKFLOW" label appears

[SCROLL 12%]
       "You're the pilot. AI is your co-pilot."
       This is the core metaphor

[SCROLL 22%]
       "Sometimes it's 'my airplane.'"
       "Sometimes it's 'your airplane.'"
       "You're always in command."

       These appear with slight alternating indentation
       Left, right, center — like a conversation

[SCROLL 35%]
       The collaboration diagram begins to build:

       First: The container appears

[SCROLL 40%]
       "HUMAN" and "AI" boxes appear on opposite sides

[SCROLL 45%]
       Step 1 arrow animates: Human → AI
       "Build an HTTP server with middleware."
       Human gives direction

[SCROLL 50%]
       Step 2 arrow animates: AI → Human
       "I'll use agent-based routing with pipelines."
       AI proposes

[SCROLL 55%]
       Step 3 arrow animates: Human → AI
       "Use our testing base class, not the default."
       Human corrects/guides

[SCROLL 60%]
       Step 4 arrow animates: AI → Human
       "Code, tests, documentation"
       AI delivers

[SCROLL 65%]
       Step 5: Verification
       Checkmark appears with subtle celebration animation
       "Contracts verify automatically"

[SCROLL 70%]
       Step 6: Ship
       "Ship with confidence"

[SCROLL 80%]
       Summary statements:
       "The AI learns your patterns."
       "The contracts catch its mistakes."
       "You build faster than ever — with proof it works."

[SCROLL 92%]
       Link: "→ The human-AI workflow in detail"

[SCROLL EXIT]
       Diagram could collapse/minimize
       Transition to invitation
```

#### Diagram Animation Details

The collaboration diagram is the visual hero:

- Arrows animate along their path (not just appear)
- Text in each step fades in as arrow completes
- Could use dotted lines for arrows to suggest ongoing conversation
- Human side: slightly blue tint
- AI side: slightly purple/magenta tint
- Verification step: green checkmark, subtle pulse

Alternative: Instead of static diagram, could be a vertical timeline with alternating sides (left = human, right = AI).

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| Diagram steps | Step highlights, shows full text if truncated | None |
| "HUMAN" / "AI" boxes | Subtle glow | None |
| Link | Standard treatment | Navigate to sub-page |

---

### SECTION 8: THE INVITATION (Come Build)

#### Copy

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  YOUR MOVE                                                                 │
│                                                                            │
│                                                                            │
│                         Ready to build?                                    │
│                                                                            │
│                                                                            │
│                  This isn't about switching tribes.                        │
│           It's about building faster with more confidence.                 │
│                                                                            │
│                                                                            │
│                          Where do you start?                               │
│                                                                            │
│                                                                            │
│     ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐        │
│     │                  │ │                  │ │                  │        │
│     │       CTO        │ │    DEVELOPER     │ │     CURIOUS      │        │
│     │                  │ │                  │ │                  │        │
│     │   Evaluating     │ │    Ready to      │ │    Want to       │        │
│     │   risk, cost,    │ │    try something │ │    understand    │        │
│     │   and capability │ │    new           │ │    the approach  │        │
│     │                  │ │                  │ │                  │        │
│     │  → The Business  │ │  → Get Started   │ │  → Read the      │        │
│     │    Case          │ │                  │ │    Paper         │        │
│     │                  │ │                  │ │                  │        │
│     └──────────────────┘ └──────────────────┘ └──────────────────┘        │
│                                                                            │
│                                                                            │
│  ───────────────────────────────────────────────────────────────────────── │
│                                                                            │
│                                                                            │
│                    This site was built with Eiffel + AI.                   │
│                         The code is on GitHub.                             │
│                                                                            │
│                                                                            │
│                         Questions? Let's talk.                             │
│                           → Contact                                        │
│                                                                            │
│                                                                            │
│  ───────────────────────────────────────────────────────────────────────── │
│                                                                            │
│                                                                            │
│                               simple_showcase                              │
│                                    2025                                    │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Script

```
[SCROLL ENTER - 0%]
       Background: welcoming, open
       Slightly warmer than previous sections
       Could have subtle gradient toward light at bottom
       Suggesting dawn, possibility, openness
       Color: deep charcoal with warm undertone (#12100f)

[SCROLL 5%]
       "YOUR MOVE" label appears
       Different tone than other labels — more personal

[SCROLL 15%]
       "Ready to build?"
       Large, inviting, not demanding
       This is a question, not a command

[SCROLL 25%]
       "This isn't about switching tribes."
       "It's about building faster with more confidence."

       The reassurance. The reframe.
       You don't have to abandon your identity.

[SCROLL 35%]
       "Where do you start?"

       Sets up the three paths

[SCROLL 45%]
       Three cards appear, staggered from left to right:

       Card 1: CTO
       - Icon or illustration (abstract, professional)
       - "Evaluating risk, cost, and capability"
       - "→ The Business Case"

[SCROLL 50%]
       Card 2: DEVELOPER
       - Icon or illustration (code-related, energetic)
       - "Ready to try something new"
       - "→ Get Started"

[SCROLL 55%]
       Card 3: CURIOUS
       - Icon or illustration (lightbulb, question mark)
       - "Want to understand the approach"
       - "→ Read the Paper"

[SCROLL 65%]
       Divider line appears

[SCROLL 70%]
       Footer content:
       "This site was built with Eiffel + AI."
       "The code is on GitHub."

       The self-reference. The proof.

[SCROLL 80%]
       "Questions? Let's talk."
       "→ Contact"

       Or whatever CTA is appropriate

[SCROLL 90%]
       Site footer:
       "simple_showcase"
       "2025"

       Clean, minimal ending

[AT REST]
       The invitation stands open
       Cards remain interactive
       No pressure, just possibility
```

#### Card Design

```
┌────────────────────────────────────────┐
│                                        │
│              [ICON]                    │
│                                        │
│               CTO                      │
│                                        │
│        Evaluating risk, cost,          │
│          and capability                │
│                                        │
│                                        │
│         → The Business Case            │
│                                        │
└────────────────────────────────────────┘

- Generous padding
- Subtle border (1px, muted)
- Hover: lift (4-8px), border brightens, shadow appears
- Icon: abstract, not literal. Could be geometric shapes.
- Title: bold, centered
- Description: smaller, muted
- CTA: arrow that extends on hover
```

#### Interaction Details

| Element | Hover | Click |
|---------|-------|-------|
| Path cards | Significant lift, glow, arrow extends | Navigate to sub-page |
| "GitHub" | Underline | Link to repo |
| Contact link | Standard | Opens contact method |

---

## Sub-Pages

---

### SUB-PAGE: Get Started

**URL:** `/get-started`

**Purpose:** The gateway for developers ready to try. Must minimize friction.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  GET STARTED                                                               │
│                                                                            │
│  Start building with Eiffel + AI in under an hour.                         │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  WHAT YOU'LL NEED                                                          │
│                                                                            │
│  • A computer (Windows, macOS, or Linux)                                   │
│  • About 45 minutes                                                        │
│  • Willingness to try something different                                  │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  STEP 1: INSTALL EIFFELSTUDIO                                              │
│                                                                            │
│  EiffelStudio is the development environment for Eiffel.                   │
│  It's free for open-source development.                                    │
│                                                                            │
│  → Download EiffelStudio (eiffel.org)                                      │
│                                                                            │
│  [Expandable: Installation notes for Windows]                              │
│  [Expandable: Installation notes for macOS]                                │
│  [Expandable: Installation notes for Linux]                                │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  STEP 2: GET AI ASSISTANCE                                                 │
│                                                                            │
│  The workflow shines with AI assistance. We recommend:                     │
│                                                                            │
│  • Claude (Anthropic) — What we use. Excellent for Eiffel with             │
│    proper reference documentation.                                         │
│  • Claude Code CLI — Direct file access, runs compiler.                    │
│                                                                            │
│  Other AI assistants work, but may need more guidance.                     │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  STEP 3: GET THE REFERENCE DOCS                                            │
│                                                                            │
│  AI works best with context. Our reference documentation                   │
│  teaches AI the patterns and pitfalls.                                     │
│                                                                            │
│  → Clone: github.com/ljr1981/claude_eiffel_op_docs                         │
│                                                                            │
│  Key files to have AI read first:                                          │
│  • CLAUDE_CONTEXT.md — Eiffel fundamentals                                 │
│  • gotchas.md — Common pitfalls and solutions                              │
│  • patterns.md — Verified working code                                     │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  STEP 4: BUILD SOMETHING                                                   │
│                                                                            │
│  Start simple. Here's a first project:                                     │
│                                                                            │
│  "Create a simple calculator class with Design by Contract.                │
│   It should have add, subtract, multiply, divide.                          │
│   Each feature should have preconditions and postconditions."              │
│                                                                            │
│  Paste this to your AI assistant along with the reference docs.            │
│  Watch what happens.                                                       │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  WHAT TO EXPECT                                                            │
│                                                                            │
│  First hour:                                                               │
│  • Environment setup                                                       │
│  • First successful compile                                                │
│  • "Hello World" with a contract                                           │
│                                                                            │
│  First day:                                                                │
│  • Comfortable with basic syntax                                           │
│  • Understanding preconditions/postconditions                              │
│  • AI generating useful code                                               │
│                                                                            │
│  First week:                                                               │
│  • Building real features                                                  │
│  • Contracts catching bugs                                                 │
│  • Productivity increasing                                                 │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  EXPLORE THE LIBRARIES                                                     │
│                                                                            │
│  Want to see real code? Clone any of our libraries:                        │
│                                                                            │
│  • simple_json — Good starting point, clear patterns                       │
│  • simple_htmx — See fluent interface design                               │
│  • simple_alpine — See library layering                                    │
│                                                                            │
│  All have tests you can run to see contracts in action.                    │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  NEED HELP?                                                                │
│                                                                            │
│  • Eiffel community: eiffel.org/community                                  │
│  • Our repos: Issues welcome on GitHub                                     │
│  • Direct questions: [contact method]                                      │
│                                                                            │
│                                                                            │
│  → Back to home                                                            │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Treatment

- Clean, documentation-style layout
- Clear step numbering
- Expandable sections for platform-specific details
- Code blocks for commands
- Progress indication (Step 1 of 4, etc.)

---

### SUB-PAGE: The Project Portfolio

**URL:** `/portfolio`

**Purpose:** Evidence. All the libraries, stats, links.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE PROJECT PORTFOLIO                                                     │
│                                                                            │
│  Everything built. Everything on GitHub. Everything tested.                │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  SUMMARY                                                                   │
│                                                                            │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐           │
│  │     11     │  │    900+    │  │     10     │  │     1      │           │
│  │  libraries │  │   tests    │  │    days    │  │   person   │           │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘           │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  THE LIBRARIES                                                             │
│                                                                            │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                                                                      │  │
│  │  simple_json                                                         │  │
│  │                                                                      │  │
│  │  Full JSON parsing and serialization for Eiffel.                     │  │
│  │  Type-safe, contract-verified, production-ready.                     │  │
│  │                                                                      │  │
│  │  Lines: 11,400+     Tests: 215      Built: Nov 2025                  │  │
│  │                                                                      │  │
│  │  Features:                                                           │  │
│  │  • Parse JSON from strings or files                                  │  │
│  │  • Serialize Eiffel objects to JSON                                  │  │
│  │  • Type-safe accessors                                               │  │
│  │  • Friction-free helper patterns                                     │  │
│  │                                                                      │  │
│  │  → View on GitHub    → Run tests locally                             │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                            │
│  [Similar cards for each library...]                                       │
│                                                                            │
│  • simple_sql (339 tests)                                                  │
│  • simple_web (95 tests)                                                   │
│  • simple_htmx (40 tests)                                                  │
│  • simple_alpine (103 tests)                                               │
│  • simple_ci                                                               │
│  • simple_process (4 tests)                                                │
│  • simple_randomizer (27 tests)                                            │
│  • simple_gui_designer (10 tests)                                          │
│  • eiffel_sqlite_2025                                                      │
│  • reference_docs                                                          │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  TIMELINE                                                                  │
│                                                                            │
│  A visual timeline showing when each library was built,                    │
│  demonstrating the velocity.                                               │
│                                                                            │
│  Nov 11-14: simple_json (11,400 lines)                                     │
│  Nov 30 - Dec 1: simple_sql (17,200 lines)                                 │
│  Dec 2: simple_web server, simple_process, simple_randomizer               │
│  Dec 2: simple_ci                                                          │
│  Dec 2-3: simple_gui_designer                                              │
│  Dec 3: simple_htmx                                                        │
│  Dec 3: simple_alpine                                                      │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  RUN THE TESTS YOURSELF                                                    │
│                                                                            │
│  Don't take our word for it. Clone any repo, run the tests.                │
│                                                                            │
│  git clone https://github.com/ljr1981/simple_json                          │
│  cd simple_json                                                            │
│  # Follow README for test commands                                         │
│                                                                            │
│  Every test passes. Every contract verified.                               │
│                                                                            │
│  → Back to home                                                            │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Treatment

- Stat boxes with large numbers at top
- Library cards with expandable details
- Visual timeline (horizontal or vertical)
- Code blocks for clone/test commands
- Links to all GitHub repos

---

### SUB-PAGE: How Design by Contract Works

**URL:** `/design-by-contract`

**Purpose:** Technical explainer for developers who want to understand DBC.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  HOW DESIGN BY CONTRACT WORKS                                              │
│                                                                            │
│  The code tells you what it guarantees.                                    │
│  The runtime enforces those guarantees.                                    │
│  Automatically.                                                            │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  THE CORE IDEA                                                             │
│                                                                            │
│  Every feature (method) is a contract between caller and implementation:   │
│                                                                            │
│  • PRECONDITIONS: What must be true before the feature runs                │
│    "I require these conditions to do my job."                              │
│                                                                            │
│  • POSTCONDITIONS: What will be true after the feature runs                │
│    "I guarantee these results when I'm done."                              │
│                                                                            │
│  • INVARIANTS: What's always true about an object                          │
│    "This is always true about me, before and after every call."            │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  A REAL EXAMPLE                                                            │
│                                                                            │
│  Let's build a bank account:                                               │
│                                                                            │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                                                                      │  │
│  │  class BANK_ACCOUNT                                                  │  │
│  │                                                                      │  │
│  │  feature -- Access                                                   │  │
│  │                                                                      │  │
│  │      balance: DECIMAL                                                │  │
│  │          -- Current account balance                                  │  │
│  │                                                                      │  │
│  │  feature -- Operations                                               │  │
│  │                                                                      │  │
│  │      withdraw (amount: DECIMAL)                                      │  │
│  │          -- Withdraw amount from account                             │  │
│  │          require                                                     │  │
│  │              positive_amount: amount > 0                             │  │
│  │              sufficient_funds: amount <= balance                     │  │
│  │          do                                                          │  │
│  │              balance := balance - amount                             │  │
│  │          ensure                                                      │  │
│  │              balance_reduced: balance = old balance - amount         │  │
│  │          end                                                         │  │
│  │                                                                      │  │
│  │  invariant                                                           │  │
│  │      non_negative_balance: balance >= 0                              │  │
│  │                                                                      │  │
│  │  end                                                                 │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                            │
│  What this guarantees:                                                     │
│                                                                            │
│  ✓ You can't withdraw zero or negative amounts                             │
│  ✓ You can't withdraw more than you have                                   │
│  ✓ After withdrawal, balance is exactly what you expect                    │
│  ✓ Balance can never go negative, ever                                     │
│                                                                            │
│  Try to violate any of these? Runtime exception with exact location.       │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  WHAT HAPPENS WHEN AI WRITES CODE                                          │
│                                                                            │
│  Without contracts:                                                        │
│  • AI writes withdraw function                                             │
│  • Maybe it checks for sufficient funds, maybe not                         │
│  • Maybe there's a bug in the calculation                                  │
│  • You find out in production. Or you don't.                               │
│                                                                            │
│  With contracts:                                                           │
│  • AI writes withdraw function with contracts                              │
│  • First test run: "POSTCONDITION VIOLATION: balance_reduced"              │
│  • Bug found instantly. Exact location. Exact expectation.                 │
│  • AI fixes it. Contracts pass. Ship with confidence.                      │
│                                                                            │
│  The contracts are the specification.                                      │
│  The runtime is the verifier.                                              │
│  AI mistakes don't reach production.                                       │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  TYPES OF CONTRACTS                                                        │
│                                                                            │
│  [Detailed section on each type with examples...]                          │
│                                                                            │
│  • Preconditions (require)                                                 │
│  • Postconditions (ensure)                                                 │
│  • Class invariants (invariant)                                            │
│  • Loop variants and invariants                                            │
│  • Check assertions                                                        │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE "OLD" KEYWORD                                                         │
│                                                                            │
│  One of the most powerful features: comparing before and after.            │
│                                                                            │
│  ensure                                                                    │
│      count_increased: items.count = old items.count + 1                    │
│                                                                            │
│  "old items.count" captures the value BEFORE the feature ran.              │
│  This lets you specify exactly how state should change.                    │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  CONTRACTS VS. TESTS                                                       │
│                                                                            │
│  Tests: "Here are some examples that should work."                         │
│  Contracts: "Here is what must ALWAYS be true."                            │
│                                                                            │
│  Tests check specific cases.                                               │
│  Contracts check every execution.                                          │
│                                                                            │
│  Tests are valuable. Contracts are comprehensive.                          │
│  Use both. But contracts catch what tests miss.                            │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  WHY THIS MATTERS FOR AI                                                   │
│                                                                            │
│  AI generates statistically likely code.                                   │
│  "Likely" isn't "correct."                                                 │
│                                                                            │
│  Contracts turn "I hope this works" into "This is proven to work."         │
│                                                                            │
│  That's the unlock.                                                        │
│                                                                            │
│  → Back to home                                                            │
│  → Get Started                                                             │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Treatment

- Code examples as the heroes
- Annotations on code blocks explaining each part
- Comparison visualizations (with/without contracts)
- Clear section hierarchy
- Interactive code explorer if technically feasible

---

### SUB-PAGE: The Human-AI Workflow

**URL:** `/workflow`

**Purpose:** Detailed explanation of how human + AI collaboration actually works.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE HUMAN-AI WORKFLOW                                                     │
│                                                                            │
│  You're the pilot. Here's how the collaboration actually works.            │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  THE METAPHOR: YOUR AIRPLANE / MY AIRPLANE                                 │
│                                                                            │
│  In aviation, pilots explicitly transfer control:                          │
│  "Your airplane" — you take the controls                                   │
│  "My airplane" — I'm flying now                                            │
│                                                                            │
│  AI-assisted development works the same way:                               │
│                                                                            │
│  HUMAN ("My airplane"):                                                    │
│  • Setting direction and goals                                             │
│  • Architectural decisions                                                 │
│  • Validating proposals                                                    │
│  • Course corrections                                                      │
│  • Final approval                                                          │
│                                                                            │
│  AI ("Your airplane"):                                                     │
│  • Code generation                                                         │
│  • Pattern application                                                     │
│  • Documentation                                                           │
│  • Test creation                                                           │
│  • Bulk operations                                                         │
│                                                                            │
│  You're always pilot-in-command.                                           │
│  But workload shifts based on the task.                                    │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  A REAL SESSION                                                            │
│                                                                            │
│  Here's an actual exchange from building simple_htmx:                      │
│                                                                            │
│  [Transcript excerpt with annotations...]                                  │
│                                                                            │
│  HUMAN: "Build a fluent HTML builder for HTMX attributes."                 │
│                                                                            │
│  AI: "I'll create an HTMX_ELEMENT base class with:                         │
│       - Attribute storage via hash table                                   │
│       - Fluent methods returning 'like Current'                            │
│       - HTML escaping for security                                         │
│       - Separate methods for each HTMX attribute..."                       │
│                                                                            │
│  HUMAN: "Use TEST_SET_BASE for tests, not EQA_TEST_SET."                   │
│                                                                            │
│  AI: [Adjusts approach, generates code]                                    │
│                                                                            │
│  COMPILER: "VKCN(1): Function used as statement"                           │
│                                                                            │
│  AI: "Adding .do_nothing to fluent chains to satisfy compiler."            │
│                                                                            │
│  [Result: 34 classes, 40 tests, 4 hours]                                   │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE REFERENCE DOCUMENTATION SYSTEM                                        │
│                                                                            │
│  AI works best with context. We maintain living documentation:             │
│                                                                            │
│  • CLAUDE_CONTEXT.md — Eiffel fundamentals, session startup                │
│  • gotchas.md — Known pitfalls and solutions                               │
│  • patterns.md — Verified working code patterns                            │
│  • Project ROADMAPs — Project-specific context                             │
│                                                                            │
│  When AI reads these first, accuracy goes from ~60% to ~95%.               │
│                                                                            │
│  The documentation captures institutional knowledge.                       │
│  AI gets smarter with every session.                                       │
│  Mistakes don't repeat.                                                    │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE VERIFICATION LAYER                                                    │
│                                                                            │
│  AI writes code. But who checks AI?                                        │
│                                                                            │
│  Layer 1: Compiler — Type errors caught immediately                        │
│  Layer 2: Contracts — Runtime verification of correctness                  │
│  Layer 3: Tests — Specific scenarios validated                             │
│  Layer 4: Human — Final review and approval                                │
│                                                                            │
│  Most AI errors are caught at Layers 1-2.                                  │
│  Before human review.                                                      │
│  Before production.                                                        │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  WHAT MAKES IT WORK                                                        │
│                                                                            │
│  1. Clear direction — AI needs to know what you want                       │
│  2. Reference context — AI needs to know your patterns                     │
│  3. Rapid feedback — Compiler/contracts catch errors fast                  │
│  4. Iteration — Build incrementally, verify continuously                   │
│  5. Human judgment — You're always the decision-maker                      │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  PRODUCTIVITY RESULTS                                                      │
│                                                                            │
│  Traditional development: X months                                         │
│  AI-assisted with verification: X days                                     │
│                                                                            │
│  Measured multipliers: 40-80x                                              │
│                                                                            │
│  This isn't hype. This is measured output.                                 │
│  → See the full competitive analysis                                       │
│                                                                            │
│  → Back to home                                                            │
│  → Get Started                                                             │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

### SUB-PAGE: The Competitive Analysis

**URL:** `/analysis`

**Purpose:** The full paper, formatted for web reading.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE COMPETITIVE ANALYSIS                                                  │
│                                                                            │
│  Eiffel + AI: Challenging Conventional Wisdom About Language Choice        │
│                                                                            │
│  December 2025 | Larry Rix and Claude (Anthropic)                          │
│                                                                            │
│  → Download PDF                                                            │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  [Full paper content, formatted for web reading...]                        │
│                                                                            │
│  TABLE OF CONTENTS                                                         │
│  [Clickable navigation to sections]                                        │
│                                                                            │
│  EXECUTIVE SUMMARY                                                         │
│  [Content]                                                                 │
│                                                                            │
│  CHALLENGING THE "NO DEVELOPERS" MYTH                                      │
│  [Content]                                                                 │
│                                                                            │
│  CHALLENGING THE "NO LIBRARIES" MYTH                                       │
│  [Content]                                                                 │
│                                                                            │
│  [... full paper ...]                                                      │
│                                                                            │
│  → Back to home                                                            │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

#### Visual Treatment

- Long-form reading optimized
- Sticky table of contents for navigation
- Pull quotes for key findings
- Data tables and charts
- PDF download option

---

### SUB-PAGE: The Business Case

**URL:** `/business-case`

**Purpose:** CTO-focused ROI and risk analysis.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE BUSINESS CASE                                                         │
│                                                                            │
│  For CTOs evaluating risk, cost, and capability.                           │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  EXECUTIVE SUMMARY                                                         │
│                                                                            │
│  • 40-80x productivity multiplier demonstrated                             │
│  • 5-day developer training (not months of hiring)                         │
│  • Zero dependency vulnerabilities (you own the code)                      │
│  • Runtime verification catches errors before production                   │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  THE COST EQUATION                                                         │
│                                                                            │
│  Traditional approach:                                                     │
│  • Hire experienced developers ($150-200K+ each)                           │
│  • Months to find qualified candidates                                     │
│  • Ongoing dependency management                                           │
│  • Framework upgrade cycles                                                │
│  • Bug discovery in production                                             │
│                                                                            │
│  Eiffel + AI approach:                                                     │
│  • Train existing developers (5 days)                                      │
│  • Build libraries as needed (hours/days)                                  │
│  • No external dependencies to manage                                      │
│  • Language stability (no framework churn)                                 │
│  • Contract verification catches bugs early                                │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  ROI ANALYSIS                                                              │
│                                                                            │
│  Case study: 11 libraries built in 10 days                                 │
│                                                                            │
│  Traditional estimate: $467,500 - $765,000                                 │
│  Actual cost: ~$7,500 (AI + human time)                                    │
│  ROI: 6,133% - 10,100%                                                     │
│                                                                            │
│  For every $1 invested: $62-$102 in value                                  │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  RISK ANALYSIS                                                             │
│                                                                            │
│  Perceived risks (and reality):                                            │
│                                                                            │
│  "Can't hire Eiffel developers"                                            │
│  Reality: 5-day training, proven with 12+ developers                       │
│                                                                            │
│  "No library ecosystem"                                                    │
│  Reality: Build velocity makes this non-issue                              │
│                                                                            │
│  "Single point of failure"                                                 │
│  Reality: Reference docs capture institutional knowledge                   │
│                                                                            │
│  "Unproven approach"                                                       │
│  Reality: Eiffel is 40 years old. DBC is battle-tested.                    │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  SECURITY ADVANTAGE                                                        │
│                                                                            │
│  Industry data on AI-generated code:                                       │
│  • 45% contains security flaws (Veracode)                                  │
│  • 10,000+ new security findings/month (Apiiro)                            │
│                                                                            │
│  With Design by Contract:                                                  │
│  • Preconditions validate all inputs                                       │
│  • Postconditions verify all outputs                                       │
│  • Invariants ensure consistent state                                      │
│  • Violations caught at runtime, not production                            │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  NEXT STEPS                                                                │
│                                                                            │
│  1. Read the full competitive analysis                                     │
│  2. Review the project portfolio                                           │
│  3. Schedule a conversation                                                │
│                                                                            │
│  → Back to home                                                            │
│  → Contact                                                                 │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

### SUB-PAGE: Why Eiffel?

**URL:** `/why-eiffel`

**Purpose:** Deep dive on the language for the curious.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  WHY EIFFEL?                                                               │
│                                                                            │
│  The language built for correctness. Now paired with AI.                   │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  THE ORIGIN                                                                │
│                                                                            │
│  Eiffel was created by Bertrand Meyer in 1986.                             │
│  One core principle: software should be correct by design.                 │
│                                                                            │
│  Not correct by testing.                                                   │
│  Not correct by code review.                                               │
│  Correct by design.                                                        │
│                                                                            │
│  Design by Contract was built into the language from day one.              │
│  Not added later. Not a library. The language itself.                      │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  WHAT MAKES EIFFEL DIFFERENT                                               │
│                                                                            │
│  Contracts as first-class citizens:                                        │
│  • require, ensure, invariant are language keywords                        │
│  • Every feature can have preconditions and postconditions                 │
│  • Every class can have invariants                                         │
│  • Runtime checks are automatic                                            │
│                                                                            │
│  Void safety:                                                              │
│  • No null pointer exceptions                                              │
│  • The type system prevents void calls                                     │
│  • "The billion dollar mistake" — solved                                   │
│                                                                            │
│  Multiple inheritance done right:                                          │
│  • Inherit from multiple classes                                           │
│  • Rename, redefine, select to resolve conflicts                           │
│  • Powerful composition patterns                                           │
│                                                                            │
│  Language stability:                                                       │
│  • Code from 20 years ago still compiles                                   │
│  • No framework churn                                                      │
│  • No "this year's new pattern"                                            │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  HONEST TRADE-OFFS                                                         │
│                                                                            │
│  We believe in honesty. Here's what you're trading:                        │
│                                                                            │
│  Smaller community:                                                        │
│  • Stack Overflow won't have answers                                       │
│  • Fewer blog posts and tutorials                                          │
│  • You may need to figure things out yourself                              │
│  • AI assistance + reference docs help significantly                       │
│                                                                            │
│  IDE lock-in:                                                              │
│  • EiffelStudio is the primary IDE                                         │
│  • No VS Code, no JetBrains (for now)                                      │
│  • AI-assisted workflow reduces IDE dependency                             │
│                                                                            │
│  Commercial perception:                                                    │
│  • "Eiffel? Is that still around?"                                         │
│  • May need to justify the choice                                          │
│  • Evidence and results speak louder than perception                       │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  WHY NOW?                                                                  │
│                                                                            │
│  Eiffel has been excellent for 40 years.                                   │
│  So why now?                                                               │
│                                                                            │
│  AI changes the equation:                                                  │
│  • AI generates code fast, but not verified                                │
│  • DBC verifies code automatically                                         │
│  • The combination: speed + correctness                                    │
│                                                                            │
│  Reference documentation changes the equation:                             │
│  • AI can be taught Eiffel patterns                                        │
│  • Institutional knowledge compounds                                       │
│  • The "small community" disadvantage shrinks                              │
│                                                                            │
│  The moment is now.                                                        │
│                                                                            │
│  → Back to home                                                            │
│  → Get Started                                                             │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

### SUB-PAGE: From Probable to Provable

**URL:** `/probable-to-provable`

**Purpose:** Meyer's framework explained for the thoughtful reader.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  FROM PROBABLE TO PROVABLE                                                 │
│                                                                            │
│  The framework for AI-assisted development that actually works.            │
│                                                                            │
│  Based on Bertrand Meyer's "AI for software engineering:                   │
│  from probable to provable" (CACM 2025)                                    │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  THE CORE INSIGHT                                                          │
│                                                                            │
│  AI produces statistically likely code.                                    │
│  Not proven correct code. Likely code.                                     │
│                                                                            │
│  "Likely" works for one module.                                            │
│  "Likely" fails at scale.                                                  │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE MATH                                                                  │
│                                                                            │
│  If each module is 99.9% correct:                                          │
│                                                                            │
│  100 modules:    0.999^100   = 90.5% system correctness                    │
│  500 modules:    0.999^500   = 60.6% system correctness                    │
│  1,000 modules:  0.999^1000  = 36.8% system correctness                    │
│  5,000 modules:  0.999^5000  = 0.7% system correctness                     │
│                                                                            │
│  Real systems have thousands of modules.                                   │
│  "Probably correct" becomes "probably broken."                             │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE SOLUTION                                                              │
│                                                                            │
│  Combine AI generation with formal verification.                           │
│                                                                            │
│  AI generates: Fast, bulk, pattern-matching                                │
│  Contracts verify: Complete, automatic, every execution                    │
│                                                                            │
│  Probable + Verification = Provable                                        │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE PROCESS                                                               │
│                                                                            │
│  1. SPECIFICATION                                                          │
│     Write contracts BEFORE implementation.                                 │
│     Define what must be true.                                              │
│                                                                            │
│  2. GENERATION                                                             │
│     AI generates implementation to satisfy contracts.                      │
│     Fast, bulk, pattern-based.                                             │
│                                                                            │
│  3. VERIFICATION                                                           │
│     Compiler checks types.                                                 │
│     Runtime checks contracts.                                              │
│     Tests check scenarios.                                                 │
│                                                                            │
│  4. ITERATION                                                              │
│     Contract violation? Fix spec or implementation.                        │
│     Repeat until correct.                                                  │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE "TRUE BUT INCOMPLETE" TRAP                                            │
│                                                                            │
│  AI often generates contracts that are true but miss guarantees:           │
│                                                                            │
│  AI might write:                                                           │
│      ensure                                                                │
│          has_item: items.has (a_item)                                      │
│                                                                            │
│  Complete contract:                                                        │
│      ensure                                                                │
│          has_item: items.has (a_item)                                      │
│          count_increased: items.count = old items.count + 1                │
│          other_items_unchanged: ...                                        │
│                                                                            │
│  Always ask: "What ELSE is guaranteed?"                                    │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE IMPLICATION                                                           │
│                                                                            │
│  AI without verification: Faster bugs at scale.                            │
│  AI with verification: Faster correctness at scale.                        │
│                                                                            │
│  The technology for verification exists.                                   │
│  It's called Design by Contract.                                           │
│  It's built into Eiffel.                                                   │
│                                                                            │
│  → Back to home                                                            │
│  → How Design by Contract Works                                            │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

### SUB-PAGE: The Real Cost of the Old Way

**URL:** `/old-way`

**Purpose:** Expanded pain points for recognition/empathy.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  THE REAL COST OF THE OLD WAY                                              │
│                                                                            │
│  The friction you've learned to live with.                                 │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  DEPENDENCY HELL                                                           │
│                                                                            │
│  "npm install" for a simple project.                                       │
│  node_modules: 500 packages.                                               │
│  You wrote 200 lines. You're responsible for 2 million.                    │
│                                                                            │
│  Then the security advisory:                                               │
│  "Critical vulnerability in left-pad-utils-helper-lite"                    │
│                                                                            │
│  You've never heard of it.                                                 │
│  It's four dependencies deep.                                              │
│  It's in production.                                                       │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  FRAMEWORK CHURN                                                           │
│                                                                            │
│  2018: "React Hooks change everything!"                                    │
│  2020: "Server Components are the future!"                                 │
│  2022: "Signals are the new paradigm!"                                     │
│  2024: "AI-first frameworks are here!"                                     │
│                                                                            │
│  Every 18 months: Rewrite.                                                 │
│  Every 18 months: Relearn.                                                 │
│  Every 18 months: "This time it's stable."                                 │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  MEETINGS THAT SHOULD BE CODE                                              │
│                                                                            │
│  Week 1: "Which database should we use?"                                   │
│  Week 2: "Let's evaluate three ORMs."                                      │
│  Week 3: "We need to align on architecture."                               │
│  Week 4: "Stakeholder review of the tech stack."                           │
│  Week 5: "Let's revisit the database decision."                            │
│                                                                            │
│  Meanwhile: Nothing shipped.                                               │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE BUG THAT "SHOULDN'T HAVE HAPPENED"                                    │
│                                                                            │
│  The code looked right.                                                    │
│  The tests passed.                                                         │
│  Code review approved.                                                     │
│                                                                            │
│  Production: NullPointerException at 2 AM.                                 │
│                                                                            │
│  "But that should never be null there."                                    │
│  "The tests cover this."                                                   │
│  "It worked in staging."                                                   │
│                                                                            │
│  Surprise. Hope isn't a strategy.                                          │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE COST ADDS UP                                                          │
│                                                                            │
│  Hours spent on dependency updates: __                                     │
│  Hours spent on framework migrations: __                                   │
│  Hours spent in decision meetings: __                                      │
│  Hours spent debugging "impossible" bugs: __                               │
│                                                                            │
│  What could you have built instead?                                        │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THERE'S ANOTHER WAY                                                       │
│                                                                            │
│  Not hope. Proof.                                                          │
│  Not dependencies. Ownership.                                              │
│  Not churn. Stability.                                                     │
│  Not meetings. Building.                                                   │
│                                                                            │
│  → Back to home                                                            │
│  → See what got built                                                      │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

### SUB-PAGE: What AI Actually Changes

**URL:** `/ai-changes`

**Purpose:** Balanced view of AI capabilities and limitations.

#### Page Structure

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  WHAT AI ACTUALLY CHANGES                                                  │
│                                                                            │
│  The real impact. Not the hype.                                            │
│                                                                            │
│  ══════════════════════════════════════════════════════════════════════    │
│                                                                            │
│  WHAT AI DOES WELL                                                         │
│                                                                            │
│  Speed:                                                                    │
│  • Boilerplate generation in seconds                                       │
│  • Pattern application across files                                        │
│  • Bulk operations that would take hours                                   │
│                                                                            │
│  Pattern recognition:                                                      │
│  • "Make this look like that"                                              │
│  • Consistent style across codebase                                        │
│  • Applying established conventions                                        │
│                                                                            │
│  Documentation:                                                            │
│  • README generation                                                       │
│  • Code comments                                                           │
│  • API documentation                                                       │
│                                                                            │
│  Exploration:                                                              │
│  • "How might I approach this?"                                            │
│  • Rapid prototyping                                                       │
│  • Trying multiple solutions quickly                                       │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  WHAT AI DOESN'T DO                                                        │
│                                                                            │
│  Guarantee correctness:                                                    │
│  • AI generates probable code, not proven code                             │
│  • "Looks right" isn't "is right"                                          │
│  • Edge cases are often missed                                             │
│                                                                            │
│  Understand your business:                                                 │
│  • AI doesn't know your domain deeply                                      │
│  • Business rules need human specification                                 │
│  • Context matters, and AI has limited context                             │
│                                                                            │
│  Replace judgment:                                                         │
│  • Architectural decisions need human wisdom                               │
│  • Trade-offs require understanding consequences                           │
│  • "Should we do this?" is a human question                                │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE EVIDENCE OF LIMITATIONS                                               │
│                                                                            │
│  [The research citations from Section 3...]                                │
│                                                                            │
│  • 41% more bugs (Uplevel)                                                 │
│  • 45% security flaws (Veracode)                                           │
│  • Trust dropping: 43% → 33% (Stack Overflow)                              │
│  • 85% task failure rate (Devin)                                           │
│                                                                            │
│  AI is powerful. AI alone isn't enough.                                    │
│                                                                            │
│  ──────────────────────────────────────────────────────────────────────    │
│                                                                            │
│  THE COMBINATION THAT WORKS                                                │
│                                                                            │
│  AI + Human judgment + Verification                                        │
│                                                                            │
│  AI generates fast.                                                        │
│  Humans direct and decide.                                                 │
│  Contracts verify automatically.                                           │
│                                                                            │
│  Speed × Wisdom × Proof = Shipping with confidence.                        │
│                                                                            │
│  → Back to home                                                            │
│  → How Design by Contract Works                                            │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## Technical Implementation Notes

### Technology Stack

```
Built with:
• Eiffel — Backend generation (of course)
• simple_htmx — HTML structure
• simple_alpine — Interactions and state
• Tailwind CSS — Styling (or custom CSS)
• Alpine.js plugins — x-intersect, x-collapse

No build step required for:
• Static HTML generation
• CSS (Tailwind CDN or custom)
• Alpine.js (CDN)

Optional enhancements:
• Lenis — Smooth scrolling
• GSAP — Advanced animations
• View Transitions API — Page transitions
```

### Animation Implementation

```javascript
// Scroll-triggered reveals with Alpine.js x-intersect
<div x-data="{ shown: false }"
     x-intersect.threshold.25="shown = true"
     :class="shown ? 'opacity-100 translate-y-0' : 'opacity-0 translate-y-8'"
     class="transition-all duration-700">
  Content here
</div>

// Staggered reveals
<div x-data="{ shown: false }" x-intersect="shown = true">
  <p :class="shown ? 'opacity-100' : 'opacity-0'"
     class="transition-opacity duration-500 delay-[0ms]">Line 1</p>
  <p :class="shown ? 'opacity-100' : 'opacity-0'"
     class="transition-opacity duration-500 delay-[150ms]">Line 2</p>
  <p :class="shown ? 'opacity-100' : 'opacity-0'"
     class="transition-opacity duration-500 delay-[300ms]">Line 3</p>
</div>

// Number counting animation
<span x-data="{ count: 0, target: 900 }"
      x-intersect.once="
        let start = 0;
        let duration = 2000;
        let startTime = null;
        function animate(timestamp) {
          if (!startTime) startTime = timestamp;
          let progress = (timestamp - startTime) / duration;
          if (progress < 1) {
            count = Math.floor(target * progress);
            requestAnimationFrame(animate);
          } else {
            count = target;
          }
        }
        requestAnimationFrame(animate);
      "
      x-text="count">
</span>
```

### Scroll Snap Setup

```css
html {
  scroll-snap-type: y mandatory;
  scroll-behavior: smooth;
}

section {
  scroll-snap-align: start;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```

### Color Transitions Between Sections

```css
/* Each section defines its own background */
.section-hero { background: #0c0b0b; }
.section-recognition { background: #0f0c0a; }
.section-shift { background: #1a0d2a; }
.section-problem { background: #1a0f05; }
.section-unlock { background: #0a2a2a; }
.section-evidence { background: #0d1428; }
.section-revelation { background: #0d1020; }
.section-workflow { background: #0f1520; }
.section-invitation { background: #12100f; }
```

---

## Build Priority

### Phase 1: Minimum Viable Showcase

1. Landing page with all 8 sections (basic animations)
2. Get Started page
3. Project Portfolio page
4. Basic navigation
5. Responsive design

**Deliverable:** Working site that tells the full story.

### Phase 2: Depth & Polish

6. How Design by Contract Works
7. The Human-AI Workflow
8. Enhanced animations (staggered reveals, counting numbers)
9. Dark mode toggle
10. Smooth scroll implementation

**Deliverable:** Polished site with key supporting content.

### Phase 3: Complete Content

11. The Competitive Analysis (formatted paper)
12. The Business Case
13. Why Eiffel?
14. From Probable to Provable
15. The Real Cost of the Old Way
16. What AI Actually Changes

**Deliverable:** Complete site with all content.

### Phase 4: Enhancement

17. Page transitions
18. Advanced animations (GSAP)
19. Sound design (optional)
20. Performance optimization
21. Analytics

**Deliverable:** Premium experience.

---

## Final Notes

This document is the complete blueprint. Everything needed to build simple_showcase is here:

- All copy written
- All visual direction specified
- All interactions scripted
- All sub-pages detailed
- Technical approach outlined
- Build priority defined

The site itself becomes proof of the paradigm:
- Built with Eiffel + AI
- Contract-verified
- Shipped fast

**"While others were debating, we were building."**

---

**Document Version:** 1.0
**Last Updated:** December 4, 2025
**Ready for implementation.**
