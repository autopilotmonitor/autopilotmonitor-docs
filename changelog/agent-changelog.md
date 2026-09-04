---
type: Changelog
tags: [agent, changelog]
description: >-
  User-facing changes to the Autopilot Monitor agent, newest first — only
  changes that affect agent behavior on the device.
---

# Agent Changelog

**Current versions:** ![Latest agent version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fdownload.autopilotmonitor.com%2Fagent%2Fversion.json&query=%24.version&label=Agent&prefix=v&color=2563eb) ![Latest bootstrapper version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fdownload.autopilotmonitor.com%2Fagent%2Fversion.json&query=%24.bootstrapVersion&label=Bootstrapper&prefix=v&color=2563eb)

User-facing changes to the Autopilot Monitor agent, newest first. Only includes changes that affect agent behavior on the device.

Per-version release notes: [GitHub Releases](https://github.com/okieselbach/AutopilotMonitor/releases).

## September 2026

* On Hybrid Join devices the agent now reports whether the signed-in user obtained an Entra token, and the sign-in-overdue warning reflects the observed desktop instead of the provisioning placeholder
* User Device Registration failures (events 304 and 305) now appear on the session timeline
* Keep-awake during User-ESP no longer releases early on an interim Account Setup checkpoint, so the device stays awake until the User-ESP page closes
* Standby episodes from before the agent started no longer trigger the sleep-during-enrollment warning

## August 2026

* RealmJoin self-updates during enrollment are now detected, and the session shows the updated version
* IME log pattern matching no longer skips patterns on heavily loaded virtual machines
* IME process monitoring re-attaches after an IME restart, so a later crash is still reported
* Local admin analysis no longer flags localized built-in accounts (e.g. `Gast`, `Administrateur`), and a local account that is signed in when enrollment completes is no longer exempted from the check
* Local admin analysis now includes disabled accounts and reports each account's membership in the local Administrators group, so a dormant admin backdoor no longer goes unnoticed
* When the backend reports that the persisted session id belongs to a different device identity (an Intune re-enrollment without a wipe), the agent now starts a fresh session instead of shutting down after repeated authorization failures
* On-demand log collection from the portal now works right after enabling diagnostics upload, and a skipped collection reports why (upload mode off, no destination configured)
* Enrollments on devices with no user-assigned apps are no longer reported as failed, and an enrollment is no longer failed while RealmJoin is still installing
* Session registration after a mid-enrollment reboot now waits for the network link and retries longer, so Wi-Fi devices no longer lose their session when the agent restarts before the connection is back
* Diagnostics packages now include the RealmJoin logs when the RealmJoin Watcher is enabled
* Diagnostics packages now include the Device Preparation bootstrapper event log on Autopilot Device Preparation enrollments
* Live power-state tracking during enrollment, with warnings when the device is unplugged or the battery runs low
* System clock changes and standby periods during enrollment are now detected and shown in the timeline
* Enrollment completion now recovers after a forced reboot during ESP
* More reliable detection and completion for Autopilot Device Preparation enrollments
* Enrollment summary dialog now includes Microsoft 365 Apps and CSP-installed packages
* New Bootstrap MSI to deploy the agent as an MDM line-of-business app
* App install tracking now cross-checks the Windows registry in addition to IME logs
* RealmJoin setup timeout extends automatically while a deployment is still active
* Opt-in Delivery Optimization group ID derived from the local network, so devices peer with each other
* Opt-in observation mode lets ESP "continue anyway" failures recover before failing the enrollment
* Download sizes are now reported correctly for compressed transfers
* Missing Autopilot profile is informational instead of a warning on Cloud PCs
* App install timings now reflect the final install attempt
* WMI gather rules can now select specific properties instead of only `SELECT *`
* Windows 365 Cloud PC enrollments can now be monitored (opt-in per tenant)
* Cloud PCs are flagged in sessions and in the Devices Not Registered report
* Devices with older bootstrap configurations no longer skip the pre-enrollment tenant-ID wait
* Gather-rule phase and event triggers now fire in sync with the session timeline
* Fixed inflated session durations on devices whose time zone changed during enrollment

## July 2026

* WiFi network details no longer depend on the OS display language
* Diagnostics are now uploaded when pre-provisioning (White Glove) completes
* New opt-in gather-rule debug log for troubleshooting rule matching
* Events replayed from before the agent started are now marked as backfilled in the timeline
* ESP app tracking now captures user-phase apps
* App install durations now account for installs retried after a failure
* A late agent start on a successful enrollment is now reported as informational instead of a warning
* Sessions now record the average agent-to-backend API latency
* An ESP app failure the user later resolves with "Try again" no longer fails the enrollment
* Old IME log entries are no longer replayed as fresh activity after an agent restart
* New warning when Account Setup hangs because an ESP policy provider never finished
* Mid-enrollment reboots are now attributed to the MDM policy that requested them
* An aborted RealmJoin setup no longer holds up enrollment completion
* Secure Boot 2023 certificate status is now verified against firmware
* Gather rules can be scoped to enrollment phases and can fire once when a phase ends
* Diagnostics path allow-list is now enforced in every gather collector
* Local admin monitoring allow-list now supports wildcard patterns
* Agent records the Windows OOBE state at startup and when OOBE completes during the session
* Trace-event upload now honors the `SendTraceEvents` setting
* Agent downloads now use `download.autopilotmonitor.com` — update firewall allow-lists if needed
* Agent release packages now include build provenance attestation
* Devices without an assigned Autopilot profile are now flagged with a warning
* Autopilot event-log errors from before the agent started are now included in the session timeline
* Enrollment no longer completes while the user is still in the Windows Hello setup wizard
* Fixed premature completion during the user phase of pre-provisioned (White Glove) enrollments
* Account Setup completion now waits until all required apps have finished installing
* Faster completion detection once the user reaches the desktop after Account Setup
* Fixed false failures on finished enrollments where the Hello policy state could never be read
* Device details no longer show "Unknown" manufacturer/model/serial early in OOBE
* Session now lists which apps ESP is tracking in each setup phase
* Fewer false "app stuck" warnings for apps that were never marked for install
* OS build changes during enrollment are detected even when Windows Update doesn't log them
* Fixed false enrollment failures during Account Setup on sessions that were still installing apps
* Passive internet-bandwidth estimate during app installs, with a LAN/WAN split
* MSIX/Store app install failures during ESP now identify the specific failing package
* Fewer false ESP failures when Windows retracts a failure after a retry
* More accurate script run-time shown for Remediation and Platform Scripts
* Fixed a rare crash-loop that could follow a self-update restart
* Windows Updates running during enrollment are now detected and reported
* Emergency-break report when the agent hits its 48-hour lifetime limit
* Registry gather collector reads the 64-bit view by default and supports emitting only when the key exists

## June 2026

* Optional keep-awake during Account Setup prevents standby from stalling app installs (off by default)
* Skipped ESP apps and advisory "continue anyway" failures no longer run into the 6-hour timeout
* Desktop-arrival detection no longer stalls when the device-owner lookup fails
* Agent now names the specific app holding up Account Setup completion
* More accurate Office (M365 Apps) install tracking — no false failures for pre-installed Office
* Low-disk-space warning when free space drops below 2 GB during enrollment
* Repetitive ModernDeployment event bursts are rolled up into a single timeline entry
* Hello for Business policy is no longer reported as "not configured" when it simply couldn't be detected
* New liveness signals show what enrollment completion is waiting on
* Startup events are no longer repeated after a mid-enrollment reboot
* RealmJoin client detection is now opt-in per-tenant (off by default) and reports its release channel
* Agent records the device's outbound IP for network correlation
* Each agent request now carries a correlation ID for easier troubleshooting
* Microsoft 365 Apps (Office) install tracking surfaces the real Click-to-Run install progress
* Provisioning-package (`.ppkg`) detection, with security rules flagging packages outside a built-in allow-list
* AutoLogon detection flags only when a plaintext password is actually stored
* ESP sub-category state changes are now surfaced even when they aren't failures
* Stall-probe file and registry scans now enforce a hard timeout

## May 2026

* RealmJoin client detection — version, deployment-phase changes, and per-package install progress
* Device hardware now reports CPU architecture (`x86` / `x64` / `ARM` / `ARM64`)
* Startup power-state check warns when the device is running on battery below 80%
* ESP app-install failures are now classified by HRESULT, with a 30-second settle window for late results
* Crash mini-dumps are captured for deeper post-mortem analysis
* More accurate Health Script results — no more false "failed" labels, and failures show an actionable message
* Fewer false failures — ESP "continue anyway" and self-deploying scenarios now get a settle window before failing
* Shutdown, diagnostics, and summary events are no longer dropped after a terminal enrollment decision
* Bootstrap reports its version so it's visible in the session
* Agent retries the remote-config fetch with a fallback when the first attempt fails
* Security hardening for agent self-update, diagnostics URLs, and PowerShell argument handling
* Agent V2 is now the primary production line — new installs ship V2 by default, existing V1 devices keep working
* Health Scripts lifecycle monitoring — detection, remediation, and post-remediation shown as separate timeline events
* Apps still installing when ESP-Apps times out are flagged "likely stuck" instead of disappearing
* ASR / EDR-blocked install handoff no longer strands devices — agent resumes on next reboot
* Hello-disabled enrollments now complete reliably instead of running into the 6-hour timeout
* Fixed premature completion signals when AccountSetup actually failed
* Hybrid User-Driven (HAADJ) enrollment-completion gaps closed — fewer sessions stuck in the timeout fallback
* TPM PSS-unsupported devices now get a clear failure category instead of a generic Schannel error
* Fixed certificate selection on devices with both MDM and MMP-C client certs
* Client certificate rejections now show detailed context — easier to diagnose mTLS auth failures
* Tenant ID resolution now falls back to the CloudDomainJoin registry when the primary key is empty
* Event-driven Tenant ID wait — agent reacts to registry changes during pre-enrollment instead of polling
* New liveness signals help distinguish a dead agent from a user who never logged in
* Detailed shutdown reasons recorded when the agent exits unexpectedly
* A prior-run crash is now surfaced in the next session instead of silently lost
* V2 diagnostics ZIP is size- and count-capped — no more multi-gigabyte uploads on long sessions
* Diagnostics ZIP now includes the State and Spool folders for richer post-mortem analysis
* Agent log files rotate at a size cap — no unbounded growth on long-running devices
* New "Submit Logs" page — admins can upload diagnostics even without an active session
* Delivery Optimization breakdown adds MCC and LinkLocal sources
* Software inventory now correctly enumerates Azure AD and personal MSA user profiles
* Hardware spec event reports VM detection — security rules skip VMs to avoid false positives
* Bootstrap `--install` mode preserves existing settings instead of clobbering them on re-install
* Optional "enrollment started" webhook fires at session registration

## April 2026

* Delivery Optimization monitoring — download performance metrics per app during install
* ConfigMgr co-management detection with confidence scoring
* Non-whitelisted hardware detection with optional admin alerts
* IME version change tracking
* Hello for Business skip detection — distinguishes completed, timed out, or explicitly skipped
* User-profile-aware diagnostics — gather rules can reference the logged-on user's profile directory
* Improved vulnerability matching accuracy
* Faster agent startup
* ESP provisioning status verification before completion, with a 30s settle window for pending results
* Structured error codes (exit codes, HRESULT) extracted from IME logs and included in timeline events
* Dual-hash integrity verification detects tampering between download and install
* Vulnerability matching improvements — confidence levels, platform-aware filtering, exclude patterns
* Vulnerability reports now available during pre-provisioning (White Glove) sessions
* More reliable enrollment summary dialog launch
* PowerShell script output is now fully captured in the timeline
* More reliable bootstrap and download handling
* Agent reports self-update events so updates are visible in the session timeline
* Emergency channel — agent can send distress signals when it detects critical failures
* ESP "resumed" event is now only emitted for Hybrid Join scenarios
* Improved crash recovery — completion state is persisted so the agent can resume after an unexpected restart

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
