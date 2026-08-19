# Lab 1 — Software Failure Forensics & Engineering Ethics
## Case Study: T8 — CrowdStrike 2024 Outage

This repository contains our team's forensic analysis of the CrowdStrike
Channel File 291 outage of 19 July 2024, produced for Lab 1 of the
Advanced Software Engineering course.

## Contents

| Deliverable | File | Milestone |
|---|---|---|
| D1 — Failure Timeline | [`timeline/D1-failure-timeline.md`](timeline/D1-failure-timeline.md) | M1 |
| D2 — Root-Cause Map | [`root-cause/D2-root-cause-map.md`](root-cause/D2-root-cause-map.md) | M2 |
| D3 — SDLC Failure Map + Origin–Detection Matrix | [`sdlc-analysis/D3-sdlc-failure-map.md`](sdlc-analysis/D3-sdlc-failure-map.md) | M3 |
| D4 — Engineering Responsibility & Ethics Map | [`ethics/D4-engineering-ethics-map.md`](ethics/D4-engineering-ethics-map.md) | M4 |
| D5 — Failure Prevention Plan | [`prevention/D5-prevention-plan.md`](prevention/D5-prevention-plan.md) | M5 |
| D6 — Oral Defence Script | [`oral-defence-script.md`](oral-defence-script.md) | M6 |
| Team Charter | [`team-charter.md`](team-charter.md) | — |

The full written submission (4–5 pages) compiling all of the above is
`lab01-failure-forensics.pdf` at the repository root.

## Case Summary
On 19 July 2024, CrowdStrike released a Rapid Response Content update
(Channel File 291) containing a parameter-count mismatch that its own
Content Validator failed to catch. The defect triggered an out-of-bounds
memory read in the Falcon sensor's kernel-mode driver, crashing Windows on
an estimated 8.5 million machines worldwide within minutes — the largest
IT outage of its kind to date — because the update was pushed to the
entire global fleet simultaneously with no staged/canary rollout.

## Sources
- CrowdStrike, *Channel File 291 Incident — External Technical Root Cause Analysis*, 6 Aug 2024
- CrowdStrike, *Preliminary Post Incident Review*, 24 Jul 2024
- CISA, *Widespread IT Outage Due to CrowdStrike Update*, Jul 2024
