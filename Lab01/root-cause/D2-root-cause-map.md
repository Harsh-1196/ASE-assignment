# D2 — Root-Cause Map
## T8: CrowdStrike Channel File 291 Outage

### 5-Why Analysis

| # | Why | Answer |
|---|-----|--------|
| 1 | Why did millions of Windows machines crash (BSOD)? | The Falcon sensor's kernel driver performed an out-of-bounds memory read while processing Channel File 291. |
| 2 | Why did it read out of bounds? | The IPC Template Type defined 21 input fields, but the sensor's integration code only ever supplied 20 values — a parameter-count mismatch — and the Content Interpreter had no runtime bounds check to catch this. |
| 3 | Why wasn't the mismatch caught before release? | The Content Validator, whose job is to reject malformed content, itself contained a logic error and passed the file as valid. |
| 4 | Why did testing never expose this? | Every prior Template Instance used wildcard matching on the 21st field, so no test case (unit, stress, or field deployment) ever exercised a real, non-wildcard value in that field until 19 July 2024. |
| 5 | Why was an untested code/data path pushed to every production machine at once? | Rapid Response Content was released straight to the entire global fleet with no canary/staged rollout and no customer-controlled deployment ring — unlike full sensor version updates, which do use phased release. |

**Root cause:** An organizational/process decision to treat content
(configuration/rule) updates as low-risk — and therefore exempt from the
staged rollout, canary testing, and customer opt-in controls applied to
sensor code — combined with a kernel-mode architecture that had no
fail-safe when it received malformed content.

### Immediate, Contributing, and Root Causes

**Immediate cause:** Out-of-bounds read in the Content Interpreter when
evaluating Channel File 291's 21st input field, crashing the kernel driver.

**Contributing causes:**
- Parameter-count mismatch (21 defined vs. 20 supplied) never reconciled between the Template Type and the integration code.
- Content Validator logic error failed to flag the malformed Template Instance.
- Test suite's reliance on wildcard values meant the non-wildcard equivalence class was never tested.
- No runtime bounds/sanity checking in the Content Interpreter as a last line of defense.
- No staged/canary deployment for Rapid Response Content.
- No customer-side control to delay or ring-fence content updates on critical production systems.

**Root causes:**
- Absence of defense-in-depth: a single, unvalidated content file could crash a kernel-mode driver with no containment.
- Release-process governance did not require the same safety gates for "content" that it required for "code," despite content having code-level blast radius.

### Failure-Cause Classification (C1–C10)

| Cause | Category |
|---|---|
| Parameter-count mismatch introduced in integration code | **C4** Implementation Failure |
| No bounds check in Content Interpreter; single point of failure across whole fleet | **C3** Design/Architecture Failure |
| Content Validator logic error | **C4** Implementation Failure (validator itself is software) |
| Wildcard-only test data meant real matching logic was never tested | **C5** Testing/Verification Failure |
| Rapid Response Content pushed globally with no canary ring, no rollback safeguard | **C6 / C7** Configuration & Deployment/Operational Failure |
| Content treated as lower-risk than code; speed of threat response prioritized over release safety gates | **C9** Organizational/Management Failure |
| No customer opt-in/delay control communicated or offered for critical systems | **C10** Ethics/Professional Responsibility (duty of transparency and safe defaults) |

### Evidence / Source
CrowdStrike, *Channel File 291 Incident — External Technical Root Cause
Analysis* (6 Aug 2024); CrowdStrike Preliminary Post Incident Review (24 Jul
2024); CISA Alert, "Widespread IT Outage Due to CrowdStrike Update" (Jul
2024).
