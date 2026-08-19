# D1 — Failure Timeline
## T8: CrowdStrike Falcon Sensor / Channel File 291 Outage (July 19, 2024)

### System Purpose
CrowdStrike Falcon is a cloud-managed Endpoint Detection and Response (EDR)
platform. Its Windows sensor runs as a kernel-mode driver so it can inspect
system calls, processes, and inter-process communication (IPC) with the
privilege needed to catch malware before the OS itself is compromised.
To keep pace with new attack techniques without shipping a full new sensor
binary, CrowdStrike pushes "Rapid Response Content" — small, cloud-delivered
rule/configuration files ("Channel Files") — directly to every connected
sensor, multiple times a day, with no customer opt-in.

### Stakeholders
- CrowdStrike engineering, QA, and release/DevOps teams
- Enterprise customers running Falcon on production Windows fleets
  (airlines, hospitals, banks, broadcasters, government agencies)
- End users of those customers' systems (patients, passengers, citizens)
- Microsoft (platform on which the sensor runs in kernel mode)

### Timeline of Major Events

1. **Feb–Mar 2024** — CrowdStrike ships Falcon sensor 7.11, adding a new
   capability to detect malicious use of Windows named pipes by
   Command-and-Control (C2) frameworks. This introduces a new **IPC Template
   Type** that defines **21 input parameter fields** for matching.

2. **Mar–Apr 2024** — Three IPC Template Instances (content updates) using
   this new Template Type are released to production. All of them use
   **wildcard matching** on the 21st field. Because a wildcard trivially
   matches anything, this field is never meaningfully exercised in testing
   or in the field.

3. **19 July 2024, 04:09 UTC** — CrowdStrike deploys two new IPC Template
   Instances via **Channel File 291**. For the first time, one instance uses
   a **non-wildcard (specific) value** in the 21st field.

4. **04:09 UTC (same moment)** — The sensor's integration code only ever
   supplied **20 input values** to the Content Interpreter, not 21 (a
   count mismatch with the Template Type definition). With a real value now
   present in the 21st field, the Content Interpreter performs an
   **out-of-bounds memory read**, and the Content Validator — which was
   supposed to catch this — contains its own logic error and does not
   reject the file.

5. **04:09–05:27 UTC** — Every Windows machine with an active Falcon sensor
   that checks in during this window downloads Channel File 291 and, at the
   next IPC evaluation, the kernel-mode sensor driver crashes, taking the
   Windows kernel with it (Blue Screen of Death). Because the driver loads
   at boot, affected machines **boot-loop** into repeated BSODs.

6. **05:27 UTC** — CrowdStrike identifies the faulty file and pulls/reverts
   Channel File 291. This stops *new* machines from being affected, but does
   nothing for machines that already downloaded the bad file — the fix
   cannot reach a machine that is stuck in a crash loop before Windows
   fully boots.

7. **19 July 2024 onward** — An estimated **8.5 million Windows devices**
   are knocked offline. Remediation requires manual intervention per
   machine (boot into Safe Mode / WinRE, delete the offending file, reboot),
   which is extremely slow at enterprise scale, especially for remote or
   BitLocker-encrypted devices. Airlines ground flights, hospitals delay
   procedures, banks and broadcasters go dark; losses are estimated in the
   billions of dollars.

8. **24 July – 6 Aug 2024** — CrowdStrike publishes a Preliminary Post
   Incident Review, then a full 12-page external Root Cause Analysis. By
   **29 July 2024**, roughly 99% of sensors are back online.

### Immediate Failure
A logic/validation defect in a routine content update caused a kernel-mode
driver to read out-of-bounds memory, crashing Windows on every machine that
received the update.

### System Consequence
Global, near-simultaneous outage of millions of Windows endpoints across
critical industries — the largest IT outage of its kind to date.
