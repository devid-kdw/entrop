# ENTROP — Visual Identity & UI Design Brief v0.1

**Document type:** Visual design specification — authoritative reference for all design, frontend, and test agents  
**Authority over:** Color system, typography, layout architecture, control design language, animation rules, live visualization behavior, asset delivery format, render workflow  
**Companion documents:** `ENTROP_Developer_Plan_v01.md` (DSP/CLAP authority) · `ENTROP_Musicality_Spec_v02.md` (musical acceptance authority)  
**Version:** 0.1

> **FOR AI AGENTS:** This document is the single source of truth for all visual and interaction design decisions. The Design agent produces specifications and assets according to this document. The Frontend agent implements those specifications. Neither agent invents design decisions not described here. When a section provides exact hex values, pixel dimensions, or behavioral rules, implement them exactly — do not substitute alternatives. When a section says something is forbidden, it is forbidden regardless of aesthetic preference.

---

## Document Map — What Each Agent Reads

| Section | Content | Primary Agent |
|---|---|---|
| 1 | Conceptual direction — the central metaphor | Design, Frontend |
| 2 | Colour system — all hex values, usage rules | Design, Frontend, Test |
| 3 | Typography — typefaces, sizes, treatment rules | Design, Frontend |
| 4 | Layout architecture — window structure, proportions | Design, Frontend |
| 5 | Live visualizations — STOCH, DIFFU, BUBBLE panels | Design, Frontend, Backend |
| 6 | Control design language — knobs, XY pad, blend triangle | Design, Frontend |
| 7 | Motion and animation — permitted and forbidden | Design, Frontend, Test |
| 8 | Technical requirements — rendering pipeline, asset formats | Design, Frontend |
| 9 | Graphic agent workflow — render discipline, Antigravity constraints | Design |
| 10 | Style constraints — visual QA pass/fail criteria | Design, Test |
| 11 | Preset browser and naming language | Design, Frontend |
| 12 | Deliverable checklist — phased output requirements | Design, Orchestrator |

---

## 1. Conceptual Direction

### 1.1 The Central Metaphor: The Observation Window

> **AGENT CONSTRAINT:** Every design decision must be evaluated against this metaphor. If a proposed element feels like it belongs in a recording studio, a vintage synthesizer, or a sci-fi game, it is wrong for ENTROP. If it feels like it belongs in a scientific measurement facility, it may be correct.

ENTROP is not a synthesizer with knobs. It is a synthesizer with physics. The interface must communicate that the user is not building a sound — they are adjusting the conditions under which a physical or mathematical process unfolds, and then watching it happen through instrumentation.

The central metaphor is the **observation window**: a scientific facility for monitoring physical processes. Think of the control room of a deep-sea research submarine combined with the visualization terminal of a particle physics simulation. Dark. Precise. Heavy with information. Materials that suggest measurement, not music.

The user of ENTROP is an experimenter, not a sound designer in the conventional sense. The interface should reflect the difference between those two roles. A sound designer turns knobs until something sounds good. An ENTROP user sets parameters until the underlying process produces a phenomenon they find musically useful. That distinction is the conceptual foundation of every design decision in this document.

### 1.2 Three Engine Personalities — One Instrument

The three engines have fundamentally different characters. The visual design must communicate these differences at a glance, without labels. A user should be able to look at the STOCH section and feel unease. They should look at DIFFU and feel slow biological time. They should look at BUBBLE and feel weight and liquid pressure.

| Engine | Physical Domain | Emotional Quality | Visual Language |
|---|---|---|---|
| **STOCH** | Probability, chaos, Xenakis mathematics | Tension. Violence barely contained. Entropy pressing against a barrier. | Sharp angles, erratic motion, no symmetry, oscilloscope waveform with visible instability |
| **DIFFU** | Chemistry, biology, Turing patterns | Growth. Slow inevitable mutation. Organic patience. | Cell-like structures, branching forms, living dot patterns that shift slowly, biological green |
| **BUBBLE** | Fluid mechanics, pressure, resonance | Depth, mass, the cold of deep water under pressure. | Circular forms, translucent layers, upward drift motion, cold blue-white palette |

### 1.3 Forbidden Aesthetics — What This Is Not

> **AGENT CONSTRAINT:** The following aesthetics are explicitly prohibited. No design output should reference, approach, or suggest any of these. If a render contains any of these qualities, it fails regardless of other qualities.

| Forbidden Aesthetic | Why It Is Wrong |
|---|---|
| **Vintage analog warmth** — wood panels, cream-colored knobs, warm backlighting | ENTROP has nothing to do with vintage technology. The physics it models did not exist as instrument technology. |
| **Cyber/neon futurism** — glowing purple grids, sci-fi HUD overlays, lens flares, gaming RGB rainbow effects | The scientific aesthetic is precise and restrained, not theatrical. Information density is not achieved through glowing decoration. |
| **Minimalist white space** — clean empty UI, generous padding, soft shadows | ENTROP is mathematically dense. The interface should carry weight and complexity without being cluttered. |
| **Playful or approachable** — rounded forms, friendly typography, welcoming color temperature | This is a serious instrument for serious sound designers. The learning curve is intentional. The interface does not apologize for complexity. |
| **Glossy plastic** — any surface that looks like consumer electronics or a toy | Surfaces should read as technical/industrial materials: dark metal, composite, precision-machined. |
| **Generic EDM plugin** — layouts and aesthetics that could belong to any synthesizer | ENTROP must be visually unmistakable. If the design could be the UI for any other synthesizer, it is wrong. |

---

## 2. Colour System

> **AGENT CONSTRAINT — COLOR DISCIPLINE IS NAVIGATION LOGIC:** Color in ENTROP is not aesthetic — it is semantic. A red element always means STOCH. A green element always means DIFFU. A blue element always means BUBBLE. Violet means cross-engine (blend, master, global LFO). Breaking this rule destroys the user's ability to parse the interface quickly. This constraint applies to every element: waveforms, knob arcs, labels at active state, meters, borders, animation highlights, everything.

### 2.1 Background Surface Hierarchy

The interface lives almost entirely in darkness. Not pure black — a deep, slightly blue-tinted dark that reads as depth rather than void. Surface layers are distinguished by very subtle lightness steps, not by color changes. There are exactly five background levels. No intermediate values.

| Level | Hex | Name | Usage |
|---|---|---|---|
| Level 0 | `#0D0D14` | Void | Deepest background. Window border, outermost chrome, space between engine sections. |
| Level 1 | `#11111E` | Base | Primary plugin background. Where most UI sits. |
| Level 2 | `#1A1A2E` | Raised | Panel backgrounds. Engine section containers. Slightly elevated surfaces. |
| Level 3 | `#252540` | Float | Controls, input fields, knob disc areas. Hover states start here. |
| Level 4 | `#2E2E50` | Active | Focused control state. Selected panel. Active engine highlight. |

> **AGENT CONSTRAINT:** Do not invent intermediate background values. Do not use pure black (`#000000`) anywhere. Do not use warm-tinted darks. The five levels above are exhaustive — all surfaces use one of these.

### 2.2 Engine Accent Colours

Each engine has exactly one accent color. These colors appear exclusively in their respective engine's section and nowhere else.

| Engine | Hex | Name | Used For |
|---|---|---|---|
| **STOCH** | `#E84545` | Ember Red | Waveform display, active parameter highlights, chaos indicators, value arcs on STOCH knobs, STOCH section border in active state. |
| **DIFFU** | `#2DC653` | Spore Green | R-D grid visualization, concentration maps, biological pattern indicators, value arcs on DIFFU knobs, F/k crosshair, Pearson region labels at active state. |
| **BUBBLE** | `#38BDF8` | Foam Blue | Bubble cloud display, fluid depth meter, resonance indicators, value arcs on BUBBLE knobs, bubble particle color (deep range). |

> **AGENT CONSTRAINT — NEVER CROSS ENGINE BOUNDARIES:** `#E84545` must never appear in the DIFFU or BUBBLE sections under any condition, including hover states, active states, or error states. Same for all other engine colors. Use the neutral palette for any state that requires color change in another engine's section.

### 2.3 Neutral and Functional Palette

Everything that is not an engine-specific element or a live data readout uses only these neutral values.

| Hex | Name | Usage |
|---|---|---|
| `#F0EEF8` | Primary Text | Main labels, values, headings. Slightly warm white — not pure. All body text at full legibility. |
| `#8888AA` | Secondary Text | Unit labels (Hz, dB, ms), helper text, parameter names at rest state. |
| `#555570` | Disabled/Dim | Inactive controls, dimmed labels, unavailable states, ghost elements. |
| `#8B5CF6` | Global Violet | **Only** for cross-engine elements: blend controls, master section, global LFO, anything that spans or combines engines. Never inside individual engine sections. |
| `#3D3D60` | Separator | Hairline dividers between sections, grid lines, inactive meter segments. |

### 2.4 Colour Usage Rules — Quick Reference

```
Surface coloring:    Use only the 5 background levels (#0D0D14 → #2E2E50)
Active engine state: Engine accent color at full opacity for arc/border/indicator
Hover state:         Surface goes one level up (Level 2 → Level 3)
Disabled state:      #555570 for label, Level 1 surface, no accent color
Error state:         #E84545 only if error is in STOCH section. Otherwise: desaturated orange #C47832
Text on dark:        #F0EEF8 primary, #8888AA secondary. Never pure white.
Cross-engine:        #8B5CF6 exclusively
Dividers/hairlines:  #3D3D60
```

---

## 3. Typography

### 3.1 Type Hierarchy

Two typeface families. One for data and interface labeling — geometric, monospaced, measurably precise. One for editorial content (tooltips, preset names, documentation panels). No decorative fonts anywhere.

| Role | Font | Fallback | Size | Usage |
|---|---|---|---|---|
| **Parameter Values** | IBM Plex Mono | Courier New | 11–13px | All numeric readouts: frequency displays, percentage values, raw parameter numbers. Always monospaced so digits align vertically during live updates. |
| **Control Labels** | IBM Plex Sans Condensed | Arial Narrow | 9–11px | Parameter names below knobs and sliders. Engine section headers. All caps with letter-spacing +0.08em. |
| **Section Titles** | Space Mono | Courier New | 13–15px | Engine names (STOCH / DIFFU / BUBBLE). Uppercase, wide letter-spacing. Set as if stamped on technical equipment. |
| **Preset Names** | IBM Plex Serif | Georgia | 12–14px | Preset browser and patch names only. The only 'human' typeface in the UI — signals editorial content vs. technical controls. |
| **Tooltips / Help** | IBM Plex Sans | Helvetica | 11px | Hover descriptions, MIDI learn overlays. Regular weight, high legibility at small size. |

> **AGENT CONSTRAINT:** Never substitute decorative or display fonts for any of these roles. Never use a serif font for parameter labels or values. Never use a monospaced font for preset names. The typeface role assignments above are not suggestions — they are the type system.

### 3.2 Numeric Display Treatment

Parameter values are the most-read text in the interface. All numeric displays follow these rules without exception:

- **Monospaced font with fixed-width digits** — prevents layout shift when values update in real time
- **Decimal alignment** — always show the same number of decimal places regardless of value. Display `0.00` not `0`. Display `440.00 Hz` not `440 Hz`.
- **Unit labeling** — units displayed in Secondary Text color (`#8888AA`) at 60% opacity, to the right of the number. `440.00` in `#F0EEF8`, `Hz` in `#8888AA`.
- **Negative values** — shown in the engine's accent color at 60% saturation. Not the full accent color — muted variant.
- **Boundary values** — values at minimum or maximum extremes shown in the engine accent color at full saturation. This is the visual confirmation that a parameter has hit its limit.
- **Live update behavior** — values must update without layout reflow. Fixed-width monospace ensures this.

### 3.3 Label Formatting Rules

- Parameter labels: ALL CAPS, letter-spacing +0.08em, IBM Plex Sans Condensed, `#8888AA` at rest, `#F0EEF8` on hover/active
- Engine titles (STOCH / DIFFU / BUBBLE): Space Mono, uppercase, letter-spacing +0.15em, engine accent color
- Section sub-headers: IBM Plex Sans Condensed, small caps or uppercase, `#8888AA`
- Never use bold for parameter labels — the condensed weight at correct tracking is sufficient
- Never use italic anywhere except tooltip help text where emphasis is needed within a sentence

---

## 4. Layout Architecture

### 4.1 Overall Window Structure

> **AGENT CONSTRAINT:** The layout structure below is fixed. Agents do not propose alternative layouts or proportional systems. The five-zone horizontal structure is the authoritative spatial logic of ENTROP.

The plugin window is a single horizontal band divided into five vertical zones:

```
┌──────────┬───────────────────┬───────────────────┬───────────────────┬──────────┐
│  MASTER  │      STOCH        │      DIFFU        │      BUBBLE       │   FX     │
│ (global) │  Stochastic/      │  Reaction-        │  Fluid-Acoustic   │ Effects  │
│          │  GENDYN           │  Diffusion        │  Resonance        │          │
└──────────┴───────────────────┴───────────────────┴───────────────────┴──────────┘
  ~80px          ~340px              ~340px              ~340px           ~100px
```

**Minimum window size:** 1280 × 600px  
**Maximum window size:** 2560 × 1200px (2× Retina)  
**Resizable:** Yes — proportional scaling only. The entire layout scales as one unit. No responsive breakpoints.

The three engine columns are the dominant visual element. Master and FX strips are subordinate. The narrow blend strip between engine columns is described in section 6.3.

### 4.2 Internal Column Structure — Per Engine

Inside each engine column, the vertical structure follows this fixed order from top to bottom. This order must not change between engines — it creates visual rhythm across the three columns that lets users parse the instrument without looking at labels.

| Zone | Height | Contents |
|---|---|---|
| **Engine Header** | ~5% | Engine name (STOCH/DIFFU/BUBBLE) in Section Title typography. Engine accent color. Small status indicators (voice count, active state). |
| **Live Visualization** | ~35% | The primary live readout. Fills the top third of the working area. The most important visual element in each column. Full description in section 5. |
| **Primary Controls** | ~30% | **Three macro knobs** (Tier 1 — see Musicality Spec Section 9) as the first visible row, then 3–4 core physical parameters (Tier 2) beneath them. Large knobs throughout. Main interaction zone. |
| **Secondary Controls** | ~20% | Supporting parameters. Smaller controls. Distribution selectors, curve shapes, modulation targets. |
| **Modulation Strip** | ~10% | MPE assignment indicators, per-voice modulation depth meters. Thin horizontal strip at column base. Engine accent color meter fills. |

> **AGENT CONSTRAINT — HIERARCHY IS FIXED:** Do not reorder zones. Do not merge Primary and Secondary Controls. The visualization panel is always at the top. The modulation strip is always at the bottom. This order applies identically to all three engine columns.

### 4.3 Master Strip (Left Edge)

The Master strip is approximately 80px wide. It contains:
- ENTROP wordmark (vertical orientation, top of strip)
- Master Gain control (large, centered)
- Global preset snapshot buttons
- **Anchor display** — small text field showing the current archetype/preset origin name. If the user has modified parameters since loading, shows name with "(edited)" suffix in Secondary Text color (`#8888AA`).
- **Return to Anchor button** — single-click returns all parameters to the last-loaded preset/archetype state. Icon: `↩` symbol, 24×24px. Always visible. Tooltip: "Return to [preset name]".
- Master output meter (peak + RMS, two-bar format)
- Plugin version and format indicator at base

The Master strip uses Global Violet (`#8B5CF6`) for its active indicators and borders. It must visually read as belonging to no engine.

### 4.4 FX Strip (Right Edge)

The FX strip is approximately 100px wide. It contains:

- Hermite waveshaper control (H2 through H6 coefficient sliders, vertical)
- Reverb Size and Reverb Mix controls
- Output soft-limiter indicator (single LED-style indicator, illuminates at threshold)

The FX strip uses neutral palette only — no engine accent colors.

### 4.5 The Blend Morphing Strip

Between the three engine columns sits a narrow vertical strip (~40px wide) containing the engine blend controls. This strip uses Global Violet (`#8B5CF6`) exclusively. It is the only element in the interface that visually spans engine boundaries.

The blend strip contains:
- The Tri-Morph blend triangle control (section 6.3)
- Individual engine contribution level indicators (three micro-meters)

> **AGENT CONSTRAINT:** The blend strip must be visually obvious and physically easy to grab. It is the most critical control for live performance. It must not be smaller than 40px wide at base resolution. Consider a 1px Void-level (`#0D0D14`) hairline on each side of the strip to create visual separation from engine columns.

---

## 5. Live Visualizations — The Central Design Challenge

> **AGENT CONSTRAINT:** The three visualization panels are not decorative. They are the primary feedback mechanism for what the synthesis engines are doing. They must be designed to be informative at a glance during active playing, not just visually impressive at rest. Every visual behavior described below has a functional purpose — do not modify behaviors for aesthetic reasons.

> **AGENT CONSTRAINT — TECHNICAL BOUNDARY:** The live visualization panels receive data from the backend via a lock-free telemetry FIFO. The GUI thread reads from this FIFO — it never accesses audio voice state directly. Design specifications must not require data that is not available through this mechanism. Verify with the Backend agent what data is exposed before specifying visualization behavior.

### 5.1 STOCH Visualization — The Chaos Scope

**Type:** Modified oscilloscope showing the actual waveform generated by the GENDYN algorithm in real time.

**Core visual character:** Unstable. The waveform must not look smooth. The jagged, angular quality is mathematically accurate and must be visible, not smoothed out for aesthetics.

**Required visual behaviors:**

| Behavior | Description | Parameter Driven By |
|---|---|---|
| Breakpoint rendering | Waveform drawn as straight line segments between breakpoints — not smoothly interpolated. Angular, discontinuous. | Always visible |
| Time-axis jitter | As Freq Scatter increases, breakpoints visibly stutter and skip on the time axis. The waveform appears to slippage horizontally. | STOCH_FREQ_SCATTER (ID 3) |
| Ghost trace accumulation | Each complete waveform cycle leaves a ghost trace at 15% opacity. Ghosts decay over 3–4 cycles. Accumulation builds a visual probability cloud showing variation range. | Always active when audio running |
| Cauchy extreme state | At maximum Cauchy distribution intensity, the display reads as nearly broken — violent amplitude jumps create near-vertical line segments. | STOCH_DISTRIBUTION (ID 1) = Cauchy |
| Idle state | Static. No animation when audio is not running. Show last-rendered frame frozen. | Audio running = false |

**Color:** Waveform in `#E84545` (STOCH red) at full opacity. Ghost traces in `#E84545` at decreasing opacity steps: 15% → 10% → 6% → 3%.

**Target frame rate:** 60fps continuous while audio is running.

**Display dimensions:** Full width of STOCH visualization zone, full height of that zone (~35% of column height).

**Background:** Level 1 (`#11111E`) with a very subtle noise texture overlay (2% opacity, fine grain) to suggest CRT phosphor. Do not use animated scanlines or any moving background element.

### 5.2 DIFFU Visualization — The Living Grid

**Type:** Dual-layer display showing the Gray-Scott reaction-diffusion grid state AND an F/k XY pad for parameter control.

**Layout within the DIFFU visualization zone:**
- Upper portion (~60% of viz zone height): Cell grid display + spectral overlay
- Lower portion (~40% of viz zone height): F/k XY pad

**Grid display — required visual behaviors:**

| Behavior | Description |
|---|---|
| Cell rendering | 128-cell horizontal strip. Cell brightness proportional to V chemical concentration. High V = full `#2DC653` (Spore Green). Low V = Level 0 (`#0D0D14`). |
| Continuous animation | Cells update in real time as the R-D system evolves. Must animate smoothly — cell transitions are the primary sonic feedback. |
| Soliton region state | A few bright spots travel slowly across the strip. Slow drift, periodic. |
| Mitosis region state | Spots divide and replicate in real time. Population growth visible. |
| Chaos region state | The strip seethes. No stable structures. Rapid brightness fluctuation across all cells. |
| Spectral overlay | Above the cell strip, a frequency spectrum display shows the same V concentration data mapped to spectral amplitudes. User sees both chemical state and sonic consequence simultaneously. Use a second row of vertical bars, same color scheme. |
| Idle state | Static. Show last rendered state frozen. |

**F/k XY pad — required visual behaviors:**

| Element | Description |
|---|---|
| Pad background | A static Pearson phase diagram heatmap rendered as a blurred, low-contrast background layer. Use `#2DC653` at 15% opacity for bright regions, Level 0 for dark. This is the map of sonically interesting F/k territory. |
| Crosshair cursor | Minimal `+` symbol, 12px, `#2DC653` at full opacity. No circle, no grab handle. Pure crosshair. |
| Region labels | SOLITONS, MITOSIS, CORAL, CHAOS, LABYRINTHINE text printed in very faint uppercase (IBM Plex Sans Condensed, `#2DC653` at 20% opacity) directly on the pad surface, positioned at their correct F/k coordinates. |
| Region enter animation | When crosshair enters a named region: label brightens to 80% opacity and a soft glow expands from the label over 800ms, then fades back to 20%. This is the only 'delight' animation in DIFFU — everything else is live scientific data. |
| Idle state | Static. Crosshair at last position. No animation. |

**Pearson region coordinates for label placement:**

| Region | F | k |
|---|---|---|
| Frozen Field | 0.020 | 0.055 |
| Solitons | 0.062 | 0.061 |
| Mitosis | 0.028 | 0.062 |
| Coral / Worms | 0.054 | 0.063 |
| Labyrinthine | 0.037 | 0.060 |
| Chaos | 0.026 | 0.051 |

### 5.3 BUBBLE Visualization — The Pressure Tank

**Type:** 2D particle physics display showing the active bubble oscillator population as a fluid tank.

**Required visual behaviors:**

| Behavior | Description | Parameter Driven By |
|---|---|---|
| Bubble rendering | Each active bubble = a circle. Circle radius proportional to physical bubble radius (therefore inversely related to frequency). | bubble_population data |
| Drift velocity | Large circles drift slowly upward. Small circles drift faster. Reflects physical buoyancy. | bubble radius |
| Decay fade | Bubbles that have passed their decay envelope fade out (opacity decay) and disappear. No pop. | bubble.alive state |
| Color gradient | Bubble color transitions from deep `#1D6FA4` (low frequency, large, deep water) through `#38BDF8` (Foam Blue) to near-white `#E0F7FF` (high frequency, tiny, surface). Map linearly to Minnaert frequency. | bubble.freq_hz |
| Coupling network | When Coupling parameter > 0.4: faint connecting lines appear between spatially nearby bubbles. Lines at 10–15% opacity, `#38BDF8`. Lines appear/disappear dynamically as bubble positions change. | BUBBLE_COUPLING (ID 19) |
| Depth meter | Left edge of tank: a vertical bar showing current fluid pressure parameter. Higher pressure = bar fills higher. Bar color: `#38BDF8` at 60% opacity. | BUBBLE_PRESSURE (ID 15) |
| Injection animation | On MIDI note-on: a burst of small bubbles spawns at the bottom of the tank and rises. Count determined by note velocity (velocity 127 = maximum spawn burst). Animation duration ~200ms for initial burst appearance. | MIDI note-on event |
| Idle state | If no notes are active and population has decayed to zero: empty tank, depth meter still shows pressure parameter. |  |

**Tank boundaries:** The tank has a visible frame (Level 3 surface `#252540`, 1px inner border in `#38BDF8` at 30% opacity). The frame must read as a physical container, not just a UI box.

**Maximum visible bubbles:** 128 circles rendered simultaneously. At maximum polyphony and spawn rate, the display should read as a dense cloud, not individual particles. Performance budget: visualization must maintain 60fps with 128 circles active.

---

## 6. Control Design Language

### 6.1 Knobs — Standard Rotary Controls

> **AGENT CONSTRAINT:** Knobs in ENTROP are abstract graphic elements, not virtual replicas of hardware knobs. No plastic textures, no metal highlights, no photorealistic rendering. The knob is a precise geometric instrument indicator.

**Construction (all knobs follow this specification exactly):**

| Element | Specification |
|---|---|
| **Disc** | Flat circle, `#252540` (Level 3). No gradient. No bevel. No shadow. |
| **Range arc** | Thin 1px arc, `#2E2E50` (Level 4). 270° sweep from 7 o'clock to 5 o'clock. Always visible. |
| **Value arc** | Engine accent color, full opacity, same 1px weight as range arc. Starts at counter-clockwise limit. Fills clockwise to current value position. |
| **Marker line** | `#F0EEF8` (Primary Text), 2px wide, from center to disc edge. Indicates exact current value. |
| **Default tick** | A 1px tick mark on the range arc at the default parameter value position. `#555570` (Disabled). No other adornment at default. |

**Interactive states:**

| State | Visual Change |
|---|---|
| Default | As specified above |
| Hover | Disc brightens to Level 4 (`#2E2E50`). Label below illuminates to `#F0EEF8`. |
| Active/drag | Value arc brightens. Numeric readout appears floating above the knob (IBM Plex Mono, 11px, engine accent color). |
| Disabled | All elements at 40% opacity. Value arc replaced with `#555570`. |

- **Danger zone arc segment:** The extreme 15% of the range arc (approaching the counter-clockwise minimum or clockwise maximum, depending on the parameter's direction of danger) uses the engine accent color at 40% saturation instead of full saturation. This visual desaturation signals "you are entering territory where results become unpredictable." Applied only to parameters identified as dangerous in Musicality Spec Section 8.4. Specific parameters: STOCH Amp Scatter (high end), STOCH Freq Scatter (high end), BUBBLE Coupling (high end), DIFFU F/k at Chaos boundary (XY pad visual treatment only — not a standard knob).

**Knob sizes:**
- Primary controls (3–4 per engine): 36–44px diameter
- Secondary controls: 24–30px diameter
- Modulation depth indicators: 18–22px diameter

> **AGENT CONSTRAINT:** All knob sizes use one of the three sizes above. No custom per-control sizing. Consistency is more important than individual emphasis.

### 6.1a Macro Knobs — Visual Distinction

Macro knobs (Tier 1 — three per engine column) are visually distinct from parameter knobs:

- **Size:** Larger than primary parameter knobs — 44–52px diameter
- **Value arc:** Same engine accent color but **4px wide** instead of 1px — communicates higher-level control
- **Label:** MACRO label in IBM Plex Sans Condensed, ALL CAPS, with a subtle `◈` prefix glyph to distinguish from parameter labels
- **Background disc:** Level 2 (`#1A1A2E`) instead of Level 3 — sits slightly recessed to suggest "layer above"
- **Tooltip:** Shows which underlying parameters are being moved (e.g., "STABILITY → Amp Scatter + Freq Scatter")

Macro knobs do not have a default tick mark — they have no fixed default position because their position is derived from the current state of underlying parameters.
- **Modulation range overlay:** When a parameter has active LFO or MPE modulation assigned, a second arc appears on the knob at 30% opacity, in the engine accent color, showing the full modulated range as a dotted arc. This arc is always visible (not only on hover) when modulation is active. Implemented by Frontend from modulation depth data in the lock-free telemetry FIFO.

### 6.2 The F/k XY Pad (DIFFU Engine Only)

The XY pad is the most spatially expressive control in the plugin. Its design is specified in section 5.2 (visualization behaviors) and here (interaction specification).

**Interaction specification:**

| Interaction | Behavior |
|---|---|
| Click | Move crosshair to clicked position. Update F and k parameters immediately. |
| Drag | Crosshair follows pointer smoothly. Parameters update at pointer rate. |
| Double-click in region | Snap crosshair to nearest Pearson region coordinate. Parameters snap to exact F/k values. |
| MIDI-learn | Assignable. Full-scale MIDI maps to full F/k range. |
| MPE Pressure | Maps to Feed Rate F (increases reaction speed). |
| MPE Slide | Maps to Kill Rate k (shifts phase boundary). |

**Size:** Full width of DIFFU visualization zone lower portion. Height ~40% of visualization zone total height.

**Padding:** 8px interior padding so crosshair at extremes does not touch the pad border.

### 6.3 The Engine Blend Tri-Morph Control

> **AGENT CONSTRAINT:** The Tri-Morph is not three separate sliders. It is a single triangular field with one movable point. This design is fixed — do not substitute alternative blend control patterns.

**Construction:**

| Element | Specification |
|---|---|
| **Triangle shape** | Downward-pointing equilateral triangle. Top-left corner = STOCH (`#E84545`). Top-right corner = DIFFU (`#2DC653`). Bottom corner = BUBBLE (`#38BDF8`). |
| **Background fill** | Actual RGB gradient interpolating between the three engine accent colors. The blend point visually 'lives in' the color of its current sound. Gradient must be smooth and accurate. |
| **Triangle border** | 1px, `#8B5CF6` (Global Violet) at 60% opacity. |
| **Blend point** | Single dot, 8px diameter, `#F0EEF8` at full opacity. White so it's always readable against any background color. 1px `#0D0D14` outline for contrast. |
| **Corner labels** | STOCH / DIFFU / BUBBLE in IBM Plex Sans Condensed, 9px. Brightness proportional to engine contribution: full brightness = dominant, 30% brightness = silent. |

**Interaction:**

| Interaction | Behavior |
|---|---|
| Click anywhere in triangle | Moves blend point to clicked position instantly |
| Drag | Blend point follows pointer smoothly |
| Double-click corner | Snaps blend point to that corner (100% that engine) |
| Double-click center | Snaps to centroid (33.3% each engine) |
| MIDI-learn | Two MIDI CCs assignable: one for S↔D axis, one for D↔B axis |

**Placement:** In the blend strip between engine columns, centered vertically. The triangle should occupy most of the available height in the strip, constrained by the 40px width.

### 6.4 Sliders

Where sliders are used (FX strip waveshaper coefficients, individual engine level meters in blend strip):

- **Track:** 2px wide line, `#2E2E50` (Level 4)
- **Fill:** Engine accent color (or `#8B5CF6` for blend strip sliders), full opacity, from track start to current value
- **Thumb:** 6×14px rectangle, `#F0EEF8`, centered on current value position
- **Label:** IBM Plex Sans Condensed, 9px, `#8888AA` at rest

### 6.5 Value Displays and Readouts

Standalone numeric value displays (not inside a knob hover state):

- IBM Plex Mono, 11–13px
- Engine accent color for live-updating values
- `#F0EEF8` for static/reference values
- Unit label in `#8888AA` at 60% opacity to the right
- Background: Level 3 (`#252540`), 4px border radius, 4px horizontal padding

---

## 7. Motion and Animation Principles

> **AGENT CONSTRAINT:** Every animation in ENTROP must earn its presence. Animation is information, not decoration. If removing an animation would make the interface harder to understand or use, it belongs. If removing it would make no functional difference, it is forbidden.

### 7.1 Permitted Animations — Full Specification

| Element | Animation Type | Duration/Rate | Trigger | Data Source |
|---|---|---|---|---|
| STOCH waveform | Continuous real-time redraw | 60fps perpetual | Audio engine running | GENDYN waveform buffer |
| STOCH ghost traces | Opacity decay per waveform cycle | 3–4 cycles to 0% | Continuous during audio | Cycle count |
| DIFFU grid cells | Continuous fade/brighten per cell | 60fps perpetual | Audio engine running | V concentration array |
| DIFFU region label glow | Soft expand then fade | 800ms total | Crosshair enters Pearson region | Crosshair position |
| BUBBLE particle drift | Per-bubble upward movement | Per-bubble lifetime | Active bubble exists | bubble.alive + bubble.radius |
| BUBBLE injection burst | Fast spawn at tank bottom | 200ms appearance | MIDI note-on | Note velocity |
| BUBBLE particle fade | Opacity decay | bubble.decay_rate | bubble.alive → false | bubble.age |
| BUBBLE coupling lines | Appear/disappear | Instant on threshold | BUBBLE_COUPLING > 0.4 | Coupling parameter |
| Knob value readout | Fade in on hover/drag | 150ms ease-out | Pointer enter | — |
| Knob value readout | Fade out on release | 200ms ease-out | Pointer leave / drag end | — |
| Engine blend triangle | Background gradient shift | Real-time | Blend control drag | Engine blend values |
| Depth meter fill | Height change | Real-time | BUBBLE_PRESSURE change | Pressure parameter |

### 7.2 Forbidden Animations — Hard No-Go List

> **AGENT CONSTRAINT:** These are absolute prohibitions. No exception exists regardless of aesthetic justification.

- **No idle animations** when the plugin is not processing audio. Nothing moves when no sound is being generated. Visualizations freeze at last-rendered state.
- **No loading indicators** of any kind. Operations that take time (preset load, grid reset) update instantly or not at all.
- **No decorative particle effects** that are not directly driven by synthesis engine data.
- **No glow blooms or light trails** that are not tied to specific parameter values.
- **No transition animations** between panels, sections, or parameter states. The interface is static in structure — only live readouts move.
- **No bounce, spring, or elastic easing** on any control. Controls respond linearly and immediately.
- **No looping background animations** (animated gradients, slow texture panning, breathing effects). Background is static.
- **No celebration or feedback animations** on preset load, save, or other non-audio actions.

---

## 8. Technical Requirements

### 8.1 Plugin GUI Rendering Pipeline

ENTROP uses the CLAP plugin format's GUI extension. The plugin window is rendered by the plugin itself — not by a web browser or a separate app.

| Technology | Available | Notes |
|---|---|---|
| **JUCE Framework (C++)** | Yes — primary option | Cross-platform widget toolkit. Custom component rendering via `juce::Component` and `juce::Graphics`. Preferred for portability. |
| **Metal / Core Animation (macOS)** | Yes — advanced option | Direct GPU access for the live visualizations. Delivers exceptional performance for the animated panels. Requires Objective-C/Swift bridge. |
| **OpenGL via JUCE** | Yes — legacy option | Available but deprecated on macOS. Viable as fallback. |
| **HTML/CSS/Web** | No | Not available in a native plugin context. SVG, CSS animations, and HTML canvas cannot be used directly. |
| **Embedded browser** | Possible but not recommended | Adds significant binary size and complexity. Reserve as fallback only. |

> **AGENT CONSTRAINT — FRONTEND INTEGRATION:** The live visualization panels (STOCH waveform, DIFFU grid, BUBBLE tank) receive data from the backend via a lock-free telemetry FIFO. The Frontend agent reads from this FIFO at display refresh rate. It never accesses audio voice state directly. This is a hard architectural constraint — design specifications must not require data paths that bypass this model.

### 8.2 Asset Delivery Format

All design assets must be delivered in formats a C++ developer can integrate without conversion.

| Asset Type | Format | Specification |
|---|---|---|
| Icons and static UI elements | **SVG** | JUCE renders SVG directly via `juce::Drawable`. No PNG icons. Clean paths, no embedded raster images, no unsupported SVG features. |
| Background textures | **PNG at 2×** | Retina minimum. Keep under 200KB each. Lossless only — no JPEG. |
| Fonts | **OTF or TTF** | Embedded in plugin binary. Confirm license allows commercial software embedding before selection. |
| Colour values | **Design tokens file** | A flat list of all named hex values. Every color in the design must have a name and a hex. No unnamed colors anywhere in any deliverable. |
| Component specifications | **Figma** | Exact pixel dimensions, padding values, color references for every interactive state (default, hover, active, disabled). Developer inspect panel must show all values. |
| Animation specifications | **Written spec** | For each animated element: trigger, duration, easing, data source, target visual property. Figma prototyping is supplementary — the written spec is authoritative. |

### 8.3 Sizing and Resolution Specifications

| Specification | Value | Notes |
|---|---|---|
| Base window size | 1280 × 600px | Minimum usable size. All layouts correct at this size. |
| Maximum window size | 2560 × 1200px | 2× Retina equivalent. All assets correct at this scale. |
| Resizable | Yes — proportional | Layout proportions fixed, whole window scales as one unit. |
| Retina support | Required | All raster assets at 2× minimum. Prefer SVG for all icons and UI graphics. |
| Visualization frame rate target | 60fps | STOCH, DIFFU, BUBBLE panels must achieve this on Apple Silicon MacBook Pro. |
| Physical/logical pixel API | Required | CLAP GUI extension exposes physical and logical pixel sizes. Frontend must use the correct API to avoid blurry rendering on Retina displays. |

### 8.4 Spacing and Grid System

All spacing uses a 4px base unit. Permitted spacing values: 4, 8, 12, 16, 24, 32, 48px. No intermediate values.

| Context | Value |
|---|---|
| Interior padding (control areas) | 8px |
| Gap between controls within a section | 8–12px |
| Gap between sections within a column | 16px |
| Column border padding | 12px |
| Knob label spacing below knob | 4px |
| Section title margin below | 8px |

### 8.5 Border Radius System

| Element | Radius |
|---|---|
| Engine column container | 4px |
| Control surface (knob area background) | 4px |
| Value display readout | 4px |
| XY pad | 4px |
| BUBBLE tank | 4px |
| Any button | 3px |
| Blend strip | 0px (straight edges for technical precision aesthetic) |

---

## 9. Graphic Agent Workflow — Antigravity Constraints

> **This section is for the Graphic Design agent only.** Backend, Frontend, and Test agents do not need to read this section.

### 9.1 The Spec-First Principle

> **AGENT CONSTRAINT:** You are not primarily an image generator. You are an art director, UI system designer, and render strategist operating under strict constraints. Your primary outputs are specifications, tokens, and component rules. Renders are milestone verification artifacts — not the product.

**The mantra:**
```
Renders are proof of direction.
Specifications are the truth.
Implementation follows specifications.
```

Do not generate renders before the relevant specification is complete and stable. A render that contradicts the specification is worthless and must be rejected regardless of visual quality.

### 9.2 Render Budget Discipline

Antigravity image generation operates under these constraints:

| Constraint | Reality | Rule |
|---|---|---|
| Single-image throughput | Often 1 render per cycle | Every render must have exactly one clearly stated goal |
| Server lock/cooldown risk | May occur after 1–3 renders | Have offline spec work planned as immediate fallback |
| Style drift | Each generation may deviate | Use approved render as anchor. State what must not change in every prompt. |
| Poor component isolation | Models produce scenes, not precise assets | Use renders for direction confirmation only, not for extractable asset production |
| Repeatability | Same prompt ≠ same result in another session | Extract rules and tokens from approved renders immediately — never rely on re-generating the same image |

**Render budget per session:** Decide at session start. Maximum 3 render attempts per session. If two renders have not confirmed the target goal, stop and return to specification work.

### 9.3 Priority Order for Renders

Renders must occur in this sequence. Never skip to a later stage:

1. **Mood frame / key art** — one screen, confirms philosophical tone and color temperature
2. **Layout wire-mood frame** — confirms proportions: Master / STOCH / DIFFU / BUBBLE / FX
3. **One engine hero panel** — confirms look of a single column at full detail (recommended: STOCH first, then DIFFU)
4. **Full assembled UI mockup** — only after tokens and component rules are locked
5. **Marketing render** — only after implementation direction is confirmed

> **AGENT CONSTRAINT:** Never generate a marketing render before the implementation direction is confirmed. Never generate a full UI mockup before component rules are stable. Rushing to a "complete picture" before system is defined produces beautiful images that cannot be implemented.

### 9.4 Prompt Contract — Required Elements in Every Render Prompt

Every image generation prompt for ENTROP must include all of the following:

```
LAYOUT INVARIANTS:
- State the fixed layout (header / triptych 3 engine panels / footer or partial view)
- State exact proportional relationships

STYLE ANCHORS:
- "dark premium pro-audio scientific instrument UI"
- "triptych layout with color-coded engine sections"
- "high contrast labels, controlled accent lighting, no gaming RGB"
- Reference the approved mood frame if one exists

COLOR SPECIFICS:
- STOCH section: #E84545 ember red
- DIFFU section: #2DC653 spore green  
- BUBBLE section: #38BDF8 foam blue
- Background: dark near-black with blue cast, #11111E base

TYPOGRAPHY DIRECTION:
- "technical monospaced readouts, condensed sans-serif labels"
- "all caps parameter labels, scientific precision aesthetic"

THIS ITERATION TESTS:
- State exactly ONE primary goal for this render

DOES NOT CHANGE FROM APPROVED:
- List what must stay identical to the approved reference

FORBIDDEN:
- No vintage analog aesthetics
- No warm backlighting or wood textures
- No neon futurism or gaming RGB
- No sci-fi HUD overlays or lens flares
- No flat mobile app UI
- No toy plastic surfaces
```

### 9.5 Post-Render Analysis Protocol

After every render, before proceeding to any next step:

1. **Evaluate against brief criteria** — list every element that matches the brief, every element that contradicts it
2. **Extract rules** — what visual decisions in this render should become permanent rules?
3. **Mark for implementation** — what from this render can the Frontend agent use directly? What cannot?
4. **Update the Visual Decision Log** — document what was approved, what was rejected, why
5. **State next action clearly** — either proceed to spec extraction, or state what must be corrected in the next render

### 9.6 Lock / Cooldown Fallback Protocol

If server lock occurs during a render session, immediately switch to non-render work:

- Update design tokens file with decisions made so far
- Write component specification for the element being designed
- Annotate any existing renders with implementation notes
- Prepare the prompt for the next render attempt (so it is ready when server recovers)
- Never attempt random or exploratory renders while waiting — that wastes the remaining budget

### 9.7 Required Documents the Graphic Agent Maintains

| Document | Location | Content |
|---|---|---|
| Visual Decision Log | `docs/design/decision_log.md` | What was approved and why, what was rejected and why. One entry per render session. |
| Render Budget Log | `docs/design/render_log.md` | Session date, render count, goal, result (approved/rejected/needs-annotation), lock events. |
| Style Guardrails | `docs/design/style_guardrails.md` | Active forbidden motifs, drift warning signs seen in renders, approved keeper rules. |
| Handoff Spec Sheets | `docs/design/handoff/[component].md` | One file per component: dimensions, states, layer names, color tokens, behavior notes. |

---

## 10. Style Constraints — Visual QA Pass/Fail Criteria

> **This section is used by both the Design agent and the Test agent.** It defines the objective criteria for whether a design output, render, or implemented UI component passes review.

### 10.1 Pass/Fail Criteria Table

| Criterion | Pass | Fail |
|---|---|---|
| **Triptych readability** | 3 engine sections immediately identifiable at glance | Sections merge, blend together, or require reading labels to distinguish |
| **Engine color identity** | Red = STOCH only, Green = DIFFU only, Blue = BUBBLE only — no exceptions | Any engine accent color appearing outside its section |
| **Background hierarchy** | 5 distinct depth levels readable, darkest elements feel like depth not void | Flat surface, insufficient contrast between levels, warm-tinted backgrounds |
| **Label legibility** | Parameter labels readable at normal viewing distance, white-on-dark, correct font | Low contrast, wrong typeface, labels smaller than 9px |
| **Premium instrument feel** | Reads as serious pro-audio scientific tool | Gaming, toy, consumer electronics, or generic synth aesthetic |
| **Glow discipline** | Accent lighting supports focus on active elements | Glow dominates, creates visual noise, or exists without data justification |
| **Implementability** | Elements decompose into distinct panels, components, layers that a developer can implement | Painterly scene where UI elements cannot be separated or reconstructed |
| **Animation discipline** | All visible motion tied to live audio data | Any idle animation, decorative motion, or motion without data source |
| **Typography system** | Correct font family for each role, consistent sizing, correct color hierarchy | Mixed font families, inconsistent sizing, text at wrong color |
| **Blend strip visibility** | Tri-morph control immediately findable, easy to grab | Blend strip hidden, too narrow, visually merged with engine columns |

### 10.2 Hard Rejection Criteria

These are automatic failures regardless of any other quality:

- Any vintage analog element (wood, cream, warm amber backlighting)
- Any gaming RGB aesthetic (rainbow effects, excessive glow, neon purple HUD)
- Any engine accent color appearing in another engine's section
- Any animation when audio is not running
- Any flat white or near-white background anywhere in the main UI
- Any element smaller than the 4px spacing grid allows
- Any font not in the specified type hierarchy used for labels or values

---

## 11. Preset Browser and Naming Language

### 11.1 Preset Browser Layout

The preset browser is a secondary panel — not always visible. When open, it slides in from the left edge and partially overlays the Master column. It does not replace the main UI.

**Browser elements:**
- Category filter tabs at top (STOCH / DIFFU / BUBBLE / BLEND / ALL)
- Scrollable preset list: preset name (IBM Plex Serif, 12px) + engine tag (small colored dot, engine accent color) + brief descriptor (IBM Plex Sans, 10px, `#8888AA`)
- Search/filter input at top
- Favorite/unfavorite star per preset
- Current preset display in Master strip (always visible, even when browser is closed)

### 11.2 Preset Naming Convention

Preset names should feel like scientific specimen labels or field observation notes. Not like electronic music genre labels. The names communicate that sounds come from real physical processes.

| Good — Scientific/Physical | Bad — Genre/Trope |
|---|---|
| Rayleigh-Taylor Instability | Dark Matter Bass |
| Nucleation Cascade | Sci-Fi Laser |
| Post-Critical Decoherence | Glitch Pad |
| Dense Phase Boundary | Neurofunk Lead |
| Minnaert Cloud Formation | Underwater Ambience |
| Xenakis Limit State | Stochastic Noise |
| Gray-Scott Soliton | Evolving Texture |
| Viscous Damping Region | Deep Pad |
| Kelvin-Helmholtz Shear | Noise Burst |
| Bifurcation Point | Tension Builder |

> **AGENT CONSTRAINT:** The preset naming system is part of the instrument's identity. It communicates that sounds come from real physical processes, not genre conventions. Propose names in this register. Any preset name that could belong to a standard synthesizer is wrong for ENTROP.

### 11.3 Preset Category Tags

| Tag | Color | Usage |
|---|---|---|
| STOCH | `#E84545` at 80% opacity | Presets dominated by STOCH engine |
| DIFFU | `#2DC653` at 80% opacity | Presets dominated by DIFFU engine |
| BUBBLE | `#38BDF8` at 80% opacity | Presets dominated by BUBBLE engine |
| BLEND | `#8B5CF6` at 80% opacity | Multi-engine blend presets |
| INIT | `#8888AA` | Init/starting point patches |

---

## 12. Deliverable Checklist — Phased Output Requirements

This section defines what the Design agent must deliver in each phase, in priority order. Orchestrator uses this to create task briefs.

### Phase 0 — Concept (Before Any Production Work)

- [ ] **Design Intent Summary** — 1–2 pages: what ENTROP must communicate visually, what it must not. Written document, no renders required.
- [ ] **Color tokens file** — all named hex values from section 2, usage rules, as a flat design tokens JSON.
- [ ] **Typography matrix** — all five roles, fonts, fallbacks, sizes, in a table format ready for frontend integration.
- [ ] **Proportional layout schema** — ASCII or diagram specifying zone proportions (Master/STOCH/DIFFU/BUBBLE/FX), column internal structure, blend strip position.
- [ ] **Mood frame render (1 render)** — one full-width image confirming tone, color temperature, and scientific aesthetic. Goal: validate philosophy, not detail.

### Phase 1 — Core UI (Parallel with Developer Phase 1)

- [ ] **Knob component specification** — SVG with separate named layers: `knob_disc`, `knob_range_arc`, `knob_value_arc`, `knob_marker`. All states documented (default/hover/active/disabled).
- [ ] **STOCH column full design** — complete Figma layout at 1280px base, all control states, all label positions, placeholder oscilloscope (no live data — static mockup).
- [ ] **Component state matrix** — for every control type (knob, slider, XY pad, button, value display): default / hover / active / disabled states. Exact hex for each state.
- [ ] **Icon set SVG** — all transport/utility icons: play, stop, MIDI, preset, settings, reset. 24×24px grid. Stroke-based, no fills.
- [ ] **Spacing and grid spec** — all spacing values, border radii, padding. Must match section 8.4 and 8.5.
- [ ] **STOCH column hero render (1 render)** — confirms look of STOCH column with oscilloscope region at medium chaos state.

### Phase 2 — Visualization Panels (Parallel with Developer Phase 2)

- [ ] **STOCH oscilloscope spec** — static mockups at three chaos states (low/medium/maximum Cauchy). Written description of ghost trace behavior, decay rate, grid reference lines.
- [ ] **DIFFU grid panel design** — static mockups at four Pearson regions (Solitons, Mitosis, Coral, Chaos). F/k XY pad design with Pearson heatmap background detail.
- [ ] **DIFFU grid animation spec** — written description with exact parameters: cell update rate, brightness range, transition style, region label glow timing.
- [ ] **BUBBLE tank panel design** — static mockups at three fluid presets (Water, Magma, Plasma). Shows bubble size distribution, depth meter, coupling line examples.
- [ ] **BUBBLE motion spec** — drift velocities by size, injection burst frame sequence, coupling line appearance threshold and opacity rules.
- [ ] **Full DIFFU column hero render (1 render)** — confirms look of DIFFU column with grid in Coral region and XY pad visible.

### Phase 3 — Full Interface + Marketing

- [ ] **Complete assembled plugin window** — Figma at 1280×600 and 2560×1200. All three engines in representative states.
- [ ] **Blend Tri-Morph design** — all blend positions: 100% STOCH, 100% DIFFU, 100% BUBBLE, 33% center.
- [ ] **Preset browser panel design** — full layout, category tabs, list items, search input, all states.
- [ ] **Full UI assembled render (1 render)** — confirms complete instrument look with all engines visible.
- [ ] **Plugin marketing render (1 render)** — assembled plugin composited in a DAW environment (Bitwig or Reaper). Marked as marketing-only — not a UI spec source.
- [ ] **Splash/about panel** — minimal: ENTROP wordmark, version, three engine names.

---

## Appendix A — Design Tokens Reference

All colour values as named tokens for developer integration:

```json
{
  "colors": {
    "background": {
      "void":   "#0D0D14",
      "base":   "#11111E",
      "raised": "#1A1A2E",
      "float":  "#252540",
      "active": "#2E2E50"
    },
    "engine": {
      "stoch":  "#E84545",
      "diffu":  "#2DC653",
      "bubble": "#38BDF8"
    },
    "global": {
      "violet":    "#8B5CF6",
      "separator": "#3D3D60"
    },
    "text": {
      "primary":   "#F0EEF8",
      "secondary": "#8888AA",
      "disabled":  "#555570"
    },
    "bubble_gradient": {
      "deep":    "#1D6FA4",
      "mid":     "#38BDF8",
      "surface": "#E0F7FF"
    }
  },
  "spacing": {
    "xs": "4px",
    "sm": "8px",
    "md": "12px",
    "lg": "16px",
    "xl": "24px",
    "xxl": "32px"
  },
  "radius": {
    "container": "4px",
    "control":   "4px",
    "display":   "4px",
    "button":    "3px",
    "strip":     "0px"
  },
  "typography": {
    "values":      { "family": "IBM Plex Mono",            "fallback": "Courier New",  "size": "11-13px" },
    "labels":      { "family": "IBM Plex Sans Condensed",  "fallback": "Arial Narrow", "size": "9-11px", "transform": "uppercase", "tracking": "+0.08em" },
    "titles":      { "family": "Space Mono",               "fallback": "Courier New",  "size": "13-15px", "transform": "uppercase", "tracking": "+0.15em" },
    "presets":     { "family": "IBM Plex Serif",           "fallback": "Georgia",      "size": "12-14px" },
    "tooltips":    { "family": "IBM Plex Sans",            "fallback": "Helvetica",    "size": "11px" }
  }
}
```

---

## Appendix B — Quick Reference for Frontend Agent

When implementing, refer to these bindings:

| Visual Element | Parameter ID | Data Source |
|---|---|---|
| STOCH waveform color | — | Always `#E84545` |
| STOCH ghost trace opacity | — | Decays 15% → 10% → 6% → 3% per cycle |
| DIFFU cell brightness | — | V concentration array, 0.0–1.0 maps to `#0D0D14` → `#2DC653` |
| DIFFU XY pad crosshair | ID 7 (F), ID 8 (k) | Current parameter values |
| BUBBLE circle radius | — | Proportional to `bubble.radius` (max 40mm = 0.04m in SI units) |
| BUBBLE circle color | — | Frequency → color gradient (see section 5.3) |
| BUBBLE coupling lines | ID 19 | Appear when value > 0.4 |
| BUBBLE depth meter fill | ID 15 | Linear map: 50000–200000 Pa → 0%–100% height |
| Blend triangle gradient | ID 20, ID 21 | Engine contribution weights → RGB interpolation |
| Knob value arc color | — | Engine accent color of the knob's engine section |
| Master section accents | — | Always `#8B5CF6` |
| FX section | — | Neutral palette only |

---

*ENTROP · Visual Identity & UI Design Brief v0.1 · Authoritative visual specification*
