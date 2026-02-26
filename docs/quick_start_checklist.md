# ENTROP — Quick-Start Checklist

Redoslijed koraka za podizanje razvojnog okruženja od nule.

---

## Day 0 — Scaffolding (before any code)

- [x] Kreirati `entrop-clap/` folder strukturu (Section A, Orchestrator Resources)
- [x] Kopirati 3 spec dokumenta u `knowledge/`
- [x] Kreirati knowledge base fajlove (parameter_contract.json, rt_safety_protocol.md, itd.)
- [x] Kreirati `docs/implementation_tracker.md`
- [x] Kreirati 4 agent briefsa
- [x] Kreirati README.md, .gitignore, DEPENDENCIES.md

## Day 1 — Dev Toolchain Setup (before any code)

- [x] `xcode-select --install` — Command Line Tools (puni Xcode IDE nije potreban)
- [x] `sudo xcodebuild -license accept` — prihvati Xcode licencu
- [x] `brew install cmake ninja` — build system
- [x] `brew install portaudio` — za sonic_test/ audio playback
- [x] `brew install qt6 rtaudio rtmidi pkgconfig` — za clap-host build
- [x] Build `clap-info` iz `external/clap-info/` — plugin validation CLI
- [x] Build `clap-host` iz `external/clap-host/` — dev CLAP host
- [x] Verifikacija: `clap-info --help` i `clap-host --help` rade bez greške
- [x] Kreirati `knowledge/techstack.md` s potvrđenim verzijama

## Day 1 — Git Init

- [x] `git init` u `entrop-clap/`
- [x] `git add . && git commit -m "Day 0: project scaffolding"`
- [x] Dodati CLAP SDK kao submodule: `git submodule add https://github.com/free-audio/clap.git external/clap`
- [x] Dodati clap-helpers: `git submodule add https://github.com/free-audio/clap-helpers.git external/clap-helpers`
- [x] Push na GitHub

## Day 2+ — Phase 1 W1-2: CLAP Boilerplate

- [ ] B-101: `src/clap/plugin_entry.cpp` — CLAP entry + factory
- [ ] B-102: `process()` writing silence to output buffers
- [ ] B-103: `src/core/parameter_model.h` — 32 param IDs as enum
- [ ] T-101: Load/unload stress test checklist

## Day 5+ — First Verification

- [ ] Pokrenuti `clap-info` na prvom `.clap` buildu — provjeri da host vidi sve parametre
- [ ] Pokrenuti `clap-host -p ./entrop.clap` — provjeri da plugin producira tišinu bez crasha
