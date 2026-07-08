---
description: >-
  User-facing changes to the Autopilot Monitor agent, newest first — only
  changes that affect agent behavior on the device.
---

# Agent Changelog

**Current versions:** ![Latest agent version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fautopilotmonitor.blob.core.windows.net%2Fagent%2Fversion.json&query=%24.version&label=Agent&prefix=v&color=2563eb) ![Latest bootstrapper version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fautopilotmonitor.blob.core.windows.net%2Fagent%2Fversion.json&query=%24.bootstrapVersion&label=Bootstrapper&prefix=v&color=2563eb)

User-facing changes to the Autopilot Monitor agent, newest first. Only includes changes that affect agent behavior on the device.

## July 2026

* Windows Update-during-OOBE detection — the agent watches the Windows Update client log during enrollment and reports when a quality/cumulative update **starts**, **succeeds**, or **fails** (with the update title and a decoded HRESULT). A startup backfill catches updates that ran on the "Getting updates" screen before the agent even started, and a pending-reboot registry check corroborates an update that needs a restart. Surfaces a real enrollment-breaker that no other tool shows well; graded by the new ANALYZE-DEV-004 / ANALYZE-DEV-005 rules
* Best-effort emergency-break report — when the agent hits its 48-hour absolute self-destruct it now sends a final "agent is gone" signal (best effort — may be lost if the device is fully offline) so the backend can settle the session's outcome immediately instead of waiting out the grace window
* Registry gather collector reads the 64-bit view by default (bitness-independent) and supports `emitOnlyIfExists`, so a rule can emit only when the target key is actually present instead of asserting absence

## June 2026

* Optional keep-awake during the User-ESP (Account Setup) phase — when enabled per tenant (off by default), the agent keeps the device awake so it can't drop into standby and stall app installs or account setup; the hold is released automatically once the phase completes, and device reboots are unaffected
* Fewer stuck enrollments — Classic sessions where a skipped user-ESP app or an advisory "continue anyway" ESP failure left completion waiting now resolve through a short completion deadline instead of running into the 6-hour timeout
* Desktop-arrival detection no longer stalls on devices where the owner lookup failed — the agent resolves the signed-in user via a Windows session (WTS) query, with the previous method as fallback
* When an app still installing blocks enrollment completion, the agent now names the specific app holding up the AccountSetup gate instead of just reporting a stall
* Office (M365 Apps) install tracking is more accurate — pre-installed OEM / consumer Office no longer triggers a false "install failed", and Office install progress now survives agent restarts and reboots
* Low-disk-space warning — agent emits a one-shot warning when free space drops below 2 GB during enrollment (re-arms once space recovers above 3 GB)
* Repetitive ModernDeployment event bursts are rolled up into a single entry and excluded from the idle timer — less timeline noise, and they no longer keep the collector awake
* Hello for Business policy is no longer reported as `not_configured` when it simply couldn't be detected, avoiding misleading Hello status
* New liveness signals show what enrollment completion is waiting on, and flag a session that is parked without a deadline — making stuck sessions easier to spot
* Startup events (e.g. timezone, NTP, geo-location) are de-duplicated across agent restarts, so a reboot mid-enrollment no longer repeats them in the timeline
* RealmJoin client detection is now an opt-in per-tenant setting (off by default) and additionally reports the RealmJoin release channel alongside its version
* Agent records the device's outbound (egress) IP for network correlation during enrollment
* Each agent request now carries a correlation ID for end-to-end tracing across agent and backend, making request-level troubleshooting easier
* Microsoft 365 Apps (Office) install tracking — agent surfaces the real Office Click-to-Run install that the Intune "integrated" app hides (IME reports done within a minute or two while Office keeps streaming in the background); shown as its own install row with live Delivery Optimization download progress
* Provisioning-package detection — agent scans for Windows provisioning packages (`.ppkg`) applied to the device and reports them in a single scan event; security rules flag packages outside a built-in allow-list of Windows-inbox packages
* AutoLogon detection — agent reports the device's automatic sign-in (AutoLogon) configuration; security rules now flag it only when a plaintext password is actually stored, so normal Autopilot enrollments no longer produce false positives
* ESP sub-category state changes are now surfaced even when they aren't failures (e.g. retry or recovery transitions) for clearer visibility into ESP progress
* Stall-probe file and registry scans now enforce a hard timeout so a slow or locked source can no longer hang the probe

## May 2026

* RealmJoin client detection — agent detects the RealmJoin client, reports its version, tracks deployment-phase changes, and surfaces per-package install progress
* Device hardware now reports CPU architecture (`x86` / `x64` / `ARM` / `ARM64`)
* Startup power-state check — agent warns when the device is running on battery below 80%, a frequent driver behind power-management enrollment stalls (enrollments are more reliable on AC power)
* ESP app-install failures are now classified by their HRESULT, surface all failure types (failed / not-installed / error), and a 30-second settle window catches results that arrive late
* Crash mini-dumps are captured for deeper post-mortem analysis when a previous-run crash is detected
* More accurate Health Script results — compliant detections are no longer mislabeled as failed, and detection failures now produce an actionable message for admins
* Fewer false enrollment failures — ESP "continue anyway" with AccountSetup, and self-deploying scenarios, now get a settle window before a failure is finalized
* Shutdown, diagnostics, and summary events are no longer dropped after a terminal enrollment decision
* Bootstrap reports its version so the bootstrap script version is visible in the session
* Agent retries the remote-config fetch with wire-visible fallback when the first attempt fails
* Security hardening for agent self-update, diagnostics URL handling, and PowerShell argument escaping
* **Agent V2 is now the primary production line** — V2 replaces V1 as the default install. Existing V1 devices keep working; new installs ship the V2 build (bootstrap script and binary renamed from `.V2` to standard)
* Health Scripts lifecycle monitoring — detection, remediation, and post-remediation phases are each captured as separate timeline events with a live "script running" indicator before the result lands
* Apps still installing when ESP-Apps times out are flagged as "likely stuck" instead of disappearing from the timeline — admins now see the app name and a hedged outcome
* ASR / EDR-blocked install handoff no longer strands devices — runtime spawn fails soft and the BootTrigger task picks the agent back up on next reboot
* Hello-disabled enrollments now complete reliably — the Classic v1 path no longer deadlocks waiting for a Hello signal that will never arrive (previously ran into the 6h max-lifetime timer)
* AccountSetup must truly succeed before Hello can trigger completion — prevents premature `enrollment_complete` when AccountSetup actually failed
* Hybrid User-Driven (HAADJ) enrollment-completion gaps closed — more completion paths recognized, fewer sessions stuck in the timeout fallback
* TPM PSS unsupported is reported as a distinct distress reason — older devices (e.g. Surface Book 1 with 2015-era Infineon TPM firmware) that can't do RSA-PSS now get a clear failure category instead of a generic Schannel error
* Intune dual-stack certificate selection fix — on devices with both MDM and MMP-C client certs, the agent now picks the correct _Microsoft Intune MDM Device CA_ cert and avoids backend chain-validation rejection
* Client certificate rejections surface with structured backend warnings and V2 distress cert-context (thumbprint, subject, issuer, validity) — easier to diagnose mTLS auth failures
* Tenant ID resolution falls back to the CloudDomainJoin registry (`TenantInfo` + `JoinInfo`) when the Enrollments key is empty — covers pre-Type-6 enrollments and MS-Organization-Access cert paths
* Event-driven Tenant ID wait via RegistryWatcher — agent reacts to registry changes during pre-enrollment instead of polling
* Desktop Arrival Detector liveness signals (started / first-poll / no-candidate) help distinguish "agent dead post-reboot" from "user never logged in" in sessions that time out without a desktop_arrived
* Detailed shutdown reasons — when the agent exits unexpectedly (Ctrl+C, process exit, unhandled exception, runtime host exit) the cause is recorded in the timeline
* Prior-run crash is surfaced in the next session via a "death rattle" event, so a mid-enrollment agent crash is visible instead of silently lost
* V2 diagnostics ZIP is size- and count-capped with streaming output — no more multi-gigabyte uploads on long or noisy sessions
* Diagnostics ZIP now includes the `State` and `Spool` folders for richer post-mortem analysis
* Agent log files rotate at a size cap — no unbounded growth on long-running devices
* New "Submit Logs" page — admins can upload diagnostics files for analysis even when no active session exists on the device
* Delivery Optimization breakdown adds MCC (Microsoft Connected Cache) and LinkLocal sources across `download_progress` and `do_telemetry` events
* Software inventory now correctly enumerates Azure AD and personal MSA user profiles (these SIDs were previously skipped)
* Hardware spec event reports VM detection — security analyze rules skip VMs to avoid false-positive vulnerability reports
* Bootstrap `--install` mode preserves an existing `bootstrap-config.json` instead of clobbering customer settings on re-install
* Optional "enrollment started" webhook fires at session registration — opt-in notification at the very start of an enrollment

## April 2026

* Delivery Optimization monitoring — agent tracks Windows DO download activity (OS level) during app installs and reports download performance metrics per application
* ConfigMgr co-management detection — agent detects Configuration Manager client presence and reports co-management status with confidence scoring
* Non-whitelisted hardware detection with optional admin alerts when devices with unapproved hardware models enroll
* IME version change tracking — Intune Management Extension version updates are recorded
* Hello for Business skip detection — agent now distinguishes between Hello setup being completed, timed out, or explicitly skipped
* User-profile-aware diagnostics — gather rules and diagnostics log paths can reference the logged-on user profile directory
* Improved vulnerability matching accuracy using fuzzy Jaro-Winkler scoring
* Faster agent startup through optimized initialization flow
* ESP provisioning status verification before enrollment completion — agent checks category outcomes and waits up to 30s for pending results to settle
* Structured error codes (exit codes, HRESULT) extracted from IME log patterns and included in timeline events
* Dual-hash integrity verification — ZIP package hash checked at download, separate EXE hash verified at runtime against backend to detect post-installation tampering
* Vulnerability matching improvements — confidence levels, platform-aware filtering, and exclude patterns for more accurate reports
* Vulnerability reports now available during pre-provisioning (White Glove) sessions
* More reliable enrollment summary dialog launch with desktop fallback strategy
* PowerShell script output is now fully captured in the timeline (multi-line output was previously truncated)
* More reliable bootstrap and download handling with improved timeout and rate-limit behavior
* Agent reports self-update events so updates are visible in the session timeline
* Emergency channel — agent can send distress signals when it detects critical failures
* ESP "resumed" event is now only emitted for Hybrid Join scenarios (avoids noise on other paths)
* Improved crash recovery — completion state is persisted so the agent can resume correctly after an unexpected restart

## Late March 2026

* Agent crash detection — crashes are automatically detected and reported to the backend
* SHA-256 integrity verification for agent downloads (bootstrapper + self-updater verify hash before install)
* Reboot tracking — reboots during enrollment are now tracked and visible in the timeline
* NTP time sync check with clock skew warning when device time is significantly off
* Automatic timezone detection and configuration
* SecureBoot certificate collection for security posture reporting
* IME process watcher — detects when the Intune Management Extension starts or stops
* Network change detection — captures network adapter changes during enrollment
* Agent self-update mechanism — outdated agents in the field update themselves automatically
* Unrestricted mode option (per-tenant) to disable most guard rails
* Notification system reworked — supports Teams (legacy + Workflow), Slack, and custom webhooks

## Mid March 2026

* Software inventory collection with automatic vulnerability correlation (CVE matching)
* Hardware specification event — detailed hardware info collected and reported
* Agent shutdown event — clean shutdown is now explicitly tracked
* Postponed app detection and handling during enrollment
* Self-deploying mode detection and event tracking
* Enrollment summary dialog shown on the device after enrollment completes
* ESP provisioning status tracking — catches non-IME errors like certificate failures
* PowerShell script execution tracking during enrollment
* Clock skew detection with geo-location failure reporting
* Community analyze rules support

## Early March 2026

* Bootstrap session support — monitoring starts before MDM enrollment (during OOBE)
* ESP configuration detection — identifies ESP settings on the device
* TPM info collection for device details
* Activity-aware idle timeout replaces fixed 4-hour collector limit (default: 15 min idle)
* Reliable session end-detection for all deployment scenarios (user-driven, pre-provisioning, hybrid)
* Network performance data collection (latency, throughput)
* Geographic location support via IP-based lookup
* Emergency break — remote kill switch to stop agents
* Automatic retry on transient backend errors
* Custom User-Agent header for easier firewall allowlisting
* ESP state tracking via registry watcher
* XML and JSON file gathering in diagnostics
* Configurable `--await-enrollment` parameter for pre-enrollment wait

## Late February 2026

* Pre-Provisioning (White Glove) support — full end-to-end monitoring of pre-provisioning sessions
* mTLS for all agent-to-backend communication (consolidated endpoints)
* Diagnostics SAS URL fetched on-demand — no longer stored on disk
* Max collector duration policy (configurable per tenant)
* Diagnostics package upload from device
* Configurable reboot-on-complete and keep-logfile options via remote config
* Configurable diagnostics log paths (global + per-tenant)
* Lenovo model detection fix (WMI query)

## Mid February 2026

* Windows Autopilot v2 (Device Preparation) support
* GatherRules guard rails — prevents collection of overly broad paths
* IME log replay for testing and demos (`--replay-log-dir`)
* Agent state persistence — survives reboots and resumes monitoring
* Embedded Intune root + intermediate certificates for chain validation
* OS info and boot time collection
* Hello screen detection improvements
* Download progress tracking

## Early February 2026

* Initial agent release
* Real-time enrollment telemetry (IME log parsing, ESP phases, app installs)
* Geolocation support for enrollment sessions
* Hello screen detector for enrollment completion
* Reboot-on-complete support
* Session ID persistence across agent restarts
* Bootstrap token authentication for pre-MDM scenarios
