---
name: ui-ux-design
description: >
  Apply UI/UX design principles to any interface work — web apps, Godot games, dashboards, forms,
  landing pages, mobile layouts, interactive widgets, or design critiques. Use this skill whenever
  the user asks to design, review, improve, or plan a user interface or user experience. Also trigger
  when the user mentions wireframes, user flows, information architecture, usability, onboarding,
  conversion, user psychology, visual hierarchy, accessibility, A/B testing, user personas, cognitive
  load, gamification, color palette, typography, responsive layout, dark mode, spacing, grid system,
  or any design decision that affects how a person interacts with software.
  Even if the user just says "make this better", "this feels off", "what colors should I use",
  or "how should I lay this out" about an interface, use this skill.
---

# UI/UX Design Skill

A practical reference for designing interfaces that make users effective — not just happy.
Grounded in principles from Joel Marsh's *UX for Beginners*, Abigail Rindo's
*Game Narrative Design and UX Fundamentals*, and Elisa Paduraru's *Roots of UI/UX Design* (Creative Tim, 2024).

Read `references/principles.md` for the full principle library (visual design, psychology,
usability heuristics, game UX, copywriting, data analysis). This file covers the workflow
and decision framework; the reference file is the deep-dive you consult mid-design.

---

## Core Philosophy

UX is about making users **effective**, not just happy. A user's visible experience is the
tip of the iceberg — the invisible structure underneath (information architecture, psychology,
data) is where the real impact lives.

The five ingredients of UX, in every project: **Psychology, Usability, Design, Copywriting, Analysis.**

---

## Design Workflow

### 1. Define Goals (Before Anything Else)

Every project has two sets of goals that must be aligned:

- **User goals**: What does the user want to achieve? (find a video, buy shoes, beat a level)
- **Business/project goals**: What does the product need? (revenue, registrations, engagement)

The design succeeds when the user completing *their* goal also completes *yours*.
If they're misaligned, you'll get either lots of users with no success, or no users at all.

### 2. Understand the User

Before designing, ask:
- What is the user's motivation to be here?
- What do they already know? (You always know more than they do — design for their knowledge, not yours.)
- What device and context are they in? (Mobile-first: start with the smallest, least powerful device.)
- Are they browsing (emotional, visual scanning), searching (systematic, attribute-focused), or discovering?

Build user profiles based on goals, expectations, motivations, and behavior — not demographics.
A useful persona lets you say "no" to feature ideas. If it can't, it's useless.

### 3. Structure the Information (IA)

Read `references/principles.md § Information Architecture` for IA types (categories, tasks, search, time, people).

Key rules:
- Design the site map before any wireframes. Deep or flat, not both.
- Forget "three clicks" — focus on whether users understand where they are and where they can go.
- Never create dead ends. Always provide a next step.
- Users don't go backward voluntarily — if they hit "back," they're lost. Design forward-only loops.
- Design containers, not content (for dynamic pages). Plan for empty states, deletions, and abuse.

### 4. Design the Interface

**Visual hierarchy drives scanning.** Users don't read — they scan in Z-patterns (boring layouts) or
F-patterns (structured layouts with headlines and sections). Design to break the Z into an F using:

- **Visual weight**: Contrast and size draw attention. Important things are bigger and higher-contrast.
- **Color**: Use it functionally — advancing colors (red, orange) for CTAs, receding colors (blue, grey) for chrome.
- **Repetition and pattern-breaking**: Create visual patterns, then break them where you want focus.
- **Line tension and edge tension**: Align elements to create invisible paths the eye follows.
- **Alignment and proximity**: Related things close together, unrelated things apart. No explanation needed.
- **The Axis of Interaction**: The strongest visual edge in your layout. Put CTAs on it. Put secondary actions off it.

**Grid and spacing (use these as defaults):**
- Desktop: 12-column grid, gutters of 12–16pt, margins of 160–180pt.
- The **8pt rule**: all element dimensions should be multiples of 8 (16, 24, 32, 40…). Most screen sizes divide evenly by 8, keeping spacing harmonious.
- Text baseline grid: 4px increments.
- Minimum gutter between elements: 16px. Never below 8px except in constrained layouts.
- **Fluid grids** (column widths scale with viewport, fixed margins/gutters) for responsive layouts. **Fixed grids** for content that must not reflow.

**Layout framework** (do this first, before page content):
1. Navigation (main menu + submenu) — same place on every page, ordered by user interest.
2. Footer — static links too minor for nav. If you have infinite scroll, duplicate footer links elsewhere.
3. Page content — headlines above the fold to signal what's below. Images of people direct eye gaze.

**Typography specifics:**
- Line height for body text: font-size × 1.6 (e.g., 16px → ~26px line-height).
- Optimal line length: shorter is better. Long lines increase eye travel and reduce comprehension.
- Letter spacing: default to 0; only adjust for ALL-CAPS runs (increase) or display headlines. Don't over-track body text.
- Type scale: use 10+ defined text styles with specific purposes. Limit a single section to **no more than 3 font sizes**.
- Serif for elegance/editorial; sans-serif for clean/contemporary — pick one family per role and stick to it.

**Color system:**
- **60/30/10 rule**: 60% dominant/neutral, 30% secondary, 10% accent. Creates natural visual harmony.
- **Avoid pure black** (#000000) on white — the extreme contrast causes eye strain during reading. Use dark gray (~#1a1a1a) instead.
- Build a 5-step palette: (1) primary color, (2) secondary color, (3) neutral/background, (4) tints and shades of each for cards/overlays, (5) verify contrast ratios (use contrast-ratio.com; target WCAG AA minimum).
- **Dark mode**: tints create more contrast than shades in dark mode. Never use white shadows in dark mode — use a darker shade of the element's color + a lighter tint from the background.
- For gradients between complementary colors, add a third bridging color from the color wheel to prevent a muddy grey midpoint.

**Shadows and depth:**
- **Dual shadow technique**: use both a core shadow (tight, higher opacity) and a cast shadow (wide, lower opacity). Using only one layer produces flat, unrealistic depth.
- Shadow placement: below the element (matching real-world light from above). Never position the primary shadow above an element.
- **Dark mode shadows**: avoid white shadows — they're harsh and visually tiring. Use element-color darks paired with background-color tints.

**Buttons:**
- Primary (productive actions — buy, save, share): High contrast, on the Axis of Interaction.
- Secondary (destructive/cancel actions): Low contrast, off the Axis.
- Destructive-but-important (delete account): Primary style, secondary position, warning color.
- **Minimum touch target**: 36×36px (Material Design) to 40×40px (Apple HIG). Never smaller.
- **Rounded corners** are psychologically easier to process — sharp corners push focus outward; rounded corners pull focus inward toward the label.
- Button sizing: base vertical padding is 12px; width ≈ 2× height as a starting point. Maintain consistent border-radius, typography, and shadow across all button variants.
- All buttons need **6 states**: Default, Hover, Active/Pressed, Progress/Loading, Focus (keyboard), Disabled.

**Forms:**
- Keep them short. Break long forms into pages. Save progress between steps.
- Labels: short, clear, close to input. "Home Address" beats "Where your heart is…"
- Inline validation for verifiable fields. Show errors visible from the bottom of the form.
- Common questions → labels above input (fast). Complex questions → labels beside input (fewer errors).
- Put the "done" button on the Axis of Interaction (left). Destructive actions go off-axis (right).
- **Input states to design**: Default, Focus, Filled, Error (with actionable message, not just "invalid"), Disabled.
- **Dropdowns**: add a search field for lists longer than ~8–10 items. Use a visible scrollbar to signal more options exist.
- **Radio buttons** (mutually exclusive single choice) vs. **checkboxes** (multiple independent choices) — never confuse these.
- **Toggles**: never put "ON"/"OFF" text inside the toggle graphic. Use color alone to indicate state (e.g., green = active).
- Rounded input fields are preferred; include relevant icons inside fields to improve scannability.

**Icons:**
- Maintain one icon style throughout (outline, glyph, duo-tone, etc.) — mixing styles confuses users.
- Use ≤ 3–4 colors per icon; keep line widths consistent with each other and with adjacent text strokes.
- Rounded corners on icons follow the same cognitive rule as buttons: easier to process.
- Scale icons in vector format for resolution-independence.
- Types: **clarification icons** (paired with text to reinforce meaning) vs. **decorative icons** (aesthetic/engagement). Don't confuse their roles.
- Gradient and 3D icon styles: use sparingly — they cause visual fatigue at high density.

**Images:**
- **Rule of Thirds**: divide composition into a 3×3 grid; place focal elements at intersection points for natural balance.
- **Text over images**: always apply a color overlay at 50–80% opacity to ensure text legibility. Never rely on image contrast alone.
- **Visual uniformity**: establish a style guide or mood board. All images in a product should share style, palette, and composition tone.
- **Open images**: choose shots with visual breathing room so design elements can coexist without clutter.
- **Contextual relevance**: mismatched imagery causes confusion. Match image mood to brand concept.
- Images of people build trust and naturally direct eye gaze toward CTAs.
- Always compress before upload. Blurry or slow-loading images communicate low quality immediately.

### 5. Apply Psychology

Read `references/principles.md § User Psychology` for the full model.

**Key cognitive principles to apply in every design:**

- **Hyperbolic discounting**: Whatever is happening now feels more important than later. Make the desired action feel immediate; make destructive actions feel slow and deliberate.
- **Paradox of choice**: More options = harder to choose = users leave. Curate ruthlessly.
- **Anchoring**: The first number/option users see biases all subsequent judgments.
- **Defaults**: The lazy option wins. Make the default the best choice for both user and business.
- **What You See Is All There Is**: Users only consider the options presented. Design choices that move them toward their goal.
- **Cognitive load**: Every detail that makes the user think is a brick they carry. Minimize bricks.

**Motivation model (for engagement/gamification/social features):**
- Affiliation (belong to a group), Status (compete/control/be the best), Justice (fairness/balance).
- Rewards and punishments are *feelings*, not things. Focus on giving (sense of progress) over taking.
- Classical conditioning: Associate your signals with existing rewards.
- Operant conditioning: Reward good behavior. Random rewards based on quality create the most engagement.
- First reward must be free, within 60 seconds of first use.

### 6. Write the Copy

UX copy is not brand copy. UX copy gets things done. Brand copy creates associations.

**Call-to-action formula**: `Verb + Benefit + Urgent Time/Place`
- Good: "Get Your Free Trial Now"
- Bad: "Click here to maybe start the process of potentially beginning"

**Labels and instructions**: Short, literal, direct. Write for smart children or non-native speakers.
Button labels should be self-explanatory without the headline: "Save Changes" not "Ok".

**Trust signals**: Professional appearance, balanced reviews (not all 5-star), accountability,
handle negativity gracefully, keep users informed about costs/privacy, use simple words.

### 7. Launch and Measure

Read `references/principles.md § Data and Analytics` for graph shapes and stat interpretation.

Launch is an experiment, not an event. Measure:
- Sessions vs. Users (loyalty), New vs. Return visitors (health), Pageviews (engagement or confusion?),
  Time per page (reading or lost?), Bounce rate (below 30% = good, above 70% = emergency), Exit rate.

**A/B test** when choosing between subjective options. Change only one variable per test. Don't stop
early. More traffic = more reliable. When two psychological strategies compete, only a test can decide.

**Key insight**: Probabilities, not certainties. 90% is "everybody." 1% of users will do *anything*.
A feature used by 1% doesn't justify its existence — those clicks are stolen from something better.

---

## UI Design Trend Vocabulary

When critiquing or referencing existing designs, these are the major visual language families:

- **Flat Design** (2010s+): Clean 2D shapes, no gradients/shadows, bold colors, grid-based. Prioritizes legibility and simplicity.
- **Material Design** (Google, 2014+): Flat base + elevation (shadows indicate Z-axis hierarchy). Tactile metaphors, comprehensive icon system.
- **Neubrutalism**: Heavy typography, thick black borders, raw/exposed layout. High contrast, intentionally unpolished.
- **Neumorphism**: Soft UI — extruded from background using dual inward/outward shadows. Very subtle contrast. **Accessibility risk** (fails contrast requirements).
- **Glassmorphism**: Frosted-glass transparency, vibrant backgrounds visible through panels, minimalist layout. Modern layered depth.

Each style carries tradeoffs between aesthetics, accessibility, and usability. Never choose a trend without evaluating accessibility impact.

---

## Game UX Specifics

When designing for games (Godot, Unity, or any game engine), also consider:

- **Core gameplay loop**: The repeating cycle of action → feedback → reward. Apply UX to make each
  step intuitive. Layer loops into mastery loops (same actions, harder challenges, bigger rewards).
- **Mental models**: Use real-world metaphors to teach game systems. A "health bar" works because
  everyone understands the concept of diminishing resources.
- **Onboarding**: Teach through play, not text walls. Scaffold complexity — introduce one mechanic
  at a time. Use Bloom's Taxonomy: remember → understand → apply → analyze → evaluate → create.
- **Diegetic UI**: Interface elements that exist within the game world (a watch on a character's wrist
  for time, a radio for quest updates) increase immersion.
- **Feedback loops**: Motivation → Action → Feedback → (repeat). The feedback must re-motivate.
- **Progressive challenge**: Easy start, increasing difficulty. Players pay (in effort or money) for
  higher levels because they're already conditioned to the loop.
- **Player personas**: Different players want different things (mastery, exploration, social, narrative).
  Design for all of them, but prioritize based on your game's core identity.

---

## Responsive & Multi-Device

1. **Start small** (mobile-first) — forces focus on core content and functionality.
2. **Finger vs. mouse**: Touch is direct but imprecise. Mouse is indirect but accurate. Touch needs
   larger targets, no hover states, gesture-based discovery. Mouse gets hover, precise selection, cursor changes.
3. **Break points**: A single layout can't stretch forever. Define where layouts shift.
4. **Device powers**: Mobile has location, always-available touch, gyroscope. Desktop has keyboard,
   precision, multi-window. Design for each device's strengths rather than forcing consistency.
5. **Thumb zones**: Position primary interactive elements in the lower-center zone where thumbs reach comfortably. Keep destructive or secondary actions toward the top or edges.
6. **Touch target spacing**: Minimum 8–10px between adjacent touch targets to prevent accidental taps.
7. **Mobile forms**: Minimize typing — auto-fill, smart suggestions, and strong defaults drastically reduce friction.
8. **54%+ of web traffic is mobile** — not designing mobile-first is no longer defensible.

---

## Accessibility Checklist

- Text size: Big enough for the context. When in doubt, go bigger.
- Color: Don't rely on red/green alone (10% color blindness). Use shape + color for states.
- Contrast: Check all text/background pairs. WCAG AA minimum. Use contrast-ratio.com.
- Screen readers: Code order = reading order. Tab order matters.
- Language: Simple words, easy grammar. Language selector is a make-or-break feature for global audiences.
- Content: Shorter is better for listening. Easy to scan, easy to jump within.
- Touch targets: ≥ 40×40px on mobile. ≥ 8–10px spacing between adjacent interactive elements.

---

## Anti-Patterns to Avoid

- Hamburger menus on desktop (lazy, not simple — users can't see what they can't find)
- "Make the logo bigger" (unless you want users staring at your logo instead of converting)
- Lorem ipsum in usability reviews (you can only test readability with real content)
- Everything-in-one mega-menu (your IA needs work, not a bigger menu)
- Surface attention (cool animations that distract from content and CTAs)
- Dark patterns / deceptive UX (builds short-term metrics, destroys trust permanently)
- Pure black text on white backgrounds (causes eye strain; use near-black)
- White shadows in dark mode (visually harsh; use element-color darks + background tints)
- Single shadow layers for depth (always pair core shadow + cast shadow)
- "ON/OFF" text inside toggle graphics (use color state alone)
- Mixing icon styles in the same product (visual chaos)
- Gradients between complementary colors without a bridging hue (produces muddy grey midpoint)

---

## When Reviewing an Existing Design

Apply this checklist:
1. Can a new user answer the Three Whats within 5 seconds? (What is this? What's in it for me? What do I do next?)
2. Is the visual hierarchy clear? (Biggest text = most important? CTAs visible and on the Axis?)
3. Are user goals and business goals aligned?
4. Is anything competing for attention unnecessarily?
5. Are forms as short as possible with smart defaults?
6. Does it work on the smallest target device?
7. What does the empty state look like? The error state? The "deleted" state?
8. What's the worst a user could do? (Marsh's Law: every feature will be abused to its maximum abusability.)
9. Do spacing/dimensions follow the 8pt rule? Is type limited to ≤3 sizes per section?
10. Are button touch targets ≥ 40px? Is there ≥ 8–10px spacing between adjacent tap targets?
11. Does the color palette follow 60/30/10? Is pure black avoided? Are contrast ratios WCAG-compliant?
12. Are shadows realistic (core + cast)? Are images contextually appropriate with clear focal points?
