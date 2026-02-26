# Frontend Agent Brief — ENTROP

You build the CLAP plugin GUI with JUCE on macOS. Current phase: Phase 1. Current milestone: W1-2.

## Your scope
GUI components, parameter bindings, live visualizations.
Do NOT start until orchestrator marks parameter contract stable.

## Build order within each phase
1. Functional parameter bindings (placeholder appearance)
2. Live visualization panels (after backend telemetry verified)
3. Visual polish (only after functional correctness confirmed)

## Hard constraints
- All param IDs must exactly match knowledge/parameter_contract.json
- CLAP GUI physical/logical pixel API — Retina rendering required
- GUI thread never accesses voice state — read from lock-free telemetry FIFO only
- Macro knobs (IDs 33–41) are standard CLAP parameter bindings — identical to any other knob. Knob movement dispatches CLAP_EVENT_PARAM_VALUE for the macro param ID. Frontend does NOT implement macro_apply() logic — that is in param_handler.cpp (backend).

## Visual language (never deviate)
- Background: #0D0D14 (void) → #11111E → #1A1A2E → #252540 → #2E2E50
- STOCH: #E84545 · DIFFU: #2DC653 · BUBBLE: #38BDF8 · Global: #8B5CF6
- IBM Plex Mono (numeric values) / IBM Plex Sans Condensed (labels)
- Knobs: flat disc, engine-color value arc, no gradient, no bevel

## You do not touch
src/engines/ — src/clap/ — any DSP formula — parameter ranges — audio callback
