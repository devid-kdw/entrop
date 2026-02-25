# ENTROP

**CLAP-native synthesizer plugin** — three physics-based synthesis engines on macOS / Apple Silicon.

> *Stochastic waveforms. Reaction-diffusion spectra. Fluid-acoustic resonance.*
> *No wavetables. No FM. No analog emulation. Every sound is emergent.*

---

## Engines

| Engine | Method | Character |
|---|---|---|
| **STOCH** | Xenakis GENDYN stochastic synthesis | Chaos, tension, probability-driven waveforms |
| **DIFFU** | Gray-Scott reaction-diffusion → 128-partial additive bank | Growth, mutation, living spectral evolution |
| **BUBBLE** | Minnaert bubble resonance + population model | Depth, mass, fluid-acoustic pressure |

## Architecture

- **Plugin format:** CLAP (native) — per-voice MPE modulation, host thread pool, project consolidation
- **Platform:** macOS · Apple Silicon (M-series)
- **Toolchain:** C++20 · clang++ · Apple Accelerate · CLAP SDK
- **GUI:** JUCE (optional) with CLAP GUI extension, Retina rendering

## Key Features

- Three blendable synthesis engines with Tri-Morph control
- Per-voice non-destructive MPE modulation (pressure/slide/Y-axis)
- Cellular Automata LFO (ECA Rules 30/90/110/150)
- Hermite polynomial waveshaper + Hadamard FDN reverb
- Xenharmonic pitch quantization with Scala .scl support
- 45 presets across 6 categories

## Project Structure

```
entrop-clap/
├── knowledge/          # Spec documents & knowledge base (read-only)
├── docs/               # Tracker, agent briefs, ADRs, handoffs
├── src/
│   ├── core/           # Parameter model, voice base, RT utilities
│   ├── clap/           # CLAP entry, plugin instance, param handler
│   ├── engines/        # stoch/ diffu/ bubble/
│   ├── modulation/     # ECA LFO, mod router
│   ├── fx/             # Hermite waveshaper, Hadamard FDN
│   └── gui/            # Editor, engine panels, visualizers
├── tests/              # Unit, integration, offline audio, benchmarks
└── presets/            # Init, archetypes, composition-ready, blend, edge-case
```

## Building

### Prerequisites

- macOS 13+ (Ventura or later)
- Xcode 15+ with Command Line Tools
- CMake 3.24+
- Git

### Dependencies (header-only or ships with macOS)

| Dependency | Source | Notes |
|---|---|---|
| CLAP SDK | `github.com/free-audio/clap` | MIT · header-only C API |
| clap-helpers | `github.com/free-audio/clap-helpers` | MIT · C++ convenience wrappers |
| Apple Accelerate | Ships with macOS | vDSP, BLAS — zero install |
| JUCE (optional) | `juce.com` | GPL/commercial — GUI only |

### Build Steps

```bash
git clone --recursive https://github.com/your-org/entrop-clap.git
cd entrop-clap
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release
```

The `.clap` bundle will be output to `build/` — copy to `~/Library/Audio/Plug-Ins/CLAP/`.

## Development Phases

| Phase | Content | Target |
|---|---|---|
| **Phase 1** | STOCH engine + CLAP wrapper + MPE + CA LFO | 2–3 months |
| **Phase 2** | DIFFU engine (Gray-Scott R-D + additive bank) | 2–3 months |
| **Phase 3** | BUBBLE engine + FX + blend + presets + polish | 1–2 months |

## Documentation

- `knowledge/ENTROP_Developer_Plan_v01.md` — DSP formulas, CLAP architecture, parameter IDs
- `knowledge/ENTROP_Musicality_Spec_v02.md` — musical acceptance criteria, failure modes
- `knowledge/ENTROP_Design_Brief_v01.md` — visual language, color system, UI layout
- `docs/implementation_tracker.md` — current phase, task log, risk registry

## License

*TBD*
