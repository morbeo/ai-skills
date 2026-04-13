# UI/UX Principles Reference

Deep-dive principle library. Read relevant sections mid-design; the main SKILL.md covers the workflow.

---

## Information Architecture

IA organises content so users can find it. Five core IA types — most products use a combination:

| Type | Organises by | Example |
|---|---|---|
| **Categories** | Topic or attribute | E-commerce nav: Electronics > Phones |
| **Tasks** | What the user does | "Pay bill", "Check balance", "Open account" |
| **Search** | Query + results | Google, site search, filtered tables |
| **Time** | Chronology | Newsfeeds, activity logs, calendars |
| **People** | Social graph | Facebook friends list, org chart |

### IA rules
- Design the site map before wireframes. Decide: deep (few top-level, many nested) or flat (many top-level, less nesting). Don't mix both — it produces navigational confusion.
- Forget "three clicks" — research shows click count doesn't predict frustration. Comprehension does. Can users always tell where they are and where they can go next?
- Every page needs a next step. Dead ends erode trust.
- Users don't backtrack voluntarily. If they hit back, they're lost — redesign the forward path.
- Design containers, not content. On dynamic pages, plan for: empty state, single item, many items, very long items, deleted items, error states.

### Card sorting (when to use)
Run a card sort when you're unsure how users categorise your content. Open sort (user creates categories) reveals mental models. Closed sort (user places cards in your categories) validates a proposed IA. Run with 15–20 participants for reliable clusters.

---

## User Psychology

### Cognitive biases designers must account for

| Bias | Effect | Design response |
|---|---|---|
| **Hyperbolic discounting** | Now > later | Make desired action immediate; slow destructive actions |
| **Anchoring** | First number seen biases all others | Lead pricing with the option you want chosen |
| **Paradox of choice** | More options → harder to choose → exit | Curate; use progressive disclosure |
| **Status quo bias** | Defaults win | Make the best option the default |
| **WYSIATI** (What You See Is All There Is) | Users only consider what's visible | Surface what drives decisions |
| **Confirmation bias** | Users notice what confirms expectations | Design onboarding to set correct expectations first |
| **Social proof** | "Others did this, so it's safe" | Show ratings, testimonials, user counts |
| **Scarcity** | Limited = valuable | "3 left in stock", "offer ends Friday" — use ethically |
| **Loss aversion** | Losses hurt ~2× more than gains please | Frame as avoiding loss, not gaining something |
| **Zeigarnik effect** | Incomplete tasks stay in working memory | Use progress bars, checklists, streaks |

### Motivation model

Three core social motivators (Marsh):
- **Affiliation** — belonging to a group, identity, community membership
- **Status** — competition, ranking, being the best, control over others
- **Justice** — fairness, balance, reciprocity

Rewards and punishments are feelings, not objects. Design for:
1. First reward free and within 60 seconds of first use
2. Variable ratio reinforcement (random rewards based on quality) = highest engagement
3. Classical conditioning: pair your product signals with existing pleasures
4. Operant conditioning: reward good behaviour immediately and consistently

### Fitts's Law
The time to reach a target = log₂(distance / size + 1). Large, nearby targets are faster to click.
- Place primary actions within easy reach (bottom of screen on mobile, near cursor resting positions on desktop)
- Make CTAs large; make destructive actions small
- Minimise distance between related sequential actions (e.g. "next step" button near the last input)

### Hick's Law
Decision time = log₂(number of choices + 1). More choices = slower decisions.
- Limit primary navigation to 5–7 items
- Break complex forms into steps
- Use progressive disclosure — reveal options only when relevant

### Miller's Law
Working memory holds 7 ± 2 items. Chunking helps: phone numbers, postal codes, credit cards.
- Group related fields
- Use section headers to reset chunks
- Never present more than 7 top-level navigation items

---

## Visual Design Principles

### Gestalt principles

| Principle | Meaning | Application |
|---|---|---|
| **Proximity** | Near things appear related | Group labels with their inputs |
| **Similarity** | Like things appear related | Consistent button styles = same function |
| **Continuity** | Eyes follow lines and curves | Use alignment to guide reading path |
| **Closure** | Mind completes incomplete shapes | Partial cards signal more content to scroll |
| **Figure/Ground** | Object vs. background distinction | Modal overlays, tooltips, dropdown menus |
| **Common fate** | Moving together = belonging together | Animate grouped elements simultaneously |

### Colour theory for UI

**Colour temperature:**
- Warm (red, orange, yellow): advancing, attention-grabbing, urgent
- Cool (blue, green, purple): receding, calming, trustworthy, professional
- Neutral (grey, beige, white/black): background and structure

**60/30/10 rule:**
- 60% dominant neutral (backgrounds, large surfaces)
- 30% secondary brand colour (headers, navigation, key UI elements)
- 10% accent (CTAs, highlights, alerts)

**Contrast requirements (WCAG 2.1):**
- Normal text (< 18pt): minimum 4.5:1, enhanced 7:1
- Large text (≥ 18pt or bold ≥ 14pt): minimum 3:1
- UI components and graphics: minimum 3:1
- Use contrast-ratio.com or browser DevTools to check

**Dark mode colour handling:**
- Use tints (lighten) for accent colours — they provide more contrast against dark backgrounds than shades
- Avoid white shadows; use a slightly lighter version of the element's colour + darker background tint
- Background stack: surface-0 (darkest) → surface-1 → surface-2 → surface-3 (lightest) — reverse of light mode

**Colour blindness:**
- Red/green confusion (deuteranopia) affects ~8% of males, ~0.5% of females
- Never use red/green alone to encode a binary state (success/error) — add icons, labels, or patterns
- Blue/yellow is the safe signal-pair alternative

---

## Usability Heuristics (Nielsen's 10)

1. **Visibility of system status** — always keep users informed; feedback within 1s for actions
2. **Match between system and real world** — speak the user's language; use familiar metaphors
3. **User control and freedom** — undo and redo; clearly marked escape routes
4. **Consistency and standards** — follow platform conventions; same label = same function
5. **Error prevention** — better than good error messages; confirm before irreversible actions
6. **Recognition over recall** — surface options; don't force users to remember between steps
7. **Flexibility and efficiency** — shortcuts for experts; accelerators invisible to novices
8. **Aesthetic and minimalist design** — every extra unit of information competes with relevant info
9. **Help users recognise, diagnose, and recover from errors** — plain language, not codes; suggest a fix
10. **Help and documentation** — searchable, task-oriented, not feature-oriented

---

## Game UX Principles

### Celia Hodent's model — brain bottlenecks

| Bottleneck | Problem | Solution |
|---|---|---|
| **Perception** | Players can't see/hear/feel the feedback | Visual hierarchy, audio cues, haptics |
| **Memory** | Players forget how mechanics work | Just-in-time tutorials, persistent HUD |
| **Attention** | Players focus on wrong thing | Reduce noise, guide with light/colour/sound |
| **Motivation** | Players don't want to play | Clear goals, autonomy, competence building |
| **Engagement** | Players quit before the hook | First 10 minutes must deliver a win |

### Onboarding design (Bloom's Taxonomy applied)
1. **Remember** — player can recall what a button does after being shown once
2. **Understand** — player can explain why a mechanic exists
3. **Apply** — player uses the mechanic spontaneously in a new situation
4. **Analyse** — player combines mechanics to solve problems you didn't anticipate
5. **Evaluate** — player can critique their own strategy
6. **Create** — player invents new approaches within your system

Teach one concept per session. Never proceed to "apply" before "remember" is confirmed.

### Diegetic vs non-diegetic UI
- **Diegetic**: exists within the game world (health shown on character's armour, ammo on the gun)
- **Non-diegetic**: overlaid on screen only the player sees (HUD bars, minimaps)
- **Spatial**: anchored to world but not part of it (floating name tags)
- **Meta**: affects screen as medium (screen crack effect for damage)

Use diegetic UI to increase immersion; use non-diegetic for clarity under pressure.

---

## Data and Analytics

### Reading graph shapes

| Shape | What it signals |
|---|---|
| **Flat line** | Stability or stagnation |
| **Hockey stick** | Exponential growth (confirm it's not data artefact) |
| **Cliff drop** | Sudden change — correlate with deploy/event |
| **Sawtooth** | Periodic behaviour (daily/weekly cycle) |
| **Step function** | Discrete change — new cohort, feature flag, external event |
| **Log curve** | Diminishing returns — normal for adoption, virality |

### Key web metrics and their meaning

| Metric | Healthy | Warning | What it tells you |
|---|---|---|---|
| Bounce rate | < 30% | > 70% | Does landing content match expectation? |
| Session duration | Depends on goal | Trending down | Engagement or confusion? |
| Conversion rate | Depends on goal | Below baseline | Funnel friction |
| DAU/MAU ratio | > 0.2 | < 0.1 | Product stickiness (daily habit formation) |
| Retention D1/D7/D30 | > 40% / > 20% / > 10% | Steep drop D1→D7 | Onboarding quality |
| NPS | > 30 good, > 50 excellent | < 0 | Promoters vs detractors |

**Probabilities, not certainties.** 90% of users doing something still means 10% don't. Design for both.
1% of users will do anything — don't build features for 1%.

### A/B testing principles
- Change **one variable** per test
- Define success metric **before** running the test
- Don't stop early — wait for statistical significance (p < 0.05, power ≥ 0.8)
- Minimum detectable effect: know what change is business-meaningful before deciding sample size
- Segment results: an overall win may hide a loss for a key user segment
- **Duration**: run for at least 2 full weekly cycles to account for day-of-week effects

### Funnel analysis
Map each step users take toward a goal. Measure drop-off at each step. The largest drop is the highest-leverage fix.

```
Landing → Sign up → Onboarding → First value → Retention
  100%      60%        45%           30%          22%
            ↑           ↑             ↑
         -40%         -25%          -33%
                                    ← biggest drop = fix here first
```
