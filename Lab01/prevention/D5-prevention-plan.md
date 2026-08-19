# D5 — Failure Prevention Plan
## T8: CrowdStrike Channel File 291 Outage

Five concrete engineering controls, spanning Design, Testing,
Release/Deployment, Configuration Management, and Operations.

### 1. Runtime Bounds Checking in the Content Interpreter
- **SDLC stage:** Design / Implementation
- **Type:** Preventive
- **Failure addressed:** Out-of-bounds memory read (C3/C4) — the immediate technical cause of the crash.
- **Description:** Require the Content Interpreter to validate the number of supplied input values against the Template Type's declared field count before use, and to fail closed (skip the rule) rather than read past the array bound. This makes the crash structurally impossible even if every upstream check fails.

### 2. Equivalence-Class Test Coverage for Every Template Type
- **SDLC stage:** Testing
- **Type:** Preventive
- **Failure addressed:** Untested non-wildcard matching path (C5).
- **Description:** Mandate that every input field of a new Template Type be tested with at least one wildcard *and* one non-wildcard (specific-value) test case before the Type is approved for use in production content, closing the exact coverage gap that let this defect ship undetected for months.

### 3. Independent, Adversarially-Tested Content Validator
- **SDLC stage:** Testing / Implementation
- **Type:** Detective
- **Failure addressed:** Content Validator logic error (C4/C5).
- **Description:** Treat the Validator itself as safety-critical code requiring its own test suite, including deliberately malformed/mismatched inputs, and independent code review — it is the last automated gate before global release and cannot be allowed to have unverified logic.

### 4. Staged / Canary Rollout for Rapid Response Content
- **SDLC stage:** Release / Deployment
- **Type:** Preventive
- **Failure addressed:** Simultaneous global blast radius (C6/C7/C9).
- **Description:** Apply the same phased-ring deployment already used for full sensor releases (e.g., 1% → 5% → 25% → 100%, with automated health checks between rings) to Rapid Response Content as well, so a defective file can only ever affect a small, monitored subset of machines before release halts automatically.

### 5. Automated Crash-Telemetry Monitoring with Auto-Rollback
- **SDLC stage:** Operations
- **Type:** Detective (with automated preventive response)
- **Failure addressed:** Delay between deployment and detection, and inability to reach already-affected machines (C7).
- **Description:** Instrument the fleet to detect abnormal crash-loop telemetry within seconds of a content push and automatically halt further distribution and revert the channel file server-side, without waiting for a human to notice — reducing the exposure window from the ~78 minutes seen in this incident to a matter of seconds, and preventing later rings from ever receiving the bad file.

### Coverage Summary
| Control | SDLC Stage | Type |
|---|---|---|
| 1. Runtime bounds checking | Design/Implementation | Preventive |
| 2. Equivalence-class test coverage | Testing | Preventive |
| 3. Adversarial Validator testing | Testing/Implementation | Detective |
| 4. Canary/staged rollout | Release/Deployment | Preventive |
| 5. Auto-rollback on crash telemetry | Operations | Detective |

Controls span **five** distinct SDLC stages (Design, Implementation, Testing,
Release/Deployment, Operations) — exceeding the minimum of three required.
