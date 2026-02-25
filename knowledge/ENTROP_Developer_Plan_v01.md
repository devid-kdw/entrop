# ENTROP — Developer Implementation Plan v0.1

**Plugin format:** CLAP (native)  
**Target platform:** macOS · Apple Silicon (M-series)  
**Toolchain:** C++20 · Xcode/clang++ · Apple Accelerate · CLAP SDK  
**Status:** Working specification — authoritative technical reference  

> **FOR AI AGENTS:** This document is the single source of truth for all ENTROP DSP and CLAP implementation decisions. Do not deviate from formulas, parameter IDs, ranges, or architectural decisions described here without an explicit orchestrator task authorizing the change. When a section provides pseudocode, implement it as written — do not substitute alternative algorithms. When a section specifies an Apple Accelerate function, use that function — do not substitute a scalar loop or a different SIMD library.

---

## Document Map

| Section | Content | Primary Agent |
|---|---|---|
| 1 | Project overview and market differentiation | Orchestrator |
| 2 | CLAP architecture — thread pool, MPE modulation | Backend |
| 3 | Engine 1: STOCH — GENDYN stochastic synthesis | Backend |
| 4 | Engine 2: DIFFU — Gray-Scott reaction-diffusion | Backend |
| 5 | Engine 3: BUBBLE — Minnaert fluid-acoustic resonance | Backend |
| 6 | Modulation system — ECA LFO, MPE routing | Backend |
| 7 | Effects — Hermite waveshaper, Hadamard FDN reverb | Backend |
| 8 | Development phases — milestones and timelines | Orchestrator |
| 9 | Complete parameter map — all 32 parameters | All agents |
| 10 | Toolchain and dependencies | Backend |
| Appendix | Key academic and technical references | All agents |

---

## 1. Project Overview

ENTROP is a CLAP-native synthesizer plugin built entirely on physical, stochastic, and mathematical sound generation systems that have no direct equivalent in any commercial product. It deliberately avoids all standard synthesis paradigms — no wavetables, no classic FM, no analog emulation. Every sound is an emergent property of the underlying mathematical or physical system.

The plugin exposes three independent synthesis engines, each with its own character and computational model. Engines can be blended and layered. All modulation is routed through CLAP's per-voice non-destructive parameter offset system, enabling MPE controllers to directly manipulate physical parameters per note.

### 1.1 Market Differentiation

| Plugin / Product | Method | Gap vs. ENTROP |
|---|---|---|
| Xenos (free) | GENDYN / Stochastic only | No Reaction-Diffusion, no Bubble physics, no CLAP, standalone stochastic only |
| Protean (sonicLAB, €139) | GPU particle fluid → FM/additive | Standard FM oscillators driven by particles — not fluid physics as oscillator |
| AudioThing Bubbles | Bubble-styled band-pass filter | Visual metaphor only — no Minnaert resonance, no physics |
| Any commercial synth | Wavetable / FM / subtractive | None implement Gray-Scott R-D or Minnaert bubble as primary oscillator |

---

## 2. CLAP Architecture & Infrastructure

### 2.1 Why CLAP Is Non-Negotiable For This Plugin

The three engines in ENTROP have processing demands that traditional plugin formats cannot satisfy without major compromises. CLAP resolves each critical bottleneck at the ABI level.

| CLAP Feature | ENTROP Usage | VST3/AU Equivalent |
|---|---|---|
| `clap_host_thread_pool` | Distribute Gray-Scott grid iteration and bubble population evaluation across all physical cores per process block | Manual internal thread management — causes CPU contention with multiple instances |
| Per-voice non-destructive parameter offsets | MPE axes (pressure, slide, Y) map to physical parameters per note without overwriting automation lane values | Global automation only or destructive MPE data overwrite |
| `clap_plugin_gui` physical/logical pixel API | Render live Gray-Scott grid and bubble cloud visualisations at native Retina resolution | No built-in logical/physical pixel distinction |
| Project file consolidation extension | Save generated stochastic state and R-D grid snapshots inside the DAW project file | Absolute/relative path dependencies on disk |

### 2.2 Thread Pool Integration

Register the thread pool host extension at plugin init. During the `process()` callback, dispatch independent per-voice workloads via `request_exec`. Do not spawn any internal `std::thread` — all parallelism must go through the host pool.

> **AGENT CONSTRAINT:** Never call `std::thread` constructor inside `process()` or any function it calls. Use `clap_host_thread_pool` exclusively. Implement the serial fallback path — it is required, not optional.

```cpp
// Plugin init — query thread pool extension
auto *host_tp = static_cast<const clap_host_thread_pool_t *>(
    host->get_extension(host, CLAP_EXT_THREAD_POOL));

// In process() — dispatch N tasks (one per active voice)
if (host_tp && host_tp->request_exec(host, active_voice_count)) {
    // Each task_index maps to one voice in the pool callback
    // clap_plugin_thread_pool_t::exec() is called per task
} else {
    process_all_voices_serial(); // fallback — always implement this
}

// Thread pool callback — called from host worker threads
void ENTROP::on_thread_pool_exec(uint32_t task_index) {
    voices[task_index].process_block(current_buffer);
}
```

> **NOTE:** Use `clap_plugin_thread_check` extension during development to validate which thread each operation runs on. Accessing the GUI from a worker thread is undefined behaviour.

### 2.3 Per-Voice MPE Modulation Routing

CLAP's parameter offset system means MPE axes never destroy the static base value stored in the automation lane. This is critical for ENTROP because physical parameters (fluid density, distribution type) must return to their configured values after note release.

> **AGENT CONSTRAINT:** MPE modulation applies as an additive offset to the base parameter value. It never overwrites `base_value`. After note release, the effective value returns to `base_value` automatically because the offset source is gone.

```cpp
// Parameter descriptor — mark as per-voice modulatable
clap_param_info_t info {
    .id    = PARAM_FLUID_DENSITY,
    .flags = CLAP_PARAM_IS_MODULATABLE | CLAP_PARAM_IS_MODULATABLE_PER_NOTE_ID,
    .min_value = 0.0, .max_value = 1.0, .default_value = 0.5,
    .name  = "Fluid Density"
};

// In process() — read modulation events from event queue
for (auto &event : input_events) {
    if (event.type == CLAP_EVENT_PARAM_MOD) {
        auto *mod = reinterpret_cast<const clap_event_param_mod_t *>(&event);
        // mod->amount is the OFFSET added to the static base value
        // mod->note_id isolates this to a single polyphonic voice
        voice_for_note(mod->note_id).apply_mod_offset(mod->param_id, mod->amount);
    }
}
```

---

## 3. Engine 1 — STOCH

**Type:** Dynamic Stochastic Synthesis · GENDYN Architecture  
**Computational cost:** Lowest of the three engines  
**Implementation order:** First — Phase 1

STOCH is a direct C++ implementation of Xenakis's GENDYN algorithm (1991), extended with pitch quantization and multiple probability distributions.

### 3.1 Core GENDYN Algorithm

GENDYN generates audio by directly manipulating the pressure curve — it operates on individual breakpoints of a waveform cycle, not on Fourier components. Each cycle, every breakpoint moves in both time and amplitude according to a chosen probability distribution applied through elastic barrier reflection.

#### 3.1.1 Breakpoint State Structure

```cpp
struct Breakpoint {
    float time;       // Normalised position [0.0, 1.0] within the cycle
    float amplitude;  // Sample value [-1.0, 1.0]
};

struct GENDYNVoice {
    static constexpr int MAX_SEGMENTS = 12;
    std::array<Breakpoint, MAX_SEGMENTS> points;
    int    num_segments   = 8;
    float  sample_phase   = 0.0f;  // Current read position within cycle
    float  cycle_freq_hz  = 110.0f;
    float  sample_rate    = 48000.0f;
    float  phase_inc      = 0.0f;  // Recomputed when freq changes
};
```

#### 3.1.2 Probability Distribution Selection

The distribution applied to each breakpoint displacement is the **primary timbral control**. Implement all four and expose as an enum parameter (see parameter map section 9, ID 1).

| Distribution | Character | Typical Use |
|---|---|---|
| Uniform `U(-b, b)` | Even spread — gentle, unpredictable wobble | Pads, slow evolving tones |
| Gaussian `N(0, σ)` | Clustered around centre — smooth fluctuation | Organic timbres, natural textures |
| Cauchy `C(0, γ)` | Heavy tails — occasional violent jumps | Aggressive noise bursts, neurofunk basses |
| Logistic `L(0, s)` | Between Gaussian and Cauchy in kurtosis | IDM, transitional textures |

> **AGENT CONSTRAINT — CAUCHY SAFETY:** The Cauchy distribution has no finite variance. Output values can be arbitrarily large before elastic barrier reflection. Apply `tanh(x / 0.8) * 0.8` soft saturation on STOCH engine output after the barrier, before the engine mixer. This prevents output spikes while preserving the violent character. Do not use hard clipping.

```cpp
// Sample from Cauchy distribution (heavy-tailed — use for aggressive textures)
// tan() of uniform [-π/2, π/2] gives Cauchy(0,1)
inline float sample_cauchy(float gamma, std::mt19937 &rng) {
    std::uniform_real_distribution<float> u(-0.4999f * M_PI, 0.4999f * M_PI);
    return gamma * std::tan(u(rng));
}

// Sample from Gaussian (Box-Muller transform)
inline float sample_gaussian(float sigma, std::mt19937 &rng) {
    std::uniform_real_distribution<float> u(1e-7f, 1.0f);
    float u1 = u(rng), u2 = u(rng);
    return sigma * std::sqrt(-2.0f * std::log(u1)) * std::cos(2.0f * M_PI * u2);
}
```

#### 3.1.3 Elastic Barrier Reflection

After displacing each breakpoint by a random amount, apply elastic reflection at the boundary to keep values in range. This is what Xenakis called the 'elastic barrier' — the reflection introduces waveform shaping without hard clipping.

> **AGENT CONSTRAINT — MINIMUM CYCLE LENGTH:** At very high Freq Scatter values, breakpoints can collapse to zero-length cycle segments, producing aliased artifacts. Enforce a minimum time separation between adjacent breakpoints of `4 / sample_rate` seconds. This guard is silent — it never activates under normal use.

```cpp
// Elastic barrier reflection — keeps value in [lo, hi]
// If the value overshoots, reflect from the boundary
float elastic_reflect(float value, float lo, float hi) {
    float range = hi - lo;
    if (range <= 0.0f) return lo;
    value -= lo;
    value = std::fmod(value, 2.0f * range);
    if (value < 0.0f)   value += 2.0f * range;
    if (value > range)  value = 2.0f * range - value;
    return value + lo;
}

// Apply per breakpoint, per cycle
bp.amplitude = elastic_reflect(bp.amplitude + delta_amp, -1.0f, 1.0f);
bp.time      = elastic_reflect(bp.time      + delta_t,   0.0f, 1.0f);
```

#### 3.1.4 Per-Cycle Pitch Quantization

After computing the new breakpoint positions, derive the current cycle's effective fundamental frequency, then snap it to the nearest pitch in the active microtonal scale. This produces xenharmonic stability from within chaotic motion.

> **AGENT CONSTRAINT — SNAP INTERPOLATION:** When Pitch Snap transitions from 0.0 to 1.0, quantization must not appear as a stepped jump. Implement as a weighted interpolation: `effective_freq = lerp(raw_gendyn_freq, quantized_freq, pitch_snap_param)`. This creates a musical gradient from free chaos to full xenharmonic quantization.

```cpp
// Xenharmonic pitch quantization — snap Hz to nearest scale degree
// scale_cents: sorted array of cent offsets from root (e.g. 31-TET)
float quantize_to_scale(float freq_hz, float root_hz,
                        const std::vector<float> &scale_cents) {
    float cents_from_root = 1200.0f * std::log2(freq_hz / root_hz);
    float octave = std::floor(cents_from_root / 1200.0f);
    float frac   = cents_from_root - octave * 1200.0f;
    auto it = std::lower_bound(scale_cents.begin(), scale_cents.end(), frac);
    if (it != scale_cents.begin()) {
        auto prev = std::prev(it);
        if (std::fabs(*prev - frac) < std::fabs(*it - frac)) it = prev;
    }
    float quantized_cents = octave * 1200.0f + *it;
    return root_hz * std::pow(2.0f, quantized_cents / 1200.0f);
}
```

### 3.2 STOCH Parameter Map

| Parameter ID | Range | Physical Meaning | MPE Mapping |
|---|---|---|---|
| `STOCH_SEGMENTS` (ID 0) | 3–12 (int) | Number of breakpoints per waveform cycle | — |
| `STOCH_DISTRIBUTION` (ID 1) | 0–3 (enum) | Uniform / Gaussian / Logistic / Cauchy |Y-axis: stepped selection (0=Uniform, 1=Gaussian, 2=Logistic, 3=Cauchy) |
| `STOCH_AMP_SCATTER` (ID 2) | 0.0–1.0 | σ / γ scale for amplitude perturbation | Pressure: modulate per voice |
| `STOCH_FREQ_SCATTER` (ID 3) | 0.0–1.0 | σ / γ scale for time-axis perturbation | Slide: modulate per voice |
| `STOCH_BARRIER_ELAST` (ID 4) | 0.0–1.0 | Coefficient of restitution at elastic walls | — |
| `STOCH_PITCH_SNAP` (ID 5) | 0.0–1.0 | Interpolation between free chaos and full quantization | — |
| `STOCH_SCALE_ROOT` (ID 6) | MIDI 0–127 | Root pitch for xenharmonic quantization | — |

### 3.3 STOCH Voice Seeding

> **AGENT CONSTRAINT:** Each polyphonic voice must be seeded with a unique `std::mt19937` seed. Derive seed as `(base_seed * (voice_index + 1)) XOR (note_on_timestamp_mod_65536)`. Never share a PRNG instance between voices. Never use a fixed seed that is the same across all voices — this would cause all voices to evolve identically.

---

## 4. Engine 2 — DIFFU

**Type:** Reaction-Diffusion Spectral Synthesis · Gray-Scott Model  
**Computational cost:** Medium — requires vDSP vectorisation to meet budget  
**Implementation order:** Second — Phase 2

DIFFU evolves a 1D Gray-Scott reaction-diffusion system and maps the spatial concentration of chemical V directly onto the amplitudes of an additive oscillator bank. No equivalent exists in any commercial synthesizer.

### 4.1 Gray-Scott Equations

The Gray-Scott model describes two interacting chemical species U and V on a 1D grid of N cells. U is fed from outside; V consumes U and degrades. The competition between feed rate F and kill rate k produces dramatically different spatial patterns.

```
dU/dt = Du·∇²U  −  U·V²  +  F·(1 − U)
dV/dt = Dv·∇²V  +  U·V²  −  (F + k)·V
```

> **AGENT CONSTRAINT — CORRECTNESS BEFORE OPTIMISATION:** Implement the scalar version first. Validate all six Pearson regions visually before writing any vDSP code. Optimising incorrect simulation produces fast incorrect output that is harder to debug.

```cpp
// 1D discrete Laplacian (Neumann boundary conditions)
inline float laplacian_1d(const float *g, int i, int N) {
    int left  = (i > 0)   ? i - 1 : i;
    int right = (i < N-1) ? i + 1 : i;
    return g[left] + g[right] - 2.0f * g[i];
}

void gray_scott_step(float *U, float *V, float *Un, float *Vn,
                     int N, float Du, float Dv, float F, float k, float dt) {
    for (int i = 0; i < N; ++i) {
        float uvv   = U[i] * V[i] * V[i];
        float lap_u = laplacian_1d(U, i, N);
        float lap_v = laplacian_1d(V, i, N);
        Un[i] = U[i] + dt * (Du * lap_u - uvv + F * (1.0f - U[i]));
        Vn[i] = V[i] + dt * (Dv * lap_v + uvv - (F + k) * V[i]);
        Un[i] = std::clamp(Un[i], 0.0f, 1.0f);
        Vn[i] = std::clamp(Vn[i], 0.0f, 1.0f);
    }
    std::swap(Un, Vn);  // pointer swap — no allocation
}
```

### 4.2 Pearson Parameter Space — Known Phase Regions

The F/k plane has been extensively mapped by Pearson (1993). These coordinates are authoritative — use them exactly as named preset snap points on the XY pad.

| Region Name | F | k | Sonic Character |
|---|---|---|---|
| Uniform (silence) | 0.020 | 0.055 | Grid converges to uniform — near silence, useful as pad release tail |
| Solitons (U-skate) | 0.062 | 0.061 | Self-replicating blobs — periodic spectral peaks with slow drift |
| Mitosis | 0.028 | 0.062 | Spots divide and multiply — dense harmonic activity, chaotic mid-range |
| Coral / Worms | 0.054 | 0.063 | Branching patterns — complex but stable harmonic structures |
| Labyrinthine | 0.037 | 0.060 | Maze-like — dense, evolving tonal noise floor |
| Chaos | 0.026 | 0.051 | Fully chaotic — broadband noise with emergent tonal flashes |

> **NOTE:** Expose F and k as continuous parameters. Map them on a 2D XY pad in the GUI. Pre-label the six Pearson regions as named presets so users can snap to them while still navigating freely.

### 4.3 Spectral Mapping — R-D Grid to Additive Bank

The 1D grid of N cells (N = 128 recommended) is iterated at control rate (every 64–256 samples). The V concentration at each cell i maps directly to the amplitude of additive partial i.

> **AGENT CONSTRAINT — PHASE COHERENCE:** When a partial's amplitude drops below `0.001`, freeze its phase accumulator. When amplitude rises again, the partial re-enters at the frozen phase — not at a random phase. Random-phase re-entry produces audible clicks. Phase freezing is mandatory.

> **AGENT CONSTRAINT — SPECTRAL CENTROID SMOOTHING:** Apply a 20ms exponential moving average to the amplitude vector feeding the additive bank. Sudden grid state changes must arrive as fast sweeps, not instant jumps. This prevents harsh transient brightness that reads as a technical artifact.

```cpp
// Map R-D grid V concentration → additive bank amplitudes
// Logarithmic frequency spacing matches human pitch perception
void rd_to_additive(const float *V_grid, float *partial_amps,
                    int num_partials, float fundamental_hz, float sample_rate) {
    for (int i = 0; i < num_partials; ++i) {
        // Log-spaced partials: partial 0 = fundamental, last = nyquist/2
        // Range: 6 octaves above fundamental
        float ratio = std::pow(2.0f, (float)i / (float)(num_partials - 1) * 6.0f);
        float freq  = fundamental_hz * ratio;
        if (freq >= sample_rate * 0.49f) { partial_amps[i] = 0.0f; continue; }
        // Squared for perceptual linearisation of V concentration
        partial_amps[i] = V_grid[i] * V_grid[i];
    }
}
```

### 4.4 Performance — Apple Accelerate Integration

The Gray-Scott step is the hottest loop. Vectorise using Accelerate's vDSP functions after the scalar correctness baseline is confirmed.

**Performance budget:**

| Operation | Grid Size | Target Time (per block @48kHz) | Method |
|---|---|---|---|
| Gray-Scott step | 128 cells | < 0.05ms | vDSP vectorised float ops |
| Additive bank render | 128 partials | < 0.2ms per voice | Phase accumulator + `vDSP_vsincos` |
| Per-voice total (16 voices) | — | < 4ms total | CLAP thread pool (parallel) |

```cpp
#include <Accelerate/Accelerate.h>

// Vectorised Gray-Scott step using vDSP
// U·V² — compute V² first, then multiply element-wise with U
vDSP_vmul(V, 1, V, 1, V_sq, 1, N);          // V_sq = V * V
vDSP_vmul(U, 1, V_sq, 1, uvv, 1, N);         // uvv  = U * V²

// dU = Du*lapU - uvv + F*(1-U)
// Decompose into: vDSP_vsadd, vDSP_vsmul, vDSP_vadd
// All operate on float arrays of length N — no scalar loops
```

### 4.5 DIFFU Parameter Map

| Parameter ID | Range | Physical Meaning | MPE Mapping |
|---|---|---|---|
| `DIFFU_FEED_F` (ID 7) | 0.01–0.08 | Feed rate F | Pressure |
| `DIFFU_KILL_K` (ID 8) | 0.04–0.07 | Kill rate k | Slide |
| `DIFFU_DU` (ID 9) | 0.10–0.25 | Diffusion coefficient U | Y-axis |
| `DIFFU_DV` (ID 10) | 0.04–0.12 | Diffusion coefficient V | — |
| `DIFFU_SPECTRAL_CURVE` (ID 11) | 0.0–1.0 | Spectral map curve shape | — |
| `DIFFU_GRID_RESET` (ID 12) | Bang | Reset grid to initial state | — |
| `DIFFU_EVO_RATE` (ID 32) | 0.1×–10× | Time-step dt multiplier for R-D grid evolution | — |

---

## 5. Engine 3 — BUBBLE

**Type:** Fluid-Acoustic Resonance · Minnaert + Coupled Oscillator Model  
**Computational cost:** Medium-high — population management, per-bubble oscillators  
**Implementation order:** Third — Phase 3

BUBBLE derives pitch and timbre entirely from the physics of resonating gas bubbles in a fluid medium. Each voice maintains a stochastic population of bubble oscillators whose frequency is computed directly from the Minnaert resonance formula. This has no equivalent in any commercial plugin.

### 5.1 Minnaert Resonance — Core Formula

The natural resonant frequency of a spherical gas bubble of radius `a` in a fluid of density `ρ` at ambient pressure `p_A`:

```
f = (1 / 2πa) · sqrt(3γp_A / ρ)

a   — bubble radius (metres)
γ   — polytropic coefficient of gas (1.4 for air)
p_A — ambient pressure (Pa) — maps to 'depth' control
ρ   — fluid density (kg/m³) — water: 1000, mercury: 13600
```

**Verification fixture:** 1mm air bubble in water must produce `f ≈ 3.26 kHz`. Use this as the correctness check before proceeding to the population model.

```cpp
inline float minnaert_freq(float radius_m, float gamma,
                            float pressure_pa, float density) {
    return (1.0f / (2.0f * M_PI * radius_m)) *
           std::sqrt(3.0f * gamma * pressure_pa / density);
}

// Young-Laplace surface tension correction
// Corrected effective pressure: p_eff = p_A + 2σ/a
inline float minnaert_freq_with_surface_tension(
    float radius_m, float gamma, float pressure_pa,
    float density, float surface_tension_nm) {
    float p_eff = pressure_pa + 2.0f * surface_tension_nm / radius_m;
    return minnaert_freq(radius_m, gamma, p_eff, density);
}
```

### 5.2 Bubble Population Model

Each note spawns a stochastic population of bubble oscillators. The stochastic birth/death process is evaluated per audio block.

> **AGENT CONSTRAINT — POPULATION CEILING:** Hard maximum of 128 simultaneously active bubble oscillators per polyphonic voice. When budget is full, new spawns displace the oldest active bubbles (FIFO eviction). Evicted bubbles perform a 5ms fade-out to avoid clicks. This ceiling applies **per voice**, not globally.

> **AGENT CONSTRAINT — SPAWN JITTER:** Each scheduled spawn event must be displaced by ±15% of its nominal inter-spawn interval using a uniform distribution. This prevents mechanical regularity at moderate spawn rates (10–50 Hz), which the ear locks onto as a machine-like pulse rather than a natural fluid process.

> **AGENT CONSTRAINT — FREQUENCY CEILING:** Silently discard any bubble whose computed Minnaert frequency exceeds `sample_rate * 0.45`. These partials are inaudible and consume DSP budget.

> **AGENT CONSTRAINT — DECAY MINIMUM:** Enforce a maximum ring time of 8 seconds per bubble regardless of surface tension parameter. Low surface tension values (Plasma preset) otherwise allow bubbles to ring indefinitely, accumulating to dangerous amplitude levels over time.

```cpp
struct BubbleOscillator {
    float freq_hz;     // Computed from Minnaert for this bubble's radius
    float phase;       // Current sample phase [0, 2π]
    float amplitude;   // Peak amplitude for this oscillator
    float decay_rate;  // Exponential decay coefficient (viscous damping)
    float age;         // Samples since birth — used for envelope
    bool  alive;
};

// Spawn a new bubble — radius drawn from log-normal distribution
// Log-normal matches measured bubble size distributions in real fluids
BubbleOscillator spawn_bubble(float mean_radius, float radius_spread,
                              float fluid_density, float surface_tension,
                              float pressure, std::mt19937 &rng) {
    std::lognormal_distribution<float> rd(std::log(mean_radius), radius_spread);
    float r = std::clamp(rd(rng), 1e-5f, 0.05f); // 0.01mm – 50mm
    float f = minnaert_freq_with_surface_tension(r, 1.4f, pressure,
                                                  fluid_density, surface_tension);
    float decay = fluid_density * 0.0001f + surface_tension * 0.01f;
    return { f, 0.0f, 1.0f / std::log2(f + 2.0f), decay, 0.0f, true };
}
```

### 5.3 Fluid Preset Archetypes

Expose named fluid presets that set density, surface tension, and pressure simultaneously. These are the authoritative archetype values — use them exactly.

| Preset | Density (kg/m³) | Surface Tension (N/m) | Sonic Result |
|---|---|---|---|
| Water | 1000 | 0.073 | High-frequency granular clicks and pops — percussive textures |
| Oil | 870 | 0.032 | Warmer, slower bubble formation — mid-range resonant plucks |
| Mercury | 13600 | 0.485 | Very low fundamental — dense, heavy sub-bass tones |
| Magma (sim) | 2700 | 0.400 | Enormous slow bubbles — deep, massive sub-bass drones |
| Plasma (sim) | 500 | 0.010 | Fast erratic high-frequency — piercing aggressive leads |

### 5.4 Bubble Cloud Coupling Effect

When bubble population density is high, neighbouring oscillators affect each other's frequency through pressure coupling. This produces emergent sub-bass frequencies below any individual bubble's Minnaert frequency — a real physical phenomenon of bubble clouds.

> **AGENT CONSTRAINT — FEATURE FLAG:** Implement coupling behind a compile-time flag `ENTROP_BUBBLE_COUPLING`. This allows isolation and testing without destabilising the base population model.

> **AGENT CONSTRAINT — SUB-BASS PROTECTION:** Apply a 6 dB/octave high-pass filter at 18 Hz on the BUBBLE engine output. Sub-bass below 18 Hz is inaudible and represents wasted headroom.

```cpp
// Mean-field coupling — each bubble sees the mean pressure of its neighbours
void apply_bubble_cloud_coupling(std::vector<BubbleOscillator> &bubbles,
                                  float coupling_coefficient) {
    if (bubbles.empty() || coupling_coefficient < 1e-4f) return;
    float mean_freq = 0.0f;
    for (auto &b : bubbles) if (b.alive) mean_freq += b.freq_hz;
    mean_freq /= bubbles.size();
    for (auto &b : bubbles) {
        if (!b.alive) continue;
        float delta = (mean_freq - b.freq_hz) * coupling_coefficient * 0.01f;
        b.freq_hz  += delta;
    }
}
```

### 5.5 BUBBLE Parameter Map

| Parameter ID | Range | Physical Meaning | MPE Mapping |
|---|---|---|---|
| `BUBBLE_DENSITY` (ID 13) | 500–14000 kg/m³ | Fluid density | — |
| `BUBBLE_TENSION` (ID 14) | 0.01–0.50 N/m | Surface tension | Slide |
| `BUBBLE_PRESSURE` (ID 15) | 50000–200000 Pa | Ambient pressure | — |
| `BUBBLE_RADIUS` (ID 16) | 0.1mm–40mm | Mean bubble radius | Slide |
| `BUBBLE_SPREAD` (ID 17) | 0.1–2.0 (log-normal σ) | Radius distribution spread | — |
| `BUBBLE_SPAWN` (ID 18) | 1–500 Hz | Spawn rate | Pressure |
| `BUBBLE_COUPLING` (ID 19) | 0.0–1.0 | Cloud coupling coefficient | Y-axis |

---

## 6. Modulation System

### 6.1 Cellular Automata LFO

LFO shapes in ENTROP are not sine or triangle waves. They are generated by evaluating 1D Elementary Cellular Automata (ECA) rules at control rate (every 64 samples). Rule selection determines the LFO's degree of periodicity vs. chaos.

| ECA Rule | Behaviour | LFO Character |
|---|---|---|
| Rule 30 | Provably chaotic — fails all randomness tests | Aperiodic, erratic — replaces random LFO entirely |
| Rule 90 | Self-similar fractal (Pascal's triangle mod 2) | Fractal periodicity — irregular but structured |
| Rule 110 | Universal computation — mix of periodic & chaotic | Slow long-period cycles with embedded chaotic bursts |
| Rule 150 | Periodic with symmetric structure | Symmetric waves, good for tremolo/vibrato replacement |

```cpp
// 1D Elementary Cellular Automaton — 8-bit rule number
// State is a bitset of width cells; next state per cell from 3-cell neighbourhood
void eca_step(uint8_t *state, uint8_t *next_state, int width, uint8_t rule) {
    for (int i = 0; i < width; ++i) {
        int left  = (i > 0)       ? i - 1 : width - 1; // wrap
        int right = (i < width-1) ? i + 1 : 0;
        uint8_t neighbourhood = (state[left] << 2) | (state[i] << 1) | state[right];
        next_state[i] = (rule >> neighbourhood) & 1;
    }
}

// Convert ECA row to LFO sample via centre-of-mass
float eca_row_to_float(const uint8_t *row, int width) {
    float sum = 0.0f;
    for (int i = 0; i < width; ++i) sum += row[i];
    return (sum / width) * 2.0f - 1.0f; // normalise to [-1, 1]
}
```

### 6.2 MPE Physical Parameter Routing

| MPE Axis | STOCH Target | DIFFU Target | BUBBLE Target |
|---|---|---|---|
| Note pressure (channel aftertouch) | Amplitude scatter σ — more pressure = more chaotic amplitude jumps | Feed rate F — more pressure = accelerate chemical reaction | Spawn rate — more pressure = denser bubble population |
| Slide (pitch bend / X-axis) | Frequency scatter σ — slide = microtonal drift intensity | Kill rate k — slide across phase boundary in Pearson space | Mean bubble radius — slide = shift fundamental frequency |
| Y-axis (MIDI CC 74) | Distribution type step — top = Cauchy (3), center-up = Logistic (2), center-down = Gaussian (1), bottom = Uniform (0). Full-scale Y divided into 4 equal zones. | Diffusion ratio Du/Dv — affect speed of pattern propagation | Coupling coefficient — Y-axis increases cloud coupling effect |

> **AGENT CONSTRAINT — MPE DEPTH CALIBRATION:** Default MPE modulation depth must be calibrated so that full-scale expression (0→127) maps to the musically useful travel of the target parameter, not its full mathematical range. Full-range MPE expression causing instant tonal collapse is a musical failure mode (FM-22 in Musicality Spec).

### 6.3 Macro Control System

Macro controls are GUI-layer widgets only — they have no CLAP parameter IDs and are not automatable by the host. Each macro translates user movement into proportional simultaneous changes across multiple underlying parameters. Presets store underlying parameter values only; macro positions are derived from underlying params on load.

**Architecture:** Each engine exposes three macro knobs in the Primary Controls zone. Macro knob movement calls a macro_apply() function that computes delta values for each underlying param and dispatches them as if the user moved each param individually.

**Macro definitions are specified in Musicality Spec Section 9.** Backend does not implement macro logic — this is handled by the Frontend agent.

---

## 7. Effects Section

### 7.1 Hermite Polynomial Waveshaper

The waveshaper uses Hermite polynomials H_n(x) as a basis for defining transfer curves. Being orthogonal, they allow precise control over which harmonic order is boosted or attenuated. Apple Accelerate's `vDSP_vpoly` evaluates polynomials over sample arrays in hardware-accelerated vectorised form.

```cpp
// Hermite basis waveshaper
// Coefficients c[n] directly control harmonic n+1 level
// H_0(x)=1, H_1(x)=x, H_2(x)=x²-1, H_3(x)=x³-3x, etc.
float hermite_waveshape(float x, const float *coeff, int order) {
    float result  = 0.0f;
    float h_prev2 = 1.0f;  // H_0
    float h_prev1 = x;     // H_1
    result += coeff[0] * h_prev2 + coeff[1] * h_prev1;
    for (int n = 2; n <= order; ++n) {
        float h_n = x * h_prev1 - (float)(n-1) * h_prev2;
        result   += coeff[n] * h_n;
        h_prev2   = h_prev1;
        h_prev1   = h_n;
    }
    return result;
}
// For block processing — use vDSP_vpoly
// The Accelerate polynomial evaluator maps directly onto Hermite recurrence
```

### 7.2 Hadamard FDN Reverb

The Feedback Delay Network uses a 16×16 Hadamard matrix as the feedback matrix. Hadamard matrices are orthogonal (H·H^T = N·I), ensuring the feedback network is lossless and distributes signal across all delay lines equally.

> **AGENT CONSTRAINT — DELAY LINE LENGTHS:** Use prime number delay lengths to prevent modal clustering. Suggested values (in samples): 1009, 1021, 1031, 1049, 1061, 1087, 1091, 1093, 1097, 1103, 1109, 1117, 1123, 1129, 1151, 1153.

```cpp
// 16x16 Hadamard matrix — generated recursively from H_1 = [[1,1],[1,-1]]
// Normalised version: H_16 / 4 — keeps signal level stable through feedback

float fdn_process(float input, float *delay_lines,
                  const int *delay_lengths, const float *gains,
                  const float hadamard[16][16], int N = 16) {
    float state[16];
    for (int i = 0; i < N; ++i)
        state[i] = delay_lines[i * delay_lengths[i]];

    float new_state[16];
    // Matrix multiply via Accelerate BLAS
    cblas_sgemv(CblasRowMajor, CblasNoTrans, N, N,
                1.0f / N, &hadamard[0][0], N, state, 1, 0.0f, new_state, 1);

    float output = 0.0f;
    for (int i = 0; i < N; ++i) {
        float in_i = (i == 0) ? input + new_state[i] * gains[i]
                               : new_state[i] * gains[i];
        push_delay_line(&delay_lines[i * delay_lengths[i]], delay_lengths[i], in_i);
        output += state[i];
    }
    return output / N;
}
```

---

## 8. Development Phases

**Toolchain:** Xcode / clang++ · CLAP SDK (`free-audio/clap`) · Apple Accelerate (zero-install, ships with macOS) · JUCE optional for GUI

### Phase 1 — STOCH Engine + CLAP Wrapper

**Target:** 2–3 months  
**Deliverable:** Functional, distributable CLAP plugin — independently marketable milestone

| Week | Task | Acceptance Check |
|---|---|---|
| W1–2 | CLAP boilerplate — `clap_plugin_t`, param registry, `process()` producing silence | Plugin loads in Bitwig/Reaper; no crash on open/close/reopen |
| W3–4 | GENDYN core — single voice, Gaussian distribution | Audible signal; no DC runaway; stable output through 60s |
| W5–6 | All four distributions + elastic barrier reflection | Param switch works without crash; Cauchy audibly more aggressive than Gaussian |
| W7–8 | Polyphony via CLAP thread pool; voice stealing | 16 voices; no internal thread spawn; thread check clean |
| W9–10 | Pitch quantization + Scala `.scl` loader | 31-TET and Bohlen-Pierce parse correctly; snap output within 0.5 cents |
| W11–12 | MPE routing + minimal GUI + CA LFO (Rules 30, 90, 110) | MPE offset returns to base value after note release; UI maps correct param IDs |

**Key dependencies:**
- `free-audio/clap` — CLAP SDK, header-only C API
- `free-audio/clap-helpers` — C++ wrapper reducing boilerplate
- Luque, S. (2009) — *Stochastic Synthesis: Origins and Extensions* — primary GENDYN reference
- Xenos source (GPL, GitHub: arme-uob/xenos) — reference implementation for A/B validation

> **NOTE:** Phase 1 alone produces a marketable, unique product. STOCH + MPE + CA LFO is already more flexible than Xenos and is CLAP-native. Consider an early release.

---

### Phase 2 — DIFFU Engine

**Target:** 2–3 months  
**Deliverable:** R-D spectral synthesis integrated into plugin

**Mandatory implementation sequence — do not skip steps:**

1. Standalone Gray-Scott C++ test harness — no CLAP, no audio
2. Visual/numerical validation of all six Pearson regions
3. Offline V-grid → partial amplitudes mapping
4. Additive bank single voice render (offline/standalone)
5. Integration into plugin voice path
6. vDSP optimisation — only after correctness baseline
7. GUI: F/k XY pad + Pearson labels + live grid visualisation

| Week | Task | Acceptance Check |
|---|---|---|
| W1–2 | Standalone Gray-Scott harness | CSV dump shows correct spatial patterns at all 6 Pearson coordinates |
| W3–4 | Audio rate integration — control rate grid, 128-partial bank (expandable from initial 64 during W5-6 vDSP optimisation) | Audible evolving harmonic output |
| W5–6 | Accelerate vectorisation | 128-cell grid update < 0.05ms on M-series; measured via benchmark |
| W7–8 | Logarithmic spectral mapping + SPECTRAL_CURVE param | Linear vs. log spacing audibly different; musical intervals confirmed |
| W9–10 | CLAP integration — per-voice grid, thread pool dispatch | 16-voice polyphony; each voice has independent grid evolution |
| W11–12 | GUI — F/k XY pad, Pearson labels, spectral display, engine blend | Pearson region labels visible and snap-to-region works |

---

### Phase 3 — BUBBLE Engine + Effects + Final Polish

**Target:** 1–2 months  
**Deliverable:** Complete ENTROP v1.0

**Mandatory implementation sequence:**

1. Single bubble Minnaert oscillator — verify 1mm water = 3.26 kHz
2. Population spawn/death model + fluid presets — no coupling
3. Cloud coupling behind `ENTROP_BUBBLE_COUPLING` feature flag
4. Hermite waveshaper — dry path + bypass A/B
5. Hadamard FDN reverb — standalone tested module
6. Engine blend + perceptual loudness normalisation
7. Preset bank (minimum 45 presets) + final QA sweep

| Week | Task | Acceptance Check |
|---|---|---|
| W1–2 | Single Minnaert oscillator | 1mm water bubble ≈ 3.26 kHz; Young-Laplace correction changes pitch |
| W3–4 | Population system + fluid presets | Fluid presets produce audibly distinct character; no CPU explosion |
| W5 | Cloud coupling | Sub-bass emergence verified at high coupling + high population density |
| W6 | Hermite waveshaper | Per-harmonic coefficients audibly distinct; vDSP_vpoly integrated |
| W7 | Hadamard FDN reverb | Smooth decay; no resonance clustering; prime-length delay lines |
| W8 | Engine blend + preset bank + final QA | All 3 engines blend without instability; 45 presets ship |

**v1.0 minimum done criteria:**
- All 3 engines functional and blendable without instability
- CLAP session save/load reliable
- MPE routing working for key parameters per engine
- CPU usage within budget on target M-series hardware
- - 45 presets covering distinct engine characters (see Musicality Spec Section 8.5 for category breakdown)
- No crash-level bugs in primary target hosts
- Preset load crossfade: 10ms crossfade from current state to recalled state when audio is active (no clicks on preset change)

---

## 9. Complete Parameter Map

> **FOR ALL AGENTS:** This is the authoritative parameter contract. Parameter IDs are fixed. Do not reassign IDs. Do not add parameters without an explicit orchestrator task that updates this table. Frontend bindings, backend enum values, and preset file serialisation must all reference these IDs.

| ID | Engine | Name | Range | Taper | MPE |
|---|---|---|---|---|---|
| 0 | STOCH | Segments | 3–12 (int) | Linear + integer snap | — | (init default: 6)
| 1 | STOCH | Distribution | 0–3 (enum) | Stepped | Y-axis blend |
| 2 | STOCH | Amp Scatter | 0.0–1.0 | Exponential (x²) | Pressure |
| 3 | STOCH | Freq Scatter | 0.0–1.0 | Exponential (x²) | Slide |
| 4 | STOCH | Barrier Elasticity | 0.0–1.0 | Linear | — |
| 5 | STOCH | Pitch Snap | 0.0–1.0 | Linear | — |
| 6 | STOCH | Scale Root | MIDI 0–127 | Quantized (12-note) | — |
| 7 | DIFFU | Feed Rate F | 0.01–0.08 | Linear | Pressure |
| 8 | DIFFU | Kill Rate k | 0.04–0.07 | Linear | Slide |
| 9 | DIFFU | Diffusion Du | 0.10–0.25 | Logarithmic | Y-axis |
| 10 | DIFFU | Diffusion Dv | 0.04–0.12 | Logarithmic | — |
| 11 | DIFFU | Spectral Map Curve | 0.0–1.0 | Sigmoid | — |
| 12 | DIFFU | Grid Reset Trigger | Bang | — | — |
| 13 | BUBBLE | Fluid Density | 500–14000 kg/m³ | Logarithmic | — |
| 14 | BUBBLE | Surface Tension | 0.01–0.50 N/m | Logarithmic | Slide |
| 15 | BUBBLE | Pressure | 50000–200000 Pa | Linear | — |
| 16 | BUBBLE | Mean Radius | 0.1mm–40mm | Logarithmic | Slide |
| 17 | BUBBLE | Radius Spread | 0.1–2.0 | Linear | — |
| 18 | BUBBLE | Spawn Rate | 1–500 Hz | Logarithmic | Pressure |
| 19 | BUBBLE | Coupling Coefficient | 0.0–1.0 | Exponential (x²) | Y-axis |
| 20 | GLOBAL | Engine Blend S↔D | 0.0–1.0 | Linear | — |
| 21 | GLOBAL | Engine Blend D↔B | 0.0–1.0 | Linear | — |
| 22 | GLOBAL | Master Gain | 0.0–2.0 | Linear | — |
| 23 | LFO | CA Rule | 0–255 | Stepped | — |
| 24 | LFO | CA Rate | 0.01–20 Hz | Logarithmic | — |
| 25 | LFO | LFO → Target | Enum | Stepped | — |
| 26 | LFO | LFO Depth | 0.0–1.0 | Linear | — |
| 27 | FX | Waveshaper H2 | -1.0–1.0 | Linear | — |
| 28 | FX | Waveshaper H3 | -1.0–1.0 | Linear | — |
| 29 | FX | Waveshaper H4–H6 | -1.0–1.0 each | Linear | — |
| 30 | FX | Reverb Size | 0.0–1.0 | Linear | — |
| 31 | FX | Reverb Mix | 0.0–1.0 | Linear | — |
| 32 | DIFFU | Evolution Rate | 0.1×–10× | Logarithmic | — |

---

## 10. Toolchain & Dependencies

> **AGENT CONSTRAINT — DEPENDENCY MINIMALISM:** Do not add any runtime dependency that is not either header-only or ships with macOS. Every external library is a distribution problem. CLAP SDK + Accelerate + clang++ is sufficient for the entire audio engine.

| Component | Source | License | Notes |
|---|---|---|---|
| CLAP SDK | `github.com/free-audio/clap` | MIT | Header-only C API — the core plugin standard |
| clap-helpers | `github.com/free-audio/clap-helpers` | MIT | C++ convenience wrappers for plugin and host |
| Apple Accelerate | Ships with macOS / Xcode | Apple (free) | vDSP, BLAS, BNNS — vectorised math on Apple Silicon |
| Xcode / clang++ | Apple Developer | Free | C++20, auto-vectorisation, Address Sanitizer |
| reaction-diffusion-cpp | `github.com/pavel-perina/reaction-diffusion-cpp` | MIT | Reference Gray-Scott — study and adapt, do not ship directly |
| JUCE (optional) | `juce.com` | GPL / commercial | Only needed for GUI — raw CLAP GUI extension is an alternative |

---

## Appendix — Key References

**GENDYN / Stochastic Synthesis**  
Luque, S. (2009). *Stochastic Synthesis: Origins and Extensions.* Institute of Sonology, Royal Conservatory, The Hague.

**Gray-Scott Reaction-Diffusion**  
Pearson, J.E. (1993). Complex Patterns in a Simple System. *Science*, 261(5118), 189–192.

**Bubble Resonance**  
Minnaert, M. (1933). On musical air-bubbles and sounds of running water. *The London, Edinburgh, and Dublin Philosophical Magazine*, 16(104), 235–248.

**Harmonic Fluids (bubble cloud physics)**  
Zheng, C. & James, D.L. (2009). Harmonic Fluids. *ACM SIGGRAPH 2009 Papers.*

**CLAP Format**  
`github.com/free-audio/clap` — CLAP 1.x specification, extensions, and headers.

**Hadamard FDN Reverb**  
Schlecht, S.J. & Habets, E.A.P. (2017). On Lossless Feedback Delay Networks. *IEEE Transactions on Signal Processing.*

**ECA Audio / Xenakis**  
Serquera, J. & Miranda, E.R. (2010). Sound Synthesis with Cellular Automata. University of Plymouth.

**Apple Accelerate / vDSP**  
`developer.apple.com/documentation/accelerate` — vDSP, BLAS, and BNNS API reference.

---

*ENTROP · Developer Implementation Plan v0.1 · Authoritative technical specification*
