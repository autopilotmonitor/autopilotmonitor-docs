---
description: >-
  Deep-diving one enrollment — device details, live timeline, automated
  analysis, and the guided diagnosis view for failures.
---

# Session Details & Diagnosis

## Session detail

Opening a session from the dashboard shows everything Autopilot Monitor knows about that enrollment, updating live over SignalR while the enrollment runs. The page is a stack of collapsible sections with a scroll-spy sidebar:

* **Session Info** — status, enrollment duration, enrollment type, ConfigMgr detection, NTP offset.
* **Device Details** — a deep telemetry card: OS build/edition, per-adapter network configuration (Wi-Fi SSID and signal, proxy config, active interface), security posture (Secure Boot, TPM, BitLocker with encryption method), agent and IME versions (with an *outdated* badge when applicable), the full **Autopilot profile** including a clickable OOBE-configuration bitmask decoder, and hardware specs down to DIMMs, disks, battery health, and GPUs.
* **Enrollment Progress** — the phase timeline; click a phase to jump to its events. Pre-provisioning (White Glove) sessions show the technician phase and the user phase separately.
* **Analysis** — the [analyze rule](../rules/analyze-rules/README.md) findings as severity-colored cards with a confidence bar and a *"fires on X % of enrollments in your tenant"* context note from 30-day rule telemetry. Expanding a card reveals the explanation, remediation steps, related docs, and an **Evidence** block whose links jump to the exact triggering event in the timeline. **Analyze Now** re-runs all rules on demand (analysis also runs automatically at completion/failure).

<figure><img src="../.gitbook/assets/session-analysis-finding.png" alt="An expanded analyze-rule finding card with severity, confidence, explanation, remediation steps, and evidence"><figcaption><p>An expanded finding: severity and confidence at the top, the explanation and remediation steps in the middle, and the raw evidence — with a link to the exact triggering event — at the bottom.</p></figcaption></figure>

* **Performance** — charts from the periodic performance snapshots.
* **Script Executions** — platform and remediation scripts with their output (stdout visibility is a [tenant setting](../reference/settings.md#agent-parameters); stderr is always shown).
* **Downloads / Install Progress** — per-app download progress and the full install lifecycle, including Office Click-to-Run with a live timer.
* **Event Timeline** — the raw, phase-grouped event stream with severity filters (Info/Warning/Error/Critical), expand/collapse controls, and auto-scroll for live sessions.

**Header actions:** **Download Diagnostics** (when a diagnostics package was uploaded), **Diagnosis** (failed sessions), **Report Session** (send a report with comment and optional attachments), and — with [Admin Mode](../concepts/roles-and-permissions.md#admin-mode) on an *In Progress*/*Pending* session — **Succeed** / **Fail** overrides.

## Guided diagnosis

For a failed session, the **Diagnosis** button opens a focused, remediation-first view that answers one question: *why did this fail, and how do I fix it?*

* The **Primary Suspect** card presents the highest-confidence finding in plain language, with an *Evidence Found* list and a **Quick Fix** box whose remediation steps can be copied with one click.
* **Other Possibilities** lists lower-ranked findings; **Error Events** and **Warnings** summarize the raw error messages with timestamps.
* **View Evidence** links back into the full session's timeline, and **Full Details** returns to the complete session page.

The diagnosis view is deliberately minimal — no device details, no admin controls — making it the right link to share with a colleague who just needs to fix the device.
