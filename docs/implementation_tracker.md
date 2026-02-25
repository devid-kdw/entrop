# ENTROP Implementation Tracker

**Last updated:** 2026-02-25
**Current phase:** Phase 1
**Current milestone:** W1-2
**Build status:** Not started — scaffolding complete
**Last merged task:** None
**Next task:** B-101 — CLAP entry + factory

---

## Phase 1 Acceptance Gate
- [ ] STOCH init TTUS ≤60s
- [ ] 5 archetypes Robustness Test pass
- [ ] Soft-limiter verified
- [ ] RT safety clean (ASan + TSan)
- [ ] Preset recall deterministic
- [ ] 16-voice polyphony verified
- [ ] MAP passed

## Phase 1 Task Log

| Task ID | Agent | Status | Handoff |
|---|---|---|---|
| B-101 | Backend | ⬜ | — |
| B-102 | Backend | ⬜ | — |
| B-103 | Backend | ⬜ | — |
| T-101 | Test | ⬜ | — |
| B-201 | Backend | ⬜ | — |
| B-202 | Backend | ⬜ | — |
| T-201 | Test | ⬜ | — |
| B-301 | Backend | ⬜ | — |
| B-302 | Backend | ⬜ | — |
| T-301 | Test | ⬜ | — |
| B-303 | Backend | ⬜ | — |
| B-401 | Backend | ⬜ | — |
| B-402 | Backend | ⬜ | — |
| T-401 | Test | ⬜ | — |
| B-501 | Backend | ⬜ | — |
| B-502 | Backend | ⬜ | — |
| T-501 | Test | ⬜ | — |
| B-601 | Backend | ⬜ | — |
| B-602 | Backend | ⬜ | — |
| F-601 | Frontend | ⬜ | Blocked: B-101–B-502 |
| F-602 | Frontend | ⬜ | Blocked: F-601 |

## Phase 2 — DIFFU
[Not started — Phase 1 gate must pass first]

## Phase 3 — BUBBLE + FX + Polish
[Not started — Phase 2 gate must pass first]

---

## Open Risks

| ID | Risk | Severity | Status |
|---|---|---|---|
| R-01 | GS instability at boundary F/k | High | Open |
| R-02 | Thread pool unsupported in hosts | Medium | Open |
| R-03 | Cauchy extreme values | High | Open |
| R-04 | DIFFU partial clicks | High | Open |
| R-05 | BUBBLE population overflow | High | Open |
| R-06 | Loudness normalization pumping | Medium | Open |
| R-07 | Blend discontinuity | Medium | Open |
| R-08 | PRNG non-determinism | High | Open |
| R-09 | vDSP macOS version diff | Low | Open |
| R-10 | Engine character too similar to competitors | High | Open |

---

## Parameter Contract
Version: 1.1 — last modified 2026-02-25
File: knowledge/parameter_contract.json
Changes: [none]
