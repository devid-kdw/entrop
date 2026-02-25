# ENTROP Musicality & Usability Specification v0.2

**Purpose**  
This document defines musical success criteria, usable patch workflow expectations, failure modes, and in-context testing rules for **ENTROP** (STOCH / DIFFU / BUBBLE) so the synth evolves as a **musically useful instrument**, not only a technically valid experimental system.

**Status:** Working spec — v0.2, expanded from v0.1 after architecture review  
**Applies to:** Developer plan, DSP implementation, UI/UX decisions, preset design, QA, agent workflows  
**Audience:** Orchestrator agent, Backend agent, Frontend agent, Graphic agent, Test agent, Human owner (creative/product director)

**Changelog v0.1 → v0.2**
- Sections 4.1–4.3: Added deep engine-specific DSP guardrails, parameter mapping tables, and musical role archetypes with concrete parameter targets
- Section 5: Added Stage F (Iterative Refinement Loop) and anti-pattern root cause analysis
- Section 6: Added FM-19 through FM-28 (new failure mode categories: polyphony, modulation, blend-specific, preset system)
- New Section 8.6: Perceptual loudness normalization specification
- New Section 9: Macro Control Architecture — how to expose complex parameters as musically usable surfaces
- New Section 10: DSP-level musical constraints — what the audio engine must guarantee regardless of parameter state
- New Section 11: Per-engine parameter taper and range specification
- New Section 12: Polyphony musical behavior contract
- New Section 13: Preset architecture specification (expanded)
- Section 14 (prev. 12): Immediate next actions expanded

---

## 1. Core Design Goal

ENTROP should feel like an **explorable avant-garde instrument** that still provides:

- **repeatable musical outcomes**
- **controllable behavior**
- **meaningful performance expression**
- **fast access to usable sounds**
- **predictable zones of behavior** (even when internally complex)

### 1.1 Non-Goal

ENTROP is **not** required to behave like a conventional subtractive synth or "bread-and-butter preset machine."  
It **is** required to let a user reliably reach useful sonic results without fighting the system.

### 1.2 The Core Tension This Document Manages

Each engine in ENTROP is built on a system that is, by mathematical definition, capable of producing outputs that are musically useless — pure stochastic noise, uniform spectral wash, a cloud of frequencies with no coherence. This is not a bug; it is the boundary condition of each system.

The purpose of this spec is not to remove access to those boundary states. It is to ensure that:

1. **The path from nowhere to somewhere is navigable**, even for a new user
2. **The most musically rich regions of each parameter space are wider than the most hostile regions**
3. **The transition between musical and non-musical states has friction** — it does not happen accidentally
4. **Recovery from a collapsed state is always one gesture away**

Every DSP, UI, and preset decision in this project must be evaluated against this tension.

---

## 2. Definition of Musical Utility (Global)

A sound/result in ENTROP is considered **musically useful** if it satisfies most of the following:

1. **Intent readability**  
   The user can hear what changed when they move a control (cause/effect is perceptible).

2. **Performance stability**  
   The result remains controllable enough for phrase playing, modulation, or arrangement placement.

3. **Mix plausibility**  
   The sound can plausibly fit in a composition after normal mixing steps (EQ/comp/reverb), without requiring "rescue surgery."

4. **Expressive gradient**  
   There are audible, interesting changes across parameter travel; not just dead zones + chaos spikes.

5. **Recallability**  
   A user can get back to a similar family of sound via presets/archetypes/regions/snaps.

### 2.1 Musical Utility Is Not the Same as Pleasantness

It is important to be explicit: a sound does not need to be pleasant, warm, or conventionally musical to be *useful*. A sound is useful if it can be placed with intent inside a composition. This includes:

- Noise textures that function as a bed under melodic content
- Chaotic glitch events used as accent elements in rhythmic patterns
- Destabilized drones that create tension before a transition
- Harsh spectral masses used in industrial or experimental contexts

The test is not "is this a nice sound?" The test is: **can a producer with a specific intent reach this result, use it, and come back to it?**

---

## 3. Global Musical Success Criteria

These criteria apply to **all engines** and blended states.

### 3.1 Time-to-Usable-Sound (TTUS)

ENTROP must support a fast path from init/archetype to musically useful result.

#### Targets

| User Profile | Target TTUS | Path Allowed |
|---|---|---|
| Experienced (knows the engine concepts) | 15–60 seconds | Free exploration from init |
| Intermediate (electronic music background, no physics knowledge) | 60–120 seconds | Preset/archetype navigation |
| New user (minimal synthesis experience) | 120–180 seconds | Guided preset + documented macros |

#### Pass Condition

In guided internal testing, at least **4/5 testers** can produce a usable sound within the target window using only documented controls.

#### TTUS Implementation Requirements

To meet these targets, the following must be true at every phase milestone:

- **Init patches are not silence or unstructured noise.** Every engine's init state produces a recognizable, non-hostile sound with clear character.
- **The first three visible controls on each engine section produce audible, interesting changes.** Layout order must reflect musical impact order.
- **Preset names communicate sonic expectation accurately.** A user who selects "Nucleation Cascade" should not get a thin sine wave.

---

### 3.2 Sweet-Spot Density

"Sweet-spot density" = percentage of parameter space that gives musically plausible results.

#### Requirement

Main controls/macros should have **broad useful travel**, especially in the middle ranges.

#### Guidelines

- Core control ranges should avoid "all action in first 10%" or "only one tiny sweet spot"
- If a control is intentionally dangerous/chaotic, this must be:
  - Visually signposted in the UI (see Design Brief: Controlled Extremes)
  - Sonically smoothed at the transition point where possible
  - Recoverable via reset/snap/safe zone

#### Measurement Protocol

For each primary engine control, during QA:
1. Divide the full control range into 10 equal segments
2. At the center value of each segment, evaluate whether the output is musically usable (binary: yes/no)
3. Record the number of segments that pass
4. **Minimum acceptable sweet-spot density: 6/10 segments pass for primary controls, 4/10 for secondary controls**

This is not a strict requirement for every edge-case parameter — it is a floor for the controls that appear most prominently in the UI.

---

### 3.3 Loudness & Gain Consistency

Exploration must not be ruined by extreme gain jumps.

#### Requirements

- Parameter changes must not create frequent accidental clipping or large dropouts during normal exploration
- Engine blend changes must remain manageable without constant manual gain compensation
- The plugin output must never exceed 0 dBFS without explicit user action (a dedicated saturation/drive control that is visually distinct from normal controls)

#### QA Target

- Typical exploratory parameter moves (any single primary control through its full range) produce no more than **±6 dB perceived loudness variation**
- Engine blend movement through the full tri-morph range produces no more than **±4 dB perceived loudness variation**
- Any high-risk gain behavior must be intentional and documented in the preset notes for that patch

#### DSP Implementation Note

The audio engine must include a **lookahead soft-limiter at the final output stage** — not as a creative effect, but as a safety floor. Parameters: 3–5ms lookahead, threshold at −1 dBFS, soft knee 6 dB, transparent release (≥300ms). This limiter is not accessible to the user and does not appear in the UI. Its purpose is to prevent equipment damage and ear fatigue during exploration, not to shape the sound.

---

### 3.4 Predictability Without Killing Discovery

ENTROP should preserve surprise, but the user must still feel in control.

#### Requirements

- The user can identify stable "zones" and "behaviors" by name and by ear
- Preset archetypes / region snaps / named modes act as **anchors** that are reliably reachable
- Extreme outcomes must be reachable, but not unavoidable

#### The "Musical Confidence Loop"

A user who is confident in an instrument tends to explore more aggressively. A user who is afraid of losing a good sound explores timidly or stops. ENTROP's design must support the confidence loop:

```
Find anchor → Explore from anchor → Hear something interesting →
Return to anchor → Explore further → (repeat)
```

Every element of the design — preset names, UI layout, parameter mapping, init patch — must support this loop by making the return step trivially easy. The "return to anchor" gesture should never cost more than two actions.

---

## 4. Engine-Specific Musical Success Definitions

### 4.1 STOCH — Musical Success Definition

STOCH is successful when it provides **controlled stochasticity** with audible character and playable intent.

#### Musical Success (STOCH)

A STOCH patch is musically successful when it provides one or more of:

- **Tonally anchored instability** — alive, shifting, but not random gibberish
- **Rhythmic unpredictability with usable pulse implication**
- **Textural richness that remains shapeable**
- **Pitch/event variance that sounds intentional**, especially with quantization/snap active

#### What "Good STOCH Behavior" Feels Like

- Chaos changes are audible, but not all-or-nothing
- Jitter/variance modifies feel rather than instantly destroying identity
- Quantization/snap can recover or impose harmonic structure when needed
- User can move between stable / semi-stable / wild in a musically legible way

#### STOCH Musical KPIs

- Quantization/snap modes produce clearly recognizable tonal stabilization
- At least 5 parameter combinations yield distinct, named musical roles (see archetype table below)
- Each musical role is reachable from the init patch within 60 seconds

#### STOCH Musical Archetype Targets

| Role | Pitch Snap | Amp Scatter | Freq Scatter | Distribution | Segments | Character |
|---|---|---|---|---|---|---|
| Unstable Drone | 0.0–0.2 | 0.3–0.5 | 0.1–0.3 | Gaussian | 4–6 | Slow-moving, dark, pitchy but alive |
| Xenharmonic Lead | 0.7–1.0 | 0.1–0.3 | 0.2–0.4 | Uniform | 6–10 | Quantized to microtonal scale, articulated |
| Glitch Accent | 0.0 | 0.7–1.0 | 0.6–1.0 | Cauchy | 3–5 | Percussive events, violent jumps, unpredictable |
| Texture Bed | 0.3–0.6 | 0.4–0.6 | 0.3–0.5 | Logistic | 8–12 | Dense mid-range texture, slow evolution |
| Noise Burst | 0.0 | 0.8–1.0 | 0.8–1.0 | Cauchy | 3–4 | Full spectral noise, shaped by barrier elasticity |

#### STOCH-Specific DSP Guardrails (Musical)

**Cauchy distribution amplitude cap.** The heavy tail of the Cauchy distribution can produce sample values of arbitrary magnitude. The elastic barrier handles this structurally, but the DSP implementation must additionally apply a **tanh soft saturation on STOCH output before the engine mixer** — not a hard clip, a smooth saturation. This preserves the violent character of Cauchy while preventing output spikes that would make the engine unusable in a mix context. Recommended: `tanh(x / 0.8) * 0.8` — introduces mild saturation at 80% of full scale, hard limits at unity.

**Minimum cycle length enforcement.** At very high Freq Scatter values, breakpoints can collapse to zero-length cycle segments, producing aliased artifacts that are not musically useful and not recoverable by the user. The DSP must enforce a minimum time separation between adjacent breakpoints of `4 / sample_rate` seconds (4 samples). This is a silent guard — it never triggers under normal use and is only activated at extreme parameter combinations.

**Pitch snap interpolation.** When Pitch Snap transitions from 0.0 to 1.0, the quantization effect must not appear as a stepped jump. Implement as a **weighted interpolation between the raw GENDYN frequency and the quantized frequency**, where the interpolation coefficient is the Pitch Snap parameter value. At Pitch Snap = 0.5, each cycle's frequency is 50% pulled toward the nearest scale degree. This creates a gradient from free chaos to full xenharmonic quantization that passes through musically interesting intermediate states.

---

### 4.2 DIFFU — Musical Success Definition

DIFFU is successful when it provides **morphologically rich pattern behavior** that can be used as musical motion, timbre evolution, or structural transformation.

#### Musical Success (DIFFU)

A DIFFU patch is musically successful when:

- Region changes (Pearson phase regime transitions) create **recognizable sonic families**
- Morphing across regions produces **meaningful transitions** (not only collapse or incoherence)
- The user can navigate pattern complexity while preserving intent

#### What "Good DIFFU Behavior" Feels Like

- Different zones sound behaviorally distinct (named zones are meaningful to the ear)
- Parameter movement creates phrase-like or form-like change over musical timescales (4–32 bars)
- There is a controllable range from: stable texture → evolving pattern → transitional mutation → chaotic diffusion
- The engine supports both "hold a state" and "animate a state"

#### DIFFU Musical KPIs

- Named regions/archetypes are **audibly distinct** in blind A/B testing (3 out of 5 untrained listeners correctly identify region change)
- Transition paths between selected regions are musically useful in at least one of: drones, evolving pads/textures, percussive patterning, abstract gestures
- Fast F/k modulation does not always collapse into harsh noise wash

#### DIFFU Musical Archetype Targets

| Region Name | F | k | Temporal Behavior | Spectral Character | Musical Use |
|---|---|---|---|---|---|
| Frozen Field | 0.020 | 0.055 | Static — minimal evolution after convergence | Near-uniform, quiet | Silence/ambient space, tail of a sound |
| Soliton Drift | 0.062 | 0.061 | Slow-moving spectral peaks traverse the grid | Isolated bright harmonics drifting | Slow evolving pad, meditation drone |
| Mitosis Burst | 0.028 | 0.062 | Rapid repeated spectral doubling events | Dense harmonic mid-range activity | Generative rhythm bed, IDM texture |
| Coral Architecture | 0.054 | 0.063 | Stable branching patterns with slow drift | Complex but consistent spectral content | Long-form evolving drone, ambient bed |
| Labyrinthine | 0.037 | 0.060 | Maze-like patterns, slow large-scale drift | Densely packed harmonics | Industrial texture, noise floor filler |
| Chaos Wash | 0.026 | 0.051 | Constant rapid state changes | Broadband with emergent tonal moments | Transition material, tension builder |

#### DIFFU-Specific DSP Guardrails (Musical)

**Grid state rate limiting.** The Gray-Scott system can evolve faster than musical time — at high feed rates, a pattern that took 8 bars to establish can collapse in 2 beats when F is increased. This destroys musical usability. The DSP must support a **temporal rate multiplier** that scales the time-step `dt` of the simulation independently of audio block size. Default dt should be calibrated so that a full state transition from Solitons to Mitosis takes approximately **4–16 seconds at default dt**. Expose a "Evolution Rate" parameter (0.1× to 10×) that adjusts dt. This gives the user musical control over time without changing the mathematical character of the system.

**Spectral centroid smoothing.** When the DIFFU grid transitions between phases (especially entering Chaos), the centroid of the additive bank can jump rapidly. This creates harsh transient brightness changes that do not read as intentional musical motion — they read as a technical artifact. Apply a **20ms exponential moving average to the amplitude vector** feeding the additive bank. The individual partial amplitudes track the grid state, but the perceptual result is smoothed so that sudden grid state changes arrive as fast sweeps rather than instant jumps.

**Phase coherence management.** When a partial's amplitude drops to zero as the grid evolves and then rises again, the phase accumulator for that partial has continued advancing. This produces clicks and pops as the partial re-enters the signal at an arbitrary phase. Implement **amplitude-gated phase freezing**: when a partial amplitude falls below a threshold (`< 0.001`), freeze its phase accumulator. When amplitude rises again, the partial re-enters at its frozen phase, producing a coherent rather than random reentry.

**Minimum partial density floor.** In Frozen Field states, the grid converges toward uniformity and the additive bank can approach silence. This is valid as a musical state but must not make the engine appear broken. Implement a **subsonic hum injection** — a sine wave at 12–18 Hz, amplitude proportional to the "silence" of the grid (inverse of total V concentration sum), mixed at approximately -40 dBFS. The user never consciously hears this; it prevents monitoring equipment from going silent and the user from assuming the plugin has crashed.

---

### 4.3 BUBBLE — Musical Success Definition

BUBBLE is successful when physical/fluid-inspired behavior becomes a **performable sonic engine**, not just a novelty simulator.

#### Musical Success (BUBBLE)

A BUBBLE patch is musically successful when it provides:

- Expressive pressure/coupling/density-driven behavior that responds to performance gestures
- Controllable articulation spanning: percussive → pulsed → swarming → floating
- Distinct archetypal "materials" (fluid presets) that stay recognizable under modulation
- A coherent relationship between physical parameters and perceived sonic weight/texture

#### What "Good BUBBLE Behavior" Feels Like

- Bubble size/density/resonance controls produce interpretable changes without requiring knowledge of Minnaert physics
- Preset archetypes (water/oil/mercury/magma/plasma) feel genuinely different, not cosmetic renaming of the same sound
- Coupling can create richness (the cloud effect) without always becoming muddy or chaotic
- Bubble population motion can be shaped into rhythmic or phrase-level structures through spawn rate and MPE control

#### BUBBLE Musical KPIs

- Archetype presets remain **identity-consistent** across moderate modulation (±20% parameter shift from archetype center)
- User can intentionally reach at least 4 distinct categories:
  - Sparse plucks/pops (Water, low spawn rate)
  - Dense shimmer/swarm (Plasma, high spawn rate)
  - Resonant pulses (Mercury, moderate coupling)
  - Atmospheric bubbling beds (Magma, maximum coupling, slow evolution)
- Engine remains controllable with expressive MPE input without immediate tonal collapse

#### BUBBLE Musical Archetype Targets

| Archetype | Density (kg/m³) | Surface Tension | Mean Radius | Spawn Rate | Coupling | Sonic Identity |
|---|---|---|---|---|---|---|
| Water | 1000 | 0.073 | 0.5–2mm | 20–80 Hz | 0.0–0.2 | Granular clicks, high-frequency sparkle, percussive |
| Oil | 870 | 0.032 | 2–8mm | 5–20 Hz | 0.1–0.3 | Warm rounded plucks, mid-range body, slower |
| Mercury | 13600 | 0.485 | 3–15mm | 2–10 Hz | 0.3–0.6 | Low fundamental, heavy resonant pulses, metallic shimmer |
| Magma | 2700 | 0.400 | 10–40mm | 0.5–3 Hz | 0.5–0.9 | Deep sub-bass drones, massive slow emergence, felt as much as heard |
| Plasma | 500 | 0.010 | 0.1–1mm | 100–500 Hz | 0.0–0.1 | Dense high-frequency energy, aggressive, near-noise |

#### BUBBLE-Specific DSP Guardrails (Musical)

**Population voice budget.** An unconstrained bubble population can grow large enough to consume all CPU or produce a sound with no defined character — just mass. The DSP must enforce a hard maximum population of **128 simultaneously active bubble oscillators per polyphonic voice**. When the budget is full, new spawns displace the oldest active bubbles (FIFO eviction). The evicted bubbles perform a fast fade-out over 5ms to avoid clicks. This ceiling is musically correct as well as computationally safe — populations above approximately 64 bubbles produce indistinct swarm textures regardless.

**Frequency ceiling enforcement.** Minnaert frequency for very small bubbles (radius < 0.1mm) in low-density fluids can exceed 20kHz. These partials are inaudible but consume DSP budget. The DSP must silently discard any bubble whose computed Minnaert frequency exceeds `sample_rate * 0.45` (Nyquist safety margin). This is transparent to the user.

**Sub-bass emergence protection.** The cloud coupling model produces emergent low frequencies. In Magma preset with maximum coupling and a large population, this emergent bass can accumulate to excessive amplitude at very low frequencies (below 20 Hz). Apply a **6 dB/octave high-pass filter at 18 Hz** on the BUBBLE engine output. Sub-bass below 18 Hz is inaudible and represents wasted headroom. This filter has no perceptible effect on the musical output.

**Resonance decay minimum.** At very low surface tension values (Plasma archetype), bubble decay rates approach zero — bubbles ring indefinitely. This produces a steady-state accumulation of sinusoidal energy that can build to an extremely loud tone over 10–20 seconds of play. Enforce a minimum decay coefficient equivalent to a maximum ring time of **8 seconds** per bubble, regardless of surface tension parameter value. This is musically appropriate — even in highly resonant physical systems, energy eventually dissipates.

**Spawn regularity vs. musicality.** At moderate spawn rates (10–50 Hz), a perfectly regular Poisson process for bubble spawning produces a rhythm that the human ear locks onto as a mechanical pulse. This is often musically undesirable — it makes BUBBLE sound like a stuttering artifact rather than a natural fluid process. Implement **spawn time jitter**: each scheduled spawn event is displaced by ±15% of its nominal inter-spawn interval using a uniform distribution. This preserves the sense of rate while preventing mechanical regularity. At very high spawn rates (> 100 Hz), the jitter has no perceptual effect and can be reduced to ±5%.

---

## 5. Usable Patch Workflow Definition

A "usable patch workflow" means the user can move from intention to result with a coherent sequence, using anchors and feedback.

### 5.1 Minimum Workflow Stages (Must Be Supported)

#### Stage A — Choose an Anchor

User starts from one of:

- Init patch (engine-specific, see Section 13 for init patch specifications)
- Named preset archetype
- Region snap / zone preset (DIFFU Pearson regions)
- Fluid archetype (BUBBLE material presets)
- STOCH quantized mode preset

**Requirement:** Starting points must be sonically meaningful, not empty or hostile.

#### Stage B — Shape the Core Behavior

User adjusts a small number of high-impact controls/macros to define:

- Stability vs. complexity
- Motion speed/rate
- Density/intensity
- Tonal anchoring (where applicable)

**Requirement:** These controls must be immediately visible, labeled in perceptual terms (not mathematical), and produce audible results within 3dB of the anchor state at default values.

#### Stage C — Refine Character

User tunes secondary controls:

- Jitter/detail/spread/coupling/resonance/etc.
- Modulation depth/routing amounts
- Dynamic response / articulation shaping

**Requirement:** Refinement changes should improve or redirect the sound, not frequently destroy it. No secondary control should be able to cause a state transition to a musically incoherent result unless the user travels past a visually marked "danger zone."

#### Stage D — Performance / Composition Fit

User aligns patch to musical context through:

- Gain/level balance
- Envelope timing / rate sync / modulation behavior
- MPE response calibration
- Blend balance between engines

**Requirement:** This stage should feel like fitting and expressing, not debugging.

#### Stage E — Recall / Save / Reuse

User can save and return to the sound family:

- Preset save with parameter snapshot
- Named region/archetype reference visible in UI at all times
- Visible parameter state (no hidden or implicit values)

**Requirement:** Preset/snapshot state recall must be reliable and predictable. Every parameter that affects the sound must be included in the preset state. There must be no "invisible state" that changes behavior but is not recalled with the preset.

#### Stage F — Iterative Refinement Loop

This stage is new in v0.2. It captures what actually happens in production: a user builds a patch, uses it in a project, returns to it, adjusts it for the new context, and resaves it.

The ENTROP preset system must support **non-destructive preset editing**: saving an edited version of a preset must not overwrite the source preset without explicit user confirmation. The user should always be able to return to the original preset family while retaining their edited version.

Additionally, the patch system must support **archetype memory**: when a user edits a patch that was derived from a named archetype (Soliton Drift, Mercury, Xenharmonic Lead), the UI should still show the originating archetype name with an "edited" indicator. This preserves the mental model of where the sound came from.

---

### 5.2 Usable Patch Workflow Anti-Patterns (Must Be Avoided)

| Anti-Pattern | Root Cause | Required Mitigation |
|---|---|---|
| "Random until lucky" | No anchor, no predictable navigation structure | Named archetypes, init patches, visual region map |
| Main controls with inaudible ranges | Linear taper on perceptually exponential parameter | Logarithmic/exponential taper on all frequency and gain-related controls |
| Constant gain explosions during exploration | No output normalization or safety ceiling | Soft-limiter at output stage (see Section 3.3), perceptual normalization in additive bank |
| Hidden knowledge required | Parameters named in technical/mathematical terms | All primary controls labeled in perceptual terms; mathematical names in tooltip only |
| Presets that collapse when touched | Preset occupies a narrow island of parameter space | Presets must be tested for robustness (±10% parameter variation test, see Section 7.2 Scenario B) |
| Labels that don't match perceived effect | Control name chosen during development, not testing | Every primary control label must be validated by a user who has never seen the developer plan |
| Exploration fatigue | No recovery path from deep states | Single gesture "return to anchor" available at all times — keyboard shortcut + visible UI button |

---

## 6. Failure Modes (Musical + UX)

The following are **musical failure modes**, not only technical bugs. They must be tracked in QA as first-class issues.

### 6.1 Stability Failure Modes

- **FM-01: Over-instability at default settings**  
  Init/archetype starts too chaotic to be playable/useful. *Root cause: parameter defaults set by developer testing, not user testing.*

- **FM-02: Collapse on small movement**  
  Tiny parameter moves radically destroy sound identity. *Root cause: parameter is near a bifurcation point in the underlying system.*

- **FM-03: Dead zones**  
  Large portions of control travel produce minimal audible change. *Root cause: linear taper on a perceptually logarithmic parameter, or parameter range extending beyond the useful physical domain.*

- **FM-04: Binary behavior**  
  Control effectively acts as on/off instead of gradient. *Root cause: parameter controls a threshold crossing in the mathematical system. Mitigation: hysteresis or soft transition implementation.*

### 6.2 Spectral / Tonal Failure Modes

- **FM-05: Harshness spikes**  
  Sudden narrowband or high-frequency energy becomes fatiguing/unusable. *Root cause: additive bank partial entering alignment at full amplitude; or Cauchy distribution extreme-value event.*

- **FM-06: Tonal incoherence without recovery path**  
  Sound becomes tonally diffuse/random and there is no quick way back via snap/anchor. *Root cause: user has navigated outside all defined archetypes with no visible indication.*

- **FM-07: Mud accumulation**  
  Coupling/density/blend states produce indistinct low-frequency mush too easily. *Root cause: bubble coupling + DIFFU low-frequency partials reinforcing each other with no inter-engine spectral management.*

- **FM-08: Identity washout in blend**  
  Combining engines removes useful character instead of creating a richer composite. *Root cause: blend is additive summation without spectral complement awareness.*

### 6.3 Dynamics / Gain Failure Modes

- **FM-09: Loudness jumps during exploration**  
  Parameter changes cause excessive volume shifts. *Threshold: > ±6 dB perceived loudness on a single primary control movement.*

- **FM-10: Accidental clipping in common workflows**  
  Normal use pushes output into clipping too easily. *Mitigation: soft limiter at output stage.*

- **FM-11: Perceived loudness collapse**  
  Interesting states become too quiet/thin to evaluate musically. *Root cause: high partial density at low individual amplitudes — many quiet partials that do not sum perceptibly.*

### 6.4 Time / Motion Failure Modes

- **FM-12: Motion without phrase value**  
  Sound moves, but the motion feels arbitrary/unmusical. *Root cause: LFO or R-D evolution rate is mismatched to musical tempo. Consider sync-to-host-tempo option.*

- **FM-13: Temporal smearing**  
  Fast changes blur into uninterpretable wash without user intent. *Root cause: modulation rate too high for the engine's natural decay time.*

- **FM-14: Static stagnation**  
  Engine sounds rich initially but becomes uninteresting over musical timescales (> 16 bars). *Root cause: R-D system has converged to a fixed point; GENDYN distribution variance too low; bubble population has settled to steady state.*

### 6.5 Interaction / UX Failure Modes

- **FM-15: Label-to-sound mismatch**  
  Control naming suggests one effect, but user hears another. *Must be caught in user testing — developer cannot self-assess this.*

- **FM-16: Hidden dependency traps**  
  A control appears broken because another hidden state suppresses it. *Example: Pitch Snap appears inactive because scale root is set to a pitch outside the current octave.*

- **FM-17: Unclear hierarchy**  
  User cannot tell which controls are primary vs. refinement vs. danger.

- **FM-18: Preset fragility**  
  Small edits destroy preset usefulness immediately. *Requirement: every preset must pass the robustness test in Scenario B.*

### 6.6 Polyphony Failure Modes (New in v0.2)

- **FM-19: Voice identity collision**  
  Multiple voices using STOCH produce a combined output where individual voice character is lost, resulting in generic noise. *Root cause: STOCH voices share a PRNG seed or have identical initial breakpoint states. Each voice must be seeded differently, with seed derived from voice index + note-on timestamp.*

- **FM-20: DIFFU polyphony phase cancellation**  
  Two DIFFU voices at the same pitch produce cancellation artifacts because their additive banks are at opposite phases for some partials. *Root cause: R-D grid state has diverged between voices (good — each voice has independent musical identity) but phase accumulators are not compensated for this. No mitigation required — this is an emergent musical property. Document it as a feature: two voices at the same pitch produce beating/chorus effects rather than reinforcement.*

- **FM-21: BUBBLE voice pile-up**  
  High polyphony with BUBBLE at high spawn rates exhausts the bubble population budget before lower voices have established themselves. *Root cause: all voices compete for the same population ceiling. Mitigation: the 128-bubble ceiling applies per voice, not globally.*

### 6.7 Modulation Failure Modes (New in v0.2)

- **FM-22: MPE expression causing instant tonal collapse**  
  Pressure or slide expression immediately moves a parameter into a musically incoherent state. *Root cause: MPE modulation depth is too large relative to the parameter's sweet-spot width. Mitigation: default MPE modulation depth must be calibrated so that full-scale expression (0→127) maps to the musically useful travel of the target parameter, not its full mathematical range.*

- **FM-23: LFO rate mismatch with engine time constant**  
  CA LFO rate is faster than the engine's internal evolution time, producing modulation that the engine cannot follow. *Root cause: LFO modulates a parameter that affects a slow physical process (R-D evolution, bubble population). The parameter changes faster than the engine's response. This is musically useless — the engine averages out the modulation and the user hears no effect. Mitigation: LFO rate should be soft-limited based on the target parameter's estimated time constant.*

- **FM-24: Modulation depth destroying preset identity**  
  When modulation is active, the patch moves so far from its anchor state that it no longer resembles the original preset. *Mitigation: modulation routing UI must show the post-modulation range of each parameter visually (a dotted arc overlay on the knob showing the modulated range).*

### 6.8 Blend-Specific Failure Modes (New in v0.2)

- **FM-25: Spectral overcrowding at full blend**  
  When all three engines are active at equal weight, the combined spectral density becomes an undifferentiated mass. *Root cause: the three engines' natural frequency distributions overlap heavily. Mitigation: implement inter-engine spectral complement behavior — see Section 10.3.*

- **FM-26: Blend transition discontinuity**  
  Moving the tri-morph control produces audible clicks or sudden timbre jumps at certain blend positions. *Root cause: one engine's output jumps from silent to audible without a fade-in. All engine contribution functions in the blend must be continuous and overlap smoothly.*

### 6.9 Preset System Failure Modes (New in v0.2)

- **FM-27: Preset recall instability**  
  Loading a preset does not produce the same sound on every load. *Root cause for STOCH: PRNG state is not reset on preset load. Requirement: STOCH voice PRNG must reset to a deterministic seed on preset load. The seed can be stored in the preset and set by the user, or default to a fixed value per preset.*

- **FM-28: Cross-version preset incompatibility**  
  A preset saved in v0.9 does not load correctly in v1.0. *Requirement: the preset file format must be versioned. Every parameter must have a documented default that is used when loading older presets that do not include that parameter.*

---

## 7. In-Context Testing (Musical Reality Testing)

Algorithmic correctness is necessary, but not sufficient. ENTROP must be tested in musical contexts.

### 7.1 Required Contexts

Every significant milestone must be tested in at least these contexts:

1. **Single-note exploratory play**
   - Sustained notes
   - Short taps at varied velocities
   - MPE gestures (if applicable)

2. **Phrase play**
   - Short melodic/atonal phrase
   - Repeated motif at different tempos
   - Rhythmic triggering patterns

3. **Composition fit**
   - Patch placed in a minimal arrangement (drums / harmonic bed / other texture)
   - Level-matched
   - Basic mix treatment applied (EQ / comp / reverb)

4. **Automation performance**
   - 1–3 macro movements over time
   - Evaluate whether motion reads as intentional musical evolution vs. random change

### 7.2 Test Scenarios (Minimum Set)

#### Scenario A — Fast Discovery Test

Goal: measure time-to-usable-sound from init/archetype.

- Start from engine init or named archetype
- Allow 2 minutes (new user profile) or 1 minute (experienced profile)
- Tester must produce one sound they would keep in a project

**Record:** success/fail, time taken, controls used, failure reason if any

#### Scenario B — Patch Edit Robustness Test

Goal: ensure patches survive normal editing.

- Load a "good" preset archetype
- Perform 10 common edits: shift each primary control ±20% from its preset value
- After each edit, evaluate: musically useful still? Recoverable within 2 gestures?

**Pass condition:** at least 7/10 edits preserve usefulness or allow quick recovery

#### Scenario C — Region/Archetype Navigation Test

Goal: test whether anchors are musically meaningful.

- For DIFFU: navigate between all 6 Pearson region presets using the XY pad
- For BUBBLE: cycle through all 5 fluid archetype presets
- For STOCH: cycle through all 5 musical role archetypes

**Pass condition:** each transition produces a clearly different sound character; the user can identify which archetype they are in without looking at the label

#### Scenario D — Arrangement Placement Test

Goal: test real usefulness beyond solo mode.

- Put patch into a simple project context (kick + bass + pad, minimal mix)
- Level-match output
- Evaluate: audibility, controllability, masking/mud/harshness, whether movement helps or distracts

**Note:** ENTROP can be abstract/avant-garde, but should still create intentionally placeable results.

#### Scenario E — Performance Expression Test (MPE/Automation)

Goal: test expressive controllability.

- Perform pressure/slides/automation curves across musical phrases
- Evaluate: response smoothness, musical usefulness of modulation, identity preservation under expression

#### Scenario F — Long-Form Static Test (New in v0.2)

Goal: test whether ENTROP sounds can sustain interest over musical timescales without additional modulation.

- Load a patch designed for ambient/drone use
- Hold a sustained note for 60 seconds with no parameter changes
- Evaluate: does the sound evolve naturally and interestingly? Does it stagnate? Does it collapse?

**Pass condition:** the DIFFU and BUBBLE engines produce audibly evolving output for the full 60 seconds without user intervention. STOCH at low Freq Scatter should also produce subtle micro-variation.

#### Scenario G — Polyphonic Character Test (New in v0.2)

Goal: test whether polyphony adds musical richness or creates chaos.

- Play a 4-note chord on a STOCH patch
- Hold all 4 notes for 8 seconds
- Evaluate: do the voices maintain individual character? Is the combined result coherent?

**Pass condition:** the combined output is denser and richer than a single voice, but individual voice motion is still perceptible in the texture. The chord does not collapse into undifferentiated noise within 8 seconds.

### 7.3 In-Context Test Logging Template

For each test run, record:

| Field | Value |
|---|---|
| Engine / Blend Mode | |
| Starting Point | (init / preset / region / archetype) |
| User Profile | (new / intermediate / experienced / developer) |
| Task | (discover / edit / perform / place in mix / long-form) |
| Time to Usable Result | (seconds, or "failed") |
| Result | (pass / partial / fail) |
| Musical Notes | (what worked, what character emerged) |
| Failure Mode IDs | (FM-xx list) |
| Root Cause Assessment | (DSP / UI / preset / mapping / docs) |
| Suggested Fix | (specific action) |

---

## 8. UX Guardrails to Improve Musical Usefulness

These are implementation and UX strategies that preserve exploration while increasing usability.

### 8.1 Anchors and Recovery

- Provide **meaningful init patches** per engine — not silence, not full chaos
- Provide **named archetypes / region snaps** accessible via single click
- Add **double-click to reset** for every control
- Add **"Return to Anchor" button** visible at all times — returns all controls to the last-loaded preset/archetype state without requiring a full preset reload
- Add **anchor memory**: the UI remembers the last archetype the user navigated from, even after parameter changes, and shows it as a reference

### 8.2 Perceptual Control Mapping

- Prefer **perceptual/nonlinear tapers** over raw linear travel for all controls that the user hears logarithmically or exponentially
- Concentrate useful changes in audible ranges — extend the parameter range of linear tapers downward below useful values only if it creates useful access to extreme states, not to pad the range
- Smooth dangerous transitions: where a physical parameter has a bifurcation point (a value at which the mathematical behavior changes qualitatively), the DSP should implement a transition zone of ±5% around that point where the behavior interpolates rather than switches

### 8.3 Clear Control Hierarchy

UI should visually distinguish four levels of control (see also Design Brief Section 6.3):

1. **Core shaping controls** — 3 per engine, most prominent
2. **Refinement controls** — 4–6 per engine, secondary tier
3. **Advanced/Danger controls** — clearly marked, requires deliberate interaction to access
4. **Performance controls** — MPE routing, blend morph, macro assignments

### 8.4 Controlled Extremes

ENTROP should allow extreme states, but:

- They must be **intentionally reachable**, not the first thing a new user encounters
- They should be **signposted** with a visual indicator at the dangerous end of the control range
- There should always be a **recovery path** that does not require knowing where you came from
- They must **not dominate default workflows**

Implementation: mark the extreme 15% of range on "dangerous" controls with a subtle color change in the UI arc (from engine accent color to a desaturated warning tone). This is a visual affordance, not a technical guard.

### 8.5 Preset Strategy as Musical Infrastructure

Presets are not just demos. They are primary usability infrastructure:

| Preset Category | Purpose | Quantity Target |
|---|---|---|
| Init presets | Starting points for each engine, neutral character | 1 per engine (3 total) |
| Archetype presets | Named musical role within each engine | 5 per engine (15 total) |
| Composition-ready presets | Tested in mix context, level-matched, immediately usable | 10 total |
| Exploration presets | Push edges of each engine, good for discovery | 8 total |
| Blend presets | Use the tri-morph, demonstrate engine combination | 5 total |
| Edge-case / showcase | Extreme states, technically interesting, clearly labeled as such | 4 total |

**Total minimum preset set at v1.0 release: 45 presets**

All presets must pass Scenario B (Patch Edit Robustness Test) before inclusion in the shipped set.

### 8.6 Perceptual Loudness Normalization (New in v0.2)

The three engines have very different natural loudness at their default operating ranges. STOCH at mid-settings produces a sparse waveform that peaks at ±0.6 amplitude. DIFFU at maximum spectral density with 128 partials can reach ±8.0 before gain scaling. BUBBLE with 64 active oscillators at moderate amplitude sums to ±4.0.

A user blending these three engines without compensation would experience constant loudness shifts. The DSP must implement **per-engine perceptual loudness normalization** before the blend stage:

- Each engine's output is measured via a fast (10ms integration window) RMS level estimator
- The gain applied to each engine's output is adjusted to maintain a target RMS of −18 dBFS
- This gain is **smoothed with a 100ms release time** to prevent audible gain pumping
- The normalization is transparent and cannot be disabled by the user — it is infrastructure, not a feature
- The normalization does not reduce the headroom available for the output soft-limiter; it operates independently

---

## 9. Macro Control Architecture

Complex synthesis engines become musically accessible when their parameter space is reduced to a small set of high-level, perceptually meaningful controls. This section defines the macro control architecture for ENTROP — how to expose the full parameter depth through a simplified surface that serves musical workflow without hiding the underlying system.

### 9.1 Macro Philosophy

Each engine exposes two tiers of control:

**Tier 1 — The Three Core Macros.** Three knobs that cover the most musically important degrees of freedom. These must be reachable without sub-panels or menus. Every archetype preset is defined primarily by the positions of these three knobs.

**Tier 2 — The Full Parameter Set.** All parameters from the developer plan, accessible but visually subordinate. For users who understand the physics and want direct control.

The macro system does not replace the full parameter set. It is a high-level view of it. Macro movements translate to coordinated movements of multiple underlying parameters — a single "Complexity" macro might simultaneously adjust Freq Scatter, Amp Scatter, and Barrier Elasticity in proportions calibrated by the preset designer.

### 9.2 STOCH Core Macros

| Macro Name | What It Controls Underneath | Musical Effect |
|---|---|---|
| **Stability** | Inverted blend of Freq Scatter + Amp Scatter; modulates toward low-scatter (stable) or high-scatter (chaotic) end together | From "almost stable" to "barely controlled chaos" |
| **Tonality** | Pitch Snap amount + Scale Root octave offset | From "free pitch chaos" to "xenharmonically quantized" |
| **Density** | Number of Segments + Barrier Elasticity inverse | From "sparse angular waves" to "dense complex contour" |

### 9.3 DIFFU Core Macros

| Macro Name | What It Controls Underneath | Musical Effect |
|---|---|---|
| **Phase** | F and k together, constrained to move along a musically useful diagonal in Pearson space (from Frozen Field → Solitons → Coral → Mitosis) | From "static" to "actively generative" |
| **Tempo** | Evolution Rate (dt multiplier) | From "glacially slow" to "faster than bar length" |
| **Complexity** | Spectral Map Curve + Du/Dv ratio | From "few dominant harmonics" to "dense harmonic cloud" |

### 9.4 BUBBLE Core Macros

| Macro Name | What It Controls Underneath | Musical Effect |
|---|---|---|
| **Material** | Fluid Density + Surface Tension coordinated (moves between Water → Oil → Mercury → Magma) | From "light and percussive" to "massive and sustained" |
| **Population** | Spawn Rate + Mean Radius coordinated | From "sparse individual bubbles" to "dense swarm" |
| **Depth** | Coupling Coefficient + Pressure | From "individual oscillators" to "cloud with emergent sub-bass" |

---

## 10. DSP-Level Musical Constraints

These are requirements for the audio engine that must be guaranteed regardless of parameter state. They are not soft guidelines — they are invariants.

### 10.1 Output Level Invariants

1. **The plugin output must never exceed 0 dBFS peak** (enforced by output soft-limiter, see Section 3.3)
2. **The plugin output must never fall below −60 dBFS RMS during sustained note play with any non-init preset** (enforced by subsonic hum injection in DIFFU Frozen Field state, and by minimum partial floor in additive bank)
3. **Engine blend transitions must never produce an output level jump greater than 6 dB instantaneously** (enforced by per-engine loudness normalization, see Section 8.6)

### 10.2 Temporal Invariants

1. **No DSP processing may cause a sustained audio dropout (> 1ms silence) during parameter changes** — all parameter changes must be interpolated over a minimum of 64 samples
2. **Preset recall must not cause audio clicks** — implement a 10ms crossfade from current state to recalled state on preset load while audio is active
3. **STOCH cycle length must never be zero** — minimum 4-sample cycle enforced by DSP

### 10.3 Inter-Engine Spectral Complement (Blend Musical Quality)

When multiple engines are active simultaneously, they should complement each other spectrally rather than compete. This is a design guideline, not an automatic DSP constraint — it must be enforced through preset design and archetype calibration:

- **STOCH + DIFFU blend** works best when STOCH is calibrated to mid/high-frequency content (high Segments, high Freq Scatter) and DIFFU is in a low-to-mid spectral region (low Spectral Map Curve)
- **DIFFU + BUBBLE blend** works best when BUBBLE is generating low-frequency content (Magma or Mercury archetype) while DIFFU covers mid-range
- **STOCH + BUBBLE blend** works best when STOCH provides pitch structure (moderate Pitch Snap) and BUBBLE provides percussive texture (Water or Oil archetype)

The blend presets in the preset library must demonstrate these complementary configurations as their starting points.

---

## 11. Per-Engine Parameter Taper and Range Specification

This section defines the required mapping curve (taper) for each primary control. "Linear" means the parameter value presented to the DSP increases proportionally to the control position. "Logarithmic" means the parameter value is proportional to the exponential of the control position — correct for any parameter the user hears in units of frequency, pitch, or perceived intensity.

### 11.1 STOCH Parameter Tapers

| Parameter | DSP Range | Control Taper | Reason |
|---|---|---|---|
| Segments | 3–12 (integer) | Linear with integer snap | Segment count is not perceptual — linear is correct |
| Amp Scatter (σ/γ) | 0.0–1.0 | Exponential (x²) | Doubling scatter is perceived as equal steps |
| Freq Scatter (σ/γ) | 0.0–1.0 | Exponential (x²) | Same reason |
| Barrier Elasticity | 0.0–1.0 | Linear | Direct reflection coefficient |
| Pitch Snap | 0.0–1.0 | Linear | Interpolation coefficient — linear is correct |
| Scale Root | MIDI 0–127 | Quantized to 12-note grid | Absolute pitch selection |

### 11.2 DIFFU Parameter Tapers

| Parameter | DSP Range | Control Taper | Reason |
|---|---|---|---|
| Feed Rate F | 0.010–0.080 | Linear | Small range; perceptual delta roughly linear in this range |
| Kill Rate k | 0.040–0.070 | Linear | Same |
| Evolution Rate (dt) | 0.1×–10× | Logarithmic | Perceived time feels logarithmic |
| Spectral Map Curve | 0.0–1.0 | Sigmoid (soft) | Avoid harsh transition at mid-point |
| Diffusion Ratio Du/Dv | 1.0–8.0 | Logarithmic | Ratio parameter — log is correct |

### 11.3 BUBBLE Parameter Tapers

| Parameter | DSP Range | Control Taper | Reason |
|---|---|---|---|
| Fluid Density | 500–14000 kg/m³ | Logarithmic | Perceived as weight — logarithmic |
| Surface Tension | 0.010–0.500 N/m | Logarithmic | Same |
| Pressure | 50000–200000 Pa | Linear | Direct pressure scaling |
| Mean Radius | 0.1mm–40mm | Logarithmic | Frequency is inverse of radius — log maps to linear pitch |
| Radius Spread (σ) | 0.1–2.0 | Linear | Direct distribution parameter |
| Spawn Rate | 0.5–500 Hz | Logarithmic | Frequency parameter — always log |
| Coupling Coefficient | 0.0–1.0 | Exponential (x²) | Most useful range is in lower values |

---

## 12. Polyphony Musical Behavior Contract

Each polyphonic voice in ENTROP is an independent instance of the active engine(s). This independence is musically valuable — it means each note has its own evolving state. But it also creates risks for musical coherence.

### 12.1 Voice Independence Rules

1. **STOCH voices must be seeded differently.** Each voice is assigned a unique seed at plugin initialization. The seed for voice N is `(base_seed * (N + 1)) XOR note_on_timestamp_mod_2^16`. This ensures that even two voices playing the same note simultaneously will evolve differently.

2. **DIFFU voices have independent grid states.** Each voice has its own 128-cell grid. Grid states diverge from the same initial condition immediately after voice birth, creating natural variation between voices.

3. **BUBBLE voices have independent bubble populations.** Voice 1's bubbles never interact with Voice 2's bubbles. Each voice has its own population ceiling of 128 bubbles.

### 12.2 Voice Coherence Rules

Despite being independent, voices must maintain enough coherence to function together:

1. **All voices share the same F/k position in DIFFU.** The Pearson space navigation affects all active voices simultaneously — the region is a global instrument state, not a per-voice state. Individual grid evolutions diverge, but they start from the same initial condition when the region changes.

2. **All voices share the same fluid archetype in BUBBLE.** Material (density, tension) is a global instrument state. Per-voice modulation via MPE can adjust spawn rate, mean radius, and coupling independently per note.

3. **STOCH pitch quantization uses the same scale across all voices.** The xenharmonic scale is global. Per-voice modulation can shift the offset within the scale, but not the scale itself.

### 12.3 Voice Stealing Musical Behavior

When the polyphony limit is reached and a new note must steal a voice:

- **Do not steal the voice with the longest sustain.** This would kill the most evolved and interesting voice — the one the user has been hearing for longest and has become part of the texture.
- **Steal the voice with the lowest envelope amplitude** (closest to silence or in its decay phase). This is the standard ADSR voice-stealing rule and it applies here.
- **Fade stolen voice over 10ms** before reassigning, even if the voice appears to be in a loud state — sudden silence is more disruptive than a short fade artifact.

---

## 13. Preset Architecture Specification

The preset system is the primary usability infrastructure of ENTROP. It must be designed with the same rigor as the synthesis engines.

### 13.1 Init Patch Specifications

Each engine must have a dedicated init patch that serves as a neutral, musically legible starting point:

**STOCH Init**  
- Segments: 6, Distribution: Gaussian, Amp Scatter: 0.25, Freq Scatter: 0.20, Barrier Elasticity: 0.60, Pitch Snap: 0.0  
- Expected sound: A slightly wavering, clearly pitched tone with gentle waveform irregularity. Not a sine wave. Not noise. Something that immediately communicates "this is alive and stochastic."  
- TTUS from this init: < 30 seconds for an experienced user

**DIFFU Init**  
- F: 0.054 (Coral Architecture region), k: 0.063, Evolution Rate: 1.0×, Spectral Map: 0.5 (logarithmic)  
- Expected sound: A slowly evolving harmonic texture with a clear sense of mid-range spectral activity. Audibly changing. Not static.  
- TTUS from this init: < 45 seconds for an experienced user

**BUBBLE Init**  
- Fluid: Oil preset (Density: 870, Surface Tension: 0.032), Mean Radius: 4mm, Spawn Rate: 12 Hz, Coupling: 0.15  
- Expected sound: Warm, irregular plucked resonances at a moderate rate. Clearly physical in character. Recognizable as something from the fluid domain without being literally "water sounds."  
- TTUS from this init: < 30 seconds for an experienced user

### 13.2 Archetype Preset Design Rules

1. Every archetype preset must be defined by its position in a two-dimensional description: *physical/mathematical state* × *musical role*. Both must be specified.
2. Every archetype preset must be tested with the Robustness Test (Scenario B) and pass with ≥ 7/10.
3. Archetype presets must demonstrate broad parameter spacing — two archetypes within the same engine should differ by at least 30% on at least two primary parameters.
4. Archetype preset names must describe the sonic result, not the mathematical state. "Soliton Drift" describes the R-D pattern but also communicates "slow, isolated movement" to a user who has never heard of solitons.

### 13.3 Preset File Format Requirements

```
ENTROP Preset Format v1.0
{
  "format_version": 1,
  "preset_name": "Soliton Drift",
  "engine_config": "DIFFU",
  "archetype_origin": "Coral Architecture",    // which init or archetype this was derived from
  "musical_role": "evolving pad",
  "test_status": "robustness_passed",          // filled by QA
  "parameters": {
    // full parameter state — all parameters, no omissions
    "diffu_F": 0.054,
    "diffu_k": 0.063,
    // ...
  },
  "notes": "Designed for slow-moving ambient contexts. Works well with long reverb. Sensitive to Spectral Map curve — keep above 0.3 for best results.",
  "tags": ["ambient", "evolving", "drone", "pad"]
}
```

**Every preset file must include:**
- `archetype_origin` — traceability to the source state
- `test_status` — QA sign-off before shipping
- `notes` — at least one sentence of practical context for the user
- All parameters at their full precision — no defaults assumed, no omissions

---

## 14. Agent Responsibilities (AI Workflow Integration)

This spec should guide all agents and the human operator.

### 14.1 Orchestrator Agent

- Enforce this document as an acceptance criteria layer — no DSP feature is "done" until it passes the relevant musical criteria
- Require each feature PR/task to answer the four questions from Section 11 ("Musical Improvement Rule")
- Prioritize fixes that improve usable outcome rate over fixes that improve technical correctness alone (a more correct physical simulation that is less musically usable is not an improvement)

### 14.2 Backend Agent (DSP/CLAP)

- Implement all constraints and guardrails listed in Sections 4.1–4.3 (engine-specific) and Section 10 (DSP-level invariants)
- Report high-risk parameter ranges to the human owner before implementing — do not silently clamp dangerous states without documentation
- Expose parameters in a way that supports musical mapping and recall (Section 11 taper specifications)
- Every time a new parameter is added, add it to the preset format specification simultaneously

### 14.3 Frontend Agent (UI)

- Make control hierarchy obvious — primary, refinement, advanced, performance
- Improve discoverability of anchors and recovery actions
- Ensure labels match perceived sonic effect — run every label through a naive user interpretation test
- Implement the modulation depth arc overlay described in FM-24 mitigation
- Implement the anchor memory system described in Stage F

### 14.4 Graphic Agent

- Support readability, hierarchy, and state clarity above aesthetic preferences when they conflict
- Avoid visuals that hide important status/anchor information (the current archetype origin must always be readable)
- Danger zone marking on controls must be visually distinct without being alarming during normal use

### 14.5 Test Agent

- Run in-context tests (Scenarios A–G), not only algorithm correctness tests
- Track musical failure modes (FM list) as first-class issues, not aesthetic comments
- Report pass/fail with reproduction steps, parameter state snapshot, and audio-context notes
- All test results must be logged using the template in Section 7.3

### 14.6 Human Owner (Creative/Product Director)

- Define what "keep-worthy" sounds mean for ENTROP — this cannot be delegated to any agent
- Approve archetypes, init patches, and engine identities before they are implemented in code
- Decide when a feature is musically valuable vs. merely novel
- Final approval of all 45 preset titles and descriptions before shipping

---

## 15. Acceptance Criteria by Phase

> Phase labels align to the ENTROP Developer Implementation Plan milestones.

### Phase 1 (STOCH Engine + CLAP Wrapper)

Must pass before Phase 2 begins:

- [ ] STOCH init patch passes TTUS target (≤ 60s experienced user)
- [ ] All 5 STOCH archetype presets pass Robustness Test
- [ ] STOCH Cauchy soft-saturation guard implemented and tested (no output > 0 dBFS at any parameter combination)
- [ ] Pitch Snap interpolation produces audible gradient across full 0.0–1.0 range
- [ ] Minimum cycle length guard verified (no aliasing artifacts at extreme Freq Scatter)
- [ ] Sweet-spot density test: at least 6/10 segments pass for Amp Scatter and Freq Scatter
- [ ] In-context phrase play test completed and logged (Scenario A + B)
- [ ] STOCH polyphony voice seeding verified (FM-19: no identical voice evolution)
- [ ] Output soft-limiter verified at all parameter extremes

### Phase 2 (DIFFU Engine)

Must pass before Phase 3 begins:

- [ ] DIFFU init patch in Coral Architecture region — audibly evolving on load
- [ ] All 6 Pearson region presets pass blind A/B audibility test (3/5 listeners)
- [ ] Spectral centroid smoothing implemented (FM-05 prevention)
- [ ] Phase coherence management for partial re-entry verified (no clicks on grid state transitions)
- [ ] Evolution Rate parameter produces audible difference across full range
- [ ] DIFFU polyphony: each voice has independent grid state
- [ ] Long-form static test (Scenario F) passed: audible evolution over 60 seconds without user input
- [ ] Patch edit robustness test passed for all 6 DIFFU archetypes

### Phase 3 (BUBBLE Engine + Effects + Full Blend)

Must pass before v1.0 release:

- [ ] BUBBLE init patch (Oil archetype) passes TTUS target
- [ ] All 5 fluid archetype presets pass identity-consistency test (recognizable after ±20% parameter shift)
- [ ] Bubble population budget per voice: 128-voice ceiling verified
- [ ] Spawn jitter implemented (FM-12 prevention: no mechanical pulse artifacts)
- [ ] Sub-bass 18 Hz HP filter on BUBBLE output verified
- [ ] Tri-morph blend transitions are continuous (no clicks at any blend position)
- [ ] Perceptual loudness normalization implemented (FM-09, FM-25 prevention)
- [ ] Full preset library (45 presets minimum) assembled, all passing Robustness Test
- [ ] Arrangement placement test (Scenario D) passed for at least 5 blend presets
- [ ] Performance expression test (Scenario E) passed across all engines

---

## 16. Musical Improvement Rule for New Features

Every new feature, parameter, mode, or modulation path must answer:

1. **What musical behavior does this enable?**
2. **How does a user reach it within 60 seconds?**
3. **What failure mode can it create (existing FM list or new FM)?**
4. **How is that failure mitigated — DSP guard, UI design, parameter mapping, preset, documentation?**

If these questions cannot be answered clearly before implementation begins, the feature is likely increasing complexity more than musical value. It should be deferred until the answer is clear.

---

## 17. Versioning Notes

This is v0.2. Update after:

- First internal playable prototype (STOCH engine running)
- First external tester session
- Each phase milestone where a new engine is introduced
- Any session that produces new failure modes not in the current FM list

Track changes as:

- **v0.3** — post-first prototype + tester feedback
- **v0.4** — post-DIFFU integration testing
- **v0.5** — post-BUBBLE integration testing
- **v1.0** — stabilized acceptance criteria, full preset library QA-passed, ready for production
