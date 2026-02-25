# ENTROP Techstack

Sve komponente su required osim gdje je označeno OPTIONAL.
Ovo je jedini izvor istine za toolchain. Agents ne dodaju dependencije bez update ovog dokumenta.

## Compiler & Build

| Component | Install | Version (verified) | Notes |
|---|---|---|---|
| clang++ | Xcode Command Line Tools | Apple clang 17.0.0 (arm64) | `xcode-select --install` — puni Xcode IDE nije potreban |
| CMake | `brew install cmake` | 4.2.3 | Build system |
| Ninja | `brew install ninja` | 1.13.2 | Faster builds than Make |
| Git | Ships with CLT | 2.50.1 | Version control |

## CLAP Ecosystem

| Component | Source | Install | Version (verified) | Notes |
|---|---|---|---|---|
| CLAP SDK | github.com/free-audio/clap | git submodule | 1.2.7 | Header-only, MIT |
| clap-helpers | github.com/free-audio/clap-helpers | git submodule | latest | C++ wrappers, MIT |
| clap-host | github.com/free-audio/clap-host | build from source | latest | Dev testing host — NO DAW needed |
| clap-info | github.com/free-audio/clap-info | build from source | latest | Plugin validation CLI |

### clap-info Build Procedure
```bash
cd external/clap-info
mkdir build && cd build
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release
ninja
# Binary: external/clap-info/build/clap-info
```

### clap-host Build Procedure
```bash
brew install qt6 rtaudio rtmidi pkgconfig ninja
cd external/clap-host
git submodule update --init --recursive
mkdir build && cd build
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_PREFIX_PATH="$(brew --prefix qt6)" \
  -DCMAKE_CXX_FLAGS="-I$(brew --prefix rtmidi)/include -I$(brew --prefix rtaudio)/include"
ninja
# Binary: external/clap-host/build/host/clap-host
```

## Apple Frameworks (zero-install, ship with macOS)

| Component | Notes |
|---|---|
| Apple Accelerate | vDSP, BLAS — ships with macOS SDK |
| CoreAudio | Audio I/O |
| CoreMIDI | MIDI I/O |

## Audio I/O (sonic_test/ only)

| Component | Install | Version (verified) | Notes |
|---|---|---|---|
| PortAudio | `brew install portaudio` | 19.7.0 | Only for sonic_test/ binaries — NOT linked into plugin |

## GUI (OPTIONAL — Phase 1 can ship without)

| Component | Source | Notes |
|---|---|---|
| JUCE | juce.com | GPL / commercial — GUI only |

## clap-host Dependencies (build-time only)

| Component | Install | Version (verified) | Notes |
|---|---|---|---|
| Qt6 | `brew install qt6` | 6.10.2 | Required by clap-host GUI |
| RtAudio | `brew install rtaudio` | 6.0.1 | Audio I/O for clap-host |
| RtMidi | `brew install rtmidi` | 6.0.0 | MIDI I/O for clap-host |
| pkgconfig | `brew install pkgconfig` | 2.5.1 | Dependency resolution |

## Reference / Study (DO NOT SHIP)

| Component | Source | Notes |
|---|---|---|
| reaction-diffusion-cpp | github.com/pavel-perina/reaction-diffusion-cpp | Study Gray-Scott only |

## Dev Testing Workflow

| Stage | Tool | Command |
|---|---|---|
| DSP math correctness | sonic_test/ | `./sonic_player_stoch --params ...` |
| Plugin load / param list | clap-info | `./external/clap-info/build/clap-info ./entrop.clap` |
| Audio through plugin | clap-host | `./external/clap-host/build/host/clap-host -p ./entrop.clap` |
| Full GUI + preset test | DAW (Bitwig/Reaper) | Once per phase MAP session only |

## System Info (verified 2026-02-25)

| Property | Value |
|---|---|
| macOS | 26.3 |
| Chip | Apple M4 Pro (arm64) |
| Homebrew | 5.0.14 |

## NOT Required

- Xcode IDE (full) — Command Line Tools are sufficient
- Any DAW for daily development — use clap-host
- iOS Simulator or any non-macOS target
