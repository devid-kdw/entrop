# Backend Agent Brief — ENTROP

You implement C++20/CLAP audio engine code. Current phase: Phase 1. Current milestone: W1-2.

## Your scope
DSP formulas, CLAP extensions, audio callback, parameter model, performance optimisation.
Implement formulas exactly as written in the Developer Plan — do not improve or substitute algorithms.

## Hard constraints (non-negotiable — see rt_safety_protocol.md)
- No malloc/new/delete in process() or any function it calls
- No std::thread — clap_host_thread_pool only
- No mutex in audio path
- All param changes interpolated over ≥64 samples
- MPE offset is non-destructive — never overwrites base value
- Apple Accelerate used exactly where the Developer Plan specifies — not elsewhere

## You do not touch
src/gui/ — any file. UI layout. Visual representation. Parameter naming.

## Required in every handoff
1. Summary | 2. Files changed | 3. Assumptions | 4. Acceptance checks |
5. Known risks | 6. RT Safety Checklist (all 7 items) | 7. Next tasks (max 3)

## Injected context for this session
[rt_safety_protocol.md + relevant Developer Plan section + parameter_contract.json]

## Dev Testing Tools

Za testiranje plugina bez DAW-a, koristiš:
- `clap-info ./entrop.clap` — validacija parametara i plugin strukture
- `clap-host -p ./entrop.clap` — audio playback kroz plugin, MIDI triggering, bez GUI DAW-a
- `sonic_test/` binaries — direktno DSP testiranje bez CLAP layer-a

Puni Xcode IDE nije potreban. Command Line Tools su dovoljni za cijeli build.
Svaki backend task koji dodaje novu runtime dependenciju mora:
1. Provjeriti je li ona već u `knowledge/techstack.md`
2. Ako nije — eskalirati prije implementacije, ne dodavati samostalno
