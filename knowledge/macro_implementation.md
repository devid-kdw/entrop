# ENTROP Macro Implementation Specification

**For:** Frontend Agent  
**Status:** Authoritative — do not deviate without orchestrator task  
**Companion docs:** `knowledge/parameter_contract.json` · Musicality Spec §9

---

## Overview

ENTROP exposes 9 macro knobs — 3 per engine. Macros are **CLAP parameters (IDs 33–41)**: they are automatable by the host, visible in automation lanes, and stored in presets. Each macro param value [0.0–1.0] triggers macro_apply() in param_handler.cpp which dispatches coordinated changes to underlying parameters.

Each macro knob movement calls `macro_apply(engine, macro_id, knob_delta)` which computes delta values for each underlying parameter and dispatches them as standard parameter change events — identical to the user moving each underlying knob individually.

**Macro knob range:** `0.0` (fully left) to `1.0` (fully right).  
**Macro knob default:** derived from underlying param defaults on plugin init.

---

## Implementation Pattern

```cpp
// Called from param_handler.cpp when CLAP_EVENT_PARAM_VALUE arrives for IDs 33–41
// macro_pos: the new macro param value [0.0, 1.0]
// All dispatched param changes are additional CLAP events in the same audio block
void macro_apply(MacroID macro, float macro_pos) {
    for (auto& mapping : macro_mappings[macro]) {
        float param_val = mapping.compute(macro_pos);
        dispatch_param_change(mapping.param_id, param_val);
    }
}

// Each mapping defines how one param responds to macro position
struct MacroMapping {
    int param_id;
    float param_min;   // value at macro_pos = 0.0
    float param_max;   // value at macro_pos = 1.0
    TaperType taper;   // applied to macro_pos before mapping
    
    float compute(float macro_pos) {
        float t = apply_taper(macro_pos, taper);
        return param_min + t * (param_max - param_min);
    }
};
```

All dispatched param changes go through the standard interpolation pipeline (≥64 samples). Macros do not bypass smoothing.

---

## STOCH Macros

### Macro S1 — Stability
**"From almost stable to barely controlled chaos"**

Moves Amp Scatter and Freq Scatter together toward the high end (chaos) or low end (stability). Barrier Elasticity moves inversely — higher chaos needs more elasticity to prevent collapse.

| Param | ID | At macro 0.0 | At macro 1.0 | Taper |
|---|---|---|---|---|
| Amp Scatter | 2 | 0.05 | 0.85 | exp_squared |
| Freq Scatter | 3 | 0.05 | 0.80 | exp_squared |
| Barrier Elasticity | 4 | 0.80 | 0.35 | linear (inverse) |

**Rationale:** Low scatter + high elasticity = stable, almost pitched. High scatter + lower elasticity = chaotic but still bounded by elastic barrier. The elasticity floor of `0.35` prevents collapse at full chaos — do not lower it.

**Musical sweet spot:** `0.2–0.6`. Macro positions above `0.75` are intentionally difficult to place in a mix (FM-01 zone).

---

### Macro S2 — Tonality
**"From free pitch chaos to xenharmonically quantized"**

Moves Pitch Snap from 0 (pure stochastic pitch) to 1 (fully quantized to scale). Scale Root moves upward in octave steps as tonality increases, centering the pitch register for melodic use.

| Param | ID | At macro 0.0 | At macro 1.0 | Taper |
|---|---|---|---|---|
| Pitch Snap | 5 | 0.0 | 1.0 | linear |
| Scale Root | 6 | 48 (C3) | 60 (C4) | quantized12, stepwise |

**Scale Root stepping rule:** Scale Root increments by 12 (one octave) at macro positions `0.5` and `0.85`. This shifts the quantized pitch register upward as tonality increases, keeping the instrument in a useful frequency range for melodic playing.

```cpp
// Scale Root stepping
if (macro_pos < 0.5f)       scale_root = 48;   // C3
else if (macro_pos < 0.85f) scale_root = 60;   // C4
else                         scale_root = 60;   // C4 (stays — higher snap)
```

**Note:** Scale Root is an integer param — dispatch as integer MIDI note value, not float.

---

### Macro S3 — Density
**"From sparse angular waves to dense complex contour"**

Moves Segments upward (more breakpoints = denser waveform) while Barrier Elasticity increases in parallel (more segments benefit from higher elasticity to stay coherent).

| Param | ID | At macro 0.0 | At macro 1.0 | Taper |
|---|---|---|---|---|
| Segments | 0 | 3 | 12 | linear, integer snap |
| Barrier Elasticity | 4 | 0.30 | 0.80 | linear |

**Segment snapping:** Segments is an integer param. Round `macro_pos * 9 + 3` to nearest integer.

```cpp
int segments = (int)std::round(macro_pos * 9.0f + 3.0f);  // 3..12
```

**Note:** Barrier Elasticity is shared with Macro S1. If S1 and S3 are both moved, the last-dispatched value wins. This is acceptable — the user controls one macro at a time in practice. Do not add special conflict resolution logic.

---

## DIFFU Macros

### Macro D1 — Phase
**"From static to actively generative"**

Moves F and k together along a diagonal path through Pearson space, visiting musically useful regions in sequence. This is not a linear sweep of F and k independently — it follows a tuned path designed around the six named regions.

**Pearson diagonal path (7 anchor points):**

| Macro pos | F | k | Region |
|---|---|---|---|
| 0.00 | 0.020 | 0.055 | Frozen Field (near-silence) |
| 0.15 | 0.037 | 0.060 | Labyrinthine |
| 0.35 | 0.026 | 0.051 | Chaos |
| 0.55 | 0.062 | 0.061 | Solitons |
| 0.70 | 0.054 | 0.063 | Coral Architecture (default) |
| 0.85 | 0.028 | 0.062 | Mitosis |
| 1.00 | 0.040 | 0.058 | Mitosis/Chaos boundary |

**Implementation:** Linear interpolation between adjacent anchor points. Do not use a single linear sweep — the path is non-linear in F/k space by design.

```cpp
// Anchor table: {macro_pos, F, k}
static const float anchors[][3] = {
    {0.00f, 0.020f, 0.055f},
    {0.15f, 0.037f, 0.060f},
    {0.35f, 0.026f, 0.051f},
    {0.55f, 0.062f, 0.061f},
    {0.70f, 0.054f, 0.063f},
    {0.85f, 0.028f, 0.062f},
    {1.00f, 0.040f, 0.058f},
};

// Find segment, lerp F and k
void macro_d1_apply(float macro_pos) {
    for (int i = 0; i < 6; i++) {
        if (macro_pos <= anchors[i+1][0]) {
            float t = (macro_pos - anchors[i][0]) / (anchors[i+1][0] - anchors[i][0]);
            float F = anchors[i][1] + t * (anchors[i+1][1] - anchors[i][1]);
            float k = anchors[i][2] + t * (anchors[i+1][2] - anchors[i][2]);
            dispatch_param_change(7, F);   // ID 7 = Feed Rate F
            dispatch_param_change(8, k);   // ID 8 = Kill Rate k
            return;
        }
    }
}
```

**Default macro position:** `0.70` (Coral Architecture — matches DIFFU init patch).

---

### Macro D2 — Tempo
**"From glacially slow to faster than bar length"**

Direct mapping of Evolution Rate. Logarithmic taper because perceived time is logarithmic.

| Param | ID | At macro 0.0 | At macro 1.0 | Taper |
|---|---|---|---|---|
| Evolution Rate | 32 | 0.1 | 10.0 | logarithmic |

```cpp
// Logarithmic mapping: param = min * (max/min)^t
float t = macro_pos;  // already 0..1
float evo_rate = 0.1f * std::pow(100.0f, t);  // 0.1 * 100^t → [0.1, 10.0]
dispatch_param_change(32, evo_rate);
```

**Musical reference points:**
- `0.0` (0.1×): state transitions take >30 seconds — geological
- `0.4` (≈1.0×): default — phrase-length evolution
- `0.8` (≈5.0×): rapid restless activity
- `1.0` (10×): near-chaotic speed

---

### Macro D3 — Complexity
**"From few dominant harmonics to dense harmonic cloud"**

Moves Spectral Map Curve (harmonic distribution shape) and Du/Dv diffusion ratio together. Higher complexity = more partials active + greater spatial differentiation in the grid.

| Param | ID | At macro 0.0 | At macro 1.0 | Taper |
|---|---|---|---|---|
| Spectral Map Curve | 11 | 0.15 | 0.90 | sigmoid |
| Diffusion Du | 9 | 0.12 | 0.22 | logarithmic |
| Diffusion Dv | 10 | 0.05 | 0.11 | logarithmic |

**Du/Dv ratio constraint:** Du must always be ≥ 2× Dv. The values above satisfy this across the full macro range (ratio stays between 2.0× and 2.2×). Do not allow macro movement to violate this constraint — clamp Dv to `Du / 2.0` if needed.

**Default macro position:** `0.50` (matches DIFFU init patch Spectral Map 0.5, Du/Dv defaults 0.16/0.08).

---

## BUBBLE Macros

### Macro B1 — Material
**"From light and percussive to massive and sustained"**

Moves Fluid Density and Surface Tension together along the fluid archetype sequence: Water → Oil → Mercury → Magma. Pressure moves slightly upward with density to maintain physical consistency.

**Fluid archetype anchor points:**

| Macro pos | Density (kg/m³) | Surface Tension (N/m) | Pressure (Pa) | Archetype |
|---|---|---|---|---|
| 0.00 | 500 | 0.010 | 80000 | Plasma |
| 0.20 | 1000 | 0.073 | 101325 | Water |
| 0.45 | 870 | 0.032 | 101325 | Oil (default) |
| 0.65 | 2700 | 0.400 | 120000 | Magma |
| 1.00 | 13600 | 0.485 | 150000 | Mercury |

**Implementation:** Same anchor-based lerp as Macro D1. Density and Surface Tension use logarithmic interpolation (both are log-taper params); Pressure is linear.

```cpp
// Log interpolation for density and surface tension
float log_lerp(float a, float b, float t) {
    return std::exp(std::log(a) + t * (std::log(b) - std::log(a)));
}
```

**Default macro position:** `0.45` (Oil — matches BUBBLE init patch).

**Note:** Plasma (`0.0`) is intentionally at the aggressive end. Full-left macro = hostile sound by design.

---

### Macro B2 — Population
**"From sparse individual bubbles to dense swarm"**

Moves Spawn Rate and Mean Radius together. Higher population = more bubbles + smaller mean size (higher frequency center). This creates a perceptual shift from individual resonant events to a continuous granular texture.

| Param | ID | At macro 0.0 | At macro 1.0 | Taper |
|---|---|---|---|---|
| Spawn Rate | 18 | 1 Hz | 200 Hz | logarithmic |
| Mean Radius | 16 | 0.020 m (20mm) | 0.0005 m (0.5mm) | logarithmic (inverse) |

**Inverse relationship:** As Population increases, Mean Radius decreases (smaller, higher-frequency bubbles dominate a dense swarm). This is the physically correct behavior.

```cpp
// Spawn Rate: log up
float spawn = 1.0f * std::pow(200.0f, macro_pos);     // 1..200 Hz

// Mean Radius: log down (inverse)
float radius = 0.020f * std::pow(0.025f, macro_pos);  // 20mm..0.5mm
```

**128-bubble ceiling reminder:** The 128-bubble ceiling per voice is enforced in the backend regardless of Spawn Rate. At high Population macro, the ceiling will be hit quickly — this is expected behavior, not a bug. Do not suppress the macro range to avoid hitting the ceiling.

**Default macro position:** `0.30` (matches BUBBLE init: Spawn Rate 12 Hz, Mean Radius 4mm).

---

### Macro B3 — Depth
**"From individual oscillators to cloud with emergent sub-bass"**

Moves Coupling Coefficient and Pressure together. Higher depth = stronger coupling between bubbles (emergent collective behavior) + higher pressure (lower resonant frequencies for all bubbles simultaneously).

| Param | ID | At macro 0.0 | At macro 1.0 | Taper |
|---|---|---|---|---|
| Coupling Coefficient | 19 | 0.0 | 0.80 | exp_squared |
| Pressure | 15 | 60000 Pa | 180000 Pa | linear |

**Coupling ceiling:** Macro B3 caps Coupling at `0.80`, not `1.0`. Full coupling (`1.0`) is reserved for direct parameter access only — it produces extreme behavior that is musically useful as a deliberate choice but dangerous as an accidental macro overshoot.

**Pressure effect on frequency:** Higher pressure lowers bubble resonant frequency (Minnaert formula: f ∝ 1/r · √(3γP/ρ)). At macro `1.0`, all bubble frequencies shift down by approximately one octave relative to macro `0.0`. This is the "sub-bass emergence" behavior.

**Default macro position:** `0.15` (matches BUBBLE init: Coupling 0.15, Pressure 101325 Pa).

---

## Macro Position Derivation on Preset Load

Presets store both macro param values (IDs 33–41) and underlying param values. On normal preset load, macro positions are read directly from the preset — no derivation needed.

Derivation is only needed when loading a legacy preset (created before the macro system) or when the user has edited underlying params directly without touching the macros. In that case, derive macro positions by inverting the mapping:

```cpp
// Example: derive Stability macro position from Amp Scatter + Freq Scatter
float stability_pos = std::sqrt((amp_scatter - 0.05f) / (0.85f - 0.05f));  // invert exp_squared

// Example: derive Tempo macro position from Evolution Rate
float tempo_pos = std::log(evo_rate / 0.1f) / std::log(100.0f);  // invert log mapping
```

For anchor-based macros (D1 Phase, B1 Material), find the nearest segment and interpolate the macro position. If params are not on the path (user moved underlying params directly), snap to the nearest point on the path.

If derivation produces a value outside `[0.0, 1.0]`, clamp to range — this indicates the underlying params are in a region not covered by the macro path, which is valid.

---

## What Macros Do Not Control

These parameters have no macro mapping. They are always directly accessible:

- Distribution (ID 1) — too discrete for smooth macro travel
- Grid Reset Trigger (ID 12) — bang, not continuous
- CA Rule / CA Rate / LFO Target / LFO Depth (IDs 23–26) — LFO system
- Waveshaper H2/H3/H4-H6 (IDs 27–29) — FX chain
- Reverb Size / Reverb Mix (IDs 30–31) — FX chain
- Engine Blend S↔D / D↔B (IDs 20–21) — tri-morph blend is its own control
- Master Gain (ID 22)
- Radius Spread (ID 17)
- Scale Root (ID 6) — partially controlled by S2 Tonality, but also direct

---

## Agent Constraints

- **Macros are CLAP parameters (IDs 33–41).** They are visible to the host and automatable.
- **Presets store macro positions alongside underlying params.** Legacy presets without macro values use derivation (see above).
- **Do not bypass parameter smoothing.** All macro-dispatched changes go through the standard ≥64 sample interpolation.
- **Do not add macro-to-macro conflict resolution logic** beyond the note about Barrier Elasticity sharing between S1 and S3. Last-write wins is correct behavior.
- **If a macro computation would push a param outside its `[min, max]` range**, clamp to range and do not report an error. This is expected at macro extremes.
- **Escalate** if a musical listening session reveals that a macro path feels wrong (dead zones, sudden jumps). Do not adjust anchor values without a dedicated task and human approval.

---

*ENTROP Macro Implementation Spec v1.0*  
*Anchor values are tuned estimates — expect refinement after first Sonic Test P1-D / P2-D sessions.*
