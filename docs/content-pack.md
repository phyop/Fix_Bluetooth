# Content Pack

Reusable public content for the `260709_Fix_Bluetooth` case study.

## Positioning

This project demonstrates evidence-based technical troubleshooting, privacy-aware public writing, and clear translation of a local debugging session into reusable engineering communication.

Core message:

```text
Diagnosed a Windows Bluetooth outage by separating service state, adapter runtime state, event-log evidence, hardware power state, and final validation.
```

Technical anchors to preserve:

- Windows Bluetooth support service was running: `bthserv: Running`
- local adapter was `Intel(R) Wireless Bluetooth(R)`
- public hardware ID prefix was `USB\VID_8087&PID_0033`
- full device instance ID was intentionally not published
- adapter was initially `Stopped`
- initial adapter state included `Status: Stopped`
- `pnputil` enable attempt reported `Device is already enabled.` while status remained `Stopped`
- restart attempt returned `Failed to restart device.` and `Access is denied.`
- `BTHUSB` logged `A command sent to the adapter has timed out. The adapter did not respond.`
- `BTHUSB` logged `The local Bluetooth adapter has failed in an undetermined manner and will not be used.` and `The driver has been unloaded.`
- recovery required full shutdown, waiting more than 15 seconds, booting again, and enabling the adapter from elevated PowerShell
- after power reset, adapter changed from `Stopped` to `Disabled`
- after elevated enable, adapter reached `Started`
- intermediate adapter state included `Status: Disabled`
- final adapter state included `Status: Started`
- final verification showed `Intel(R) Wireless Bluetooth(R): Started`, `bthserv: Running`, and no new Bluetooth `BTHUSB` errors

## Resume STAR

### Short Version

- Diagnosed and documented a Windows Bluetooth outage by tracing symptoms from repeated peripheral disconnects to an Intel adapter runtime failure, using Plug and Play state, `bthserv` status, and `BTHUSB` event-log evidence.
- Restored service by performing a full power-off reset for more than 15 seconds, enabling `USB\VID_8087&PID_0033\...` from elevated PowerShell, and verifying the adapter moved from `Stopped` to `Disabled` to `Started` with no new `BTHUSB` errors.
- Converted the incident into privacy-safe public documentation that preserved exact technical evidence while omitting the full device instance ID, machine names, account identifiers, paired-device addresses, and private screenshots.

### STAR Version

**Situation:** A Windows laptop lost usable Bluetooth after repeated peripheral disconnects. After Bluetooth was turned off, the Bluetooth icon and switch disappeared, making the issue look like a Windows UI problem.

**Task:** Identify whether the fault was a paired-device issue, service issue, driver issue, adapter runtime issue, or hardware power-state issue, then document a public-safe recovery path without exposing private identifiers.

**Action:** Verified `bthserv: Running`, queried Plug and Play state for `Intel(R) Wireless Bluetooth(R)`, found the adapter `Stopped`, reviewed `BTHUSB` System event logs, preserved the exact timeout and driver-unload text, performed a full shutdown for more than 15 seconds, enabled the adapter from elevated PowerShell with a redacted `pnputil /enable-device "USB\VID_8087&PID_0033\..."` command, and validated final adapter and service state.

**Result:** Restored Bluetooth by moving the adapter from `Stopped` to `Disabled` after hardware power reset and then to `Started` after elevated enable. Produced a public case study, Medium article draft, debug timeline, final-fix notes, privacy notes, and this content pack while preserving technical depth and removing the full device instance ID.

## LinkedIn Post

Bluetooth issues are easy to misdiagnose when the first symptom appears in a peripheral.

I documented a Windows case where a Bluetooth device kept disconnecting, then the Bluetooth icon and switch disappeared after Bluetooth was turned off.

The key was separating layers:

- `bthserv` was running
- paired-device history still existed
- `Intel(R) Wireless Bluetooth(R)` was visible but `Stopped`
- `BTHUSB` logged: `A command sent to the adapter has timed out. The adapter did not respond.`
- `BTHUSB` then logged: `The local Bluetooth adapter has failed in an undetermined manner and will not be used.` and `The driver has been unloaded.`

The fix was not re-pairing a device. The working recovery was:

```text
Full shutdown -> wait >15 seconds -> boot -> enable Intel Bluetooth from elevated PowerShell -> verify Started
```

After the power reset, the adapter changed from `Stopped` to `Disabled`. Then `pnputil /enable-device "USB\VID_8087&PID_0033\..."` restored it to `Started`.

Final validation:

```text
Intel(R) Wireless Bluetooth(R): Started
bthserv: Running
No new BTHUSB errors
```

I also wrote the public version with privacy constraints: no full device instance ID, no machine name, no account identifiers, no full paired-device hardware addresses, and no private screenshots.

Good troubleshooting is not just fixing the issue. It is preserving the chain of evidence so the next person can reason from symptoms to root cause.

## Commit Message

```text
docs: publish bluetooth recovery case study

Rewrite the README and Medium draft into public-facing documentation
while preserving the full debug trail: bthserv running, Intel adapter
Stopped/Disabled/Started states, BTHUSB timeout and driver-unload event
text, >15s full power-off reset, elevated pnputil enable step, and
privacy-safe hardware ID truncation.

Add a content pack with resume STAR, LinkedIn, PR copy, and future
extension ideas toward an AI Agent Consultant portfolio.
```

## Pull Request Copy

### Title

```text
Publish Windows Bluetooth recovery case study content pack
```

### Summary

- Rewrites `README.md` into a complete public case-study template with problem, environment, evidence, debug timeline, PowerShell enable steps, validation, privacy handling, and disclaimer sections.
- Rewrites `medium-bluetooth-debug-article.md` into a 1500-2500 word English article with title ideas, SEO metadata, tags, and a narrative debugging flow.
- Adds `docs/content-pack.md` with resume STAR bullets, LinkedIn post copy, commit message, PR description, and future extension ideas toward AI Agent Consultant positioning.

### Technical Details Preserved

- `bthserv: Running`
- `Intel(R) Wireless Bluetooth(R)`
- `USB\VID_8087&PID_0033`
- `Stopped` versus `Disabled` versus `Started`
- `Device is already enabled.` with adapter still `Stopped`
- `Failed to restart device.` and `Access is denied.`
- `A command sent to the adapter has timed out. The adapter did not respond.`
- `The local Bluetooth adapter has failed in an undetermined manner and will not be used.`
- `The driver has been unloaded.`
- full shutdown and more than 15 seconds powered off
- elevated PowerShell enable command
- final event-log check with no new Bluetooth `BTHUSB` errors
- privacy constraint against publishing the full device instance ID

### Validation

- Confirmed required diagnostic strings remain present.
- Confirmed Medium article draft word count is within 1500-2500 words.
- Confirmed the public command keeps the full instance ID truncated.

## Future Extensions Toward AI Agent Consultant

### 1. Privacy-Safe Troubleshooting Corpus

Turn this case into one example in a small corpus of public, sanitized troubleshooting incidents. Each incident should preserve exact technical evidence, remove private identifiers, and include a clear reasoning trail.

Consulting angle:

```text
I help teams convert messy operational incidents into privacy-safe knowledge bases that humans and AI agents can reuse.
```

### 2. AI Agent Diagnostic Playbook

Convert the debug timeline into an agent playbook:

1. Ask whether one paired device failed or the Bluetooth switch disappeared.
2. Check `bthserv`.
3. Query Plug and Play adapter state.
4. Inspect `BTHUSB` System event logs.
5. Distinguish `Started`, `Stopped`, and `Disabled`.
6. Recommend a full power-off reset only when evidence points to adapter or hardware state.
7. Require final validation before declaring success.

Consulting angle:

```text
I design AI troubleshooting agents that ask for evidence, separate system layers, and avoid premature fixes.
```

### 3. Redaction Workflow

Build a lightweight redaction checklist for logs and screenshots:

- remove full device instance IDs
- remove machine names
- remove Windows user names
- remove account identifiers
- remove paired-device hardware addresses
- crop or avoid screenshots with private device names
- preserve provider names, service names, event text, state transitions, and generalized commands

Consulting angle:

```text
I help teams publish useful technical knowledge without leaking user, device, or account identifiers.
```

### 4. Windows Support Automation Prototype

Create a small PowerShell collector that gathers non-sensitive evidence:

- `bthserv` status
- Bluetooth adapter friendly name and status
- redacted hardware ID prefix
- recent `BTHUSB` event messages
- final verification checklist

The collector should redact or omit the full instance ID by default.

Consulting angle:

```text
I prototype support tools that produce agent-ready diagnostics while respecting privacy boundaries.
```

### 5. Incident-to-Content Pipeline

Use this repository as a template for turning one debugging session into multiple outputs:

- README case study
- Medium article
- timeline doc
- final fix doc
- privacy doc
- resume STAR
- LinkedIn post
- commit and PR copy
- AI agent extension roadmap

Consulting angle:

```text
I help technical professionals turn real troubleshooting work into portfolio assets, support playbooks, and AI-agent-ready documentation.
```

## One-Line Portfolio Summary

Diagnosed a Windows Intel Bluetooth adapter outage using service checks, Plug and Play state, `BTHUSB` event logs, hardware power reset, and elevated PowerShell recovery, then converted the incident into privacy-safe public documentation and AI-agent-ready consulting material.
