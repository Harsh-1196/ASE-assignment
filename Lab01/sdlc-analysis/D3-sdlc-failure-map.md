# D3 — SDLC Failure Map & Origin–Detection Matrix
## T8: CrowdStrike Channel File 291 Outage

### Origin–Detection Matrix

| Cause | Origin Phase | Detection Phase | Gap (cost-of-late-detection) |
|---|---|---|---|
| IPC Template Type defines 21 fields but integration code supplies only 20 | **Implementation** (Feb–Mar 2024, sensor 7.11) | **Operation** (production, 19 Jul 2024) | ~5 months and a full production incident later; could have been a single unit test |
| No bounds/array-length check in Content Interpreter | **Design/Architecture** | **Operation** | Never caught pre-release; only found during post-incident RCA |
| Content Validator logic error (fails to reject the malformed instance) | **Implementation** (of the validator tool itself) | **Operation** | Validator is the last automated gate before release — its failure meant nothing downstream could catch the defect |
| Test suite only ever exercised wildcard matching for field 21 | **Testing** | **Operation** (and formally, post-incident RCA) | An entire equivalence class of inputs was untested for ~4 months across multiple releases |
| No staged/canary rollout for Rapid Response Content | **Release/Deployment process design** (policy predates this incident) | **Operation**, discovered only after global impact | Defect reached 100% of the fleet within minutes instead of a small canary ring |
| No customer-controlled update rings for content | **Requirements/Release policy** (a scope decision made early in the product's life) | **Operation** | Customers had zero ability to delay or pilot-test the update on non-critical systems first |

### SDLC Phase Mapping

- **Requirements:** The decision to give customers no control over Rapid Response Content timing was effectively a requirements gap — availability/safety-of-update-cadence was never treated as a first-class requirement for a kernel-mode security product running on critical infrastructure.
- **Design:** The Content Interpreter was designed without defensive bounds checking, and the overall content-delivery architecture had no blast-radius containment (one bad file could reach every machine at once).
- **Implementation:** The mismatch between the 21-field Template Type and the 20-value integration code, and the bug in the Content Validator, were both coding defects.
- **Testing:** Equivalence-class coverage was incomplete — every historical test case used a wildcard for field 21, so the non-wildcard path was structurally untested.
- **Release/Deployment:** Content updates bypassed the phased/canary rollout used for sensor binaries, so the untested path was deployed to the entire global fleet simultaneously.
- **Operation:** This is where every defect above was ultimately detected — via mass production failure — rather than at any earlier, cheaper stage.

### Cost-of-Late-Detection Argument
Every root and contributing cause in this incident **originated** between
the Implementation and Release-policy phases (months earlier, in some
cases), but every one of them was **detected** only in full Operation, at
global scale. Under the classic cost-of-change curve, a defect caught in a
unit test costs a rerun; the same defect caught in production here cost an
estimated 8.5 million crashed machines, multi-day recovery efforts, and
billions of dollars in business interruption. A single missing test case
(non-wildcard matching for field 21) and a single missing deployment control
(canary rollout) would each independently have shifted detection from
Operation back to Testing or early Release — at a small fraction of the
actual cost.
