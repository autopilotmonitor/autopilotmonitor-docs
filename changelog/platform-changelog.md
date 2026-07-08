---
description: >-
  Significant platform changes — portal, backend, and data-flow updates,
  newest first.
---

# Platform Changelog

This changelog tracks significant platform changes during Private Preview — architecture updates, data flow changes, and anything else that might briefly affect the UI or monitoring data. If something looks off, check here first. A recent entry might explain it.

Found a bug or want to give feedback? [Open a GitHub Issue](https://github.com/okieselbach/Autopilot-Monitor/issues) — it helps more than you might think.

## 2026-07-08 — Honest enrollment outcomes (timeout ≠ failure) and Windows Update-during-OOBE detection

_Platform Update — 12:00 CET_

* **Timeout is no longer a failure — two new session states** — The 5-hour maintenance sweep used to mark *every* silent session **Failed**, conflating "the agent stopped sending telemetry" with "the enrollment broke" and badly inflating failure rates. The backend now classifies a silent session from the evidence it already holds: a device that finished Device Setup and went quiet waiting on the user becomes **Awaiting User** (non-terminal), and a session that expires with neither a completion nor an explicit failure signal settles as **Incomplete** (terminal, but **not** a failure). Only real, explicit failures stay **Failed**. Both new states appear as status pills on the Dashboard and session detail. See [Sessions & Statuses](../concepts/sessions-and-statuses.md).
* **Late completions are reconciled to Succeeded** — A user often finishes the account phase hours (or a day) after the sweep already ran. When a genuine completion signal later arrives, the session is upgraded to **Succeeded** — even from Failed, Incomplete, or Awaiting User. The backend waits out the agent's 48-hour absolute cap plus a small buffer (~51 h) before settling *Awaiting User* → *Incomplete*, so legitimately late completions still land. This is pure backend bookkeeping — no agent heartbeat, no extra device load.
* **Honest Fleet Health** — The **Success Rate** denominator is now terminal-only (Succeeded + Failed); Incomplete sessions are counted on a dedicated **Incomplete** stat card and excluded from the failure rate. A fleet of laptops closed mid-ESP no longer reads as a fleet of broken enrollments.
* **Windows Update during OOBE is now visible** — A cumulative/quality update installing during the "Getting updates" screen or the ESP *Install Windows quality updates during OOBE* option can silently break or stall an enrollment — a blind spot the Intune console doesn't surface. The agent now watches the Windows Update client log during enrollment (with startup backfill for updates that ran before it started) and emits update started / succeeded / failed events with decoded HRESULTs. Two new analyze rules grade them: **ANALYZE-DEV-004** (a WU **failed** mid-enrollment, high) and **ANALYZE-DEV-005** (a WU **installed** mid-enrollment, info), corroborated by a pending-reboot registry snapshot. See [Built-in Rules](../rules/analyze-rules/built-in-rules.md#device).
* **Self-maintaining rule lifecycle** — Gather rules now get the same automatic sunset/cleanup that analyze rules already had (a rule removed from the product is cleanly retired from all tenants), and rule provenance now distinguishes product-shipped rules from GitHub-contributed rules so a community rule that's ahead of the deployed build is never mistaken for "removed" and hidden.

## 2026-06-16 — Software hub with self-service vulnerability exposure, faster Fleet Health, and expanded MCP tools

_Platform Update — 12:00 CET_

* **Software hub with self-service vulnerability exposure** — The App Health page is now a tabbed Software hub bringing app installs, installed-software inventory, and CVE/KEV vulnerability exposure together in one place. For the first time, tenant admins can see their own fleet's vulnerability exposure (Fleet Exposure) — previously a Global-Admin-only view — with per-device matches, a severity breakdown, and data-source sync-status counters.
* **Faster Fleet Health on large fleets** — Fleet Health now loads from server-aggregated metrics endpoints instead of draining up to 200,000 sessions into the browser, so the dashboard opens far faster on big fleets with identical numbers.
* **"Not registered" devices overview** — A new view under enrollment validation lists, per serial number, devices rejected with HTTP 403 over the last 14 days because they were not in the tenant's Autopilot or Corporate Identifier registry.
* **Richer session detail** — The Session Info card now shows Reboots, Enrollment Type (Device Preparation / Autopilot, with PreProvisioned and Gather Rules qualifiers), Join Type (Hybrid / Entra), and a "last contact" tooltip. Per-session reboot count is also stored and queryable via MCP.
* **Office & RealmJoin install rows** — Microsoft 365 Apps (Office) and RealmJoin packages now appear as first-class rows in the session Install Progress alongside Intune apps, with live timers, durations, and exit-code badges. The RealmJoin agent version is shown in the session System panel, and the RealmJoin watcher is now a per-tenant portal toggle (default off).
* **Expanded MCP tools** — New `get_software_inventory` and `get_app_install_metrics` tools surface installed-software inventory and per-app install metrics (including Delivery Optimization peer/MCC offload savings); `get_audit_logs` gains action / performed-by / entity-type / entity-id filters; `list_tenants` and a fleet vulnerability summary were added; raw session/event query tools now return literal table rows; hybrid event search is genuinely semantic; and the tool surface is role-aware so Global-Admin-only tools are hidden from regular users. Analysis output also resolves rule-template placeholders so explanations never leak raw `{{...}}` tokens.
* **Clearer error reporting** — Event and error views now show enriched, human-readable error-code descriptions and clearly distinguish a detection-failure from an install-failure from a "likely stuck" enrollment.
* **Deep links survive re-authentication** — Opening a session, diagnosis, or other in-app link in a new tab now returns you to that page after login instead of dropping you on the dashboard.
* **Health dashboard MCP card** — The Health dashboard adds an MCP Server card with version and wake-aware cold-start handling, alongside the SignalR quota card.
* **Device auto-block on runaway sessions** — Devices that emit an excessive number of session events can now be auto-blocked and their sessions killed, with block/kill-by-session-ID shortcuts and ops-alert integration.
* **CPU architecture** — Device hardware now reports CPU architecture (`x86` / `x64` / `ARM` / `ARM64`) as a queryable property.
* **Security hardening** — Added `nosniff` and stricter `x-forwarded-for` handling, tightened query-filter validation, enforced tenant-id presence on scoped endpoints, and patched a prototype-pollution advisory in a web dependency.

## 2026-05-19 — Safe cascade-delete & tenant offboarding, Intune script display names, full Remediation lifecycle, and Agent V2 robustness

_Platform Update — 12:00 CET_

* **Safe session cascade-delete with restore window** — Session deletion is now a snapshot-based, crash-safe cascade across all related tables (events, audit, app installs, vulnerability data, software inventory, etc.) with a deterministic order, hash-verified manifest, and a 33-day restore window.
* **Async tenant offboarding** — Tenant offboarding from Settings is now a queued async cascade that flips the tenant to disabled, waits for the config cache to drain, then safely wipes all 24 tenant-scoped tables with fetch-verify-delete semantics. The UI shows a minute-based countdown and you can close the tab — deletion finishes in the background. Self-service re-onboarding still works after a clean offboard.
* **Optional Graph add-on — Intune script display names** — A new Optional Graph capabilities page lets tenant admins grant a tightly-scoped Graph permission to resolve _real_ Intune Platform Script and Remediation display names in session timelines (instead of bare IDs). Opt-in only — no app manifest change for tenants who skip it. The page provides a copy-paste PowerShell command (and a `Grant-AutopilotMonitorAddOn.ps1` helper) with idempotent grant / verify / revoke flows.
* **Full Intune Remediation lifecycle** — Health scripts are now captured end-to-end: detection → remediation → post-detection are folded into a single Remediation cycle card with a parent header showing the overall outcome (Compliant, Remediated successfully, Non-compliant after remediation, Remediation script failed, …). A live "running" indicator appears the moment a script starts, ahead of the consolidated final result. Phase-aware semantics avoid counting a non-compliant detection that was successfully remediated as a "failed" event, and a two-stage emission closes the timing gap on short Autopilot sessions.
* **SLA notification spam fix** — SLA throttling moved from an in-memory cooldown to a persistent per-tenant status row with CAS-guarded at-most-once semantics. AppInstall SLA now actually evaluates against the same ISO-week window as the dashboard, and a single `sla_resolved` notification fires when a breach clears. Configurable cooldown via Global Settings.
* **Dashboard stats overhaul** — The five dashboard cards (Active Sessions, Success Rate, Avg. Duration, Total Today, Failed Today) are now server-aggregated over the full 7-day window instead of derived from the currently-paginated client list, so they no longer drift as the session table grows. Active Sessions is pinned to In-Progress only; Pending (WhiteGlove waiting for power-on) and Stalled remain visible via list filters. Stats reset cleanly on scope changes.
* **Agent V2 robustness** — Several correctness and recovery fixes: Classic enrollments with Hello disabled no longer deadlock at the AwaitingHello stage (EspExiting now routes through the Hello-disabled fast-path); AwaitingHello is gated on AccountSetup _truly_ succeeding instead of the page-handoff event; new desktop-detector liveness signals (started / first-poll / no-candidate) help distinguish a dead post-reboot agent from broken DAD wiring; four new `agent_shutting_down` emit paths (Ctrl+C, process exit, unhandled exception, runtime host exit) close gaps that previously left sessions without a shutdown breadcrumb. Hot-path event dictionaries are pre-sized to skip ~3k Gen0 allocations per session.
* **Fail-soft runtime handoff & ASR resilience** — A Defender ASR rule blocking WMI process creation can no longer strand a freshly-bootstrapped device. The agent's `--install` now runs a two-tier fallback (WMI Win32_Process → `schtasks /Run` → next BootTrigger reboot) and treats a runtime-spawn failure as a warning, not a fatal error. The bootstrap script's process probe remains the canonical "did the agent come up" signal.
* **Intune dual-stack certificate selection** — On Intune-managed devices that have both the MDM device certificate and the newer MMP-C certificate installed side-by-side, the agent now exact-matches the MDM issuer (`CN=Microsoft Intune MDM Device CA`) before presenting it for mTLS, instead of letting the OS pick. Resolves a class of `SecureChannelFailure` / `ChainFailed` rejections that surfaced on newer Windows builds.
* **"Likely stuck" app-install hedge** — When ESP times out on the Apps subcategory, the in-flight app is no longer left silently frozen in the Installing bucket. It is now promoted as a hedged "likely stuck" entry with confidence _presumed_, so operators see which app the enrollment was waiting on at termination.
* **Notifications & UX polish** — Opt-in "enrollment started" webhook (Teams Legacy / Workflow / Slack) fires on fresh session registration and WhiteGlove Part 2 resume, default off so existing tenants don't get a surprise notification storm. The notification bell is now always visible. Sidebar usage telemetry (full / icons / hidden) is captured as a global App Insights property for understanding portal interaction patterns.
* **Bugfixes & polish** — Various fixes throughout the whole platform

## 2026-05-10 — Agent V2 rollout, MS-Graph pagination, SignalR push notifications, and security hardening

_Platform Update — 12:00 CET_

* **Agent V2 rollout (action recommended)** — The agent has been rebuilt on a new internal architecture (Decision Engine, lifecycle-anchor allowlist, Death-Rattle recovery for crashed runs) for more reliable session detection and stricter completion logic. The Intune bootstrapper script (`Install-AutopilotMonitor.ps1`) was updated and now verifies the runtime process started after launch. Replace the script in your Intune tenant with the latest version from the repository.
* **Submit Logs page** — A new Submit Logs page lets you send diagnostic files to the Autopilot Monitor team without an associated session, useful for issues caught outside of an active enrollment.
* **Real-time notifications via SignalR push** — The notification bell now updates purely from SignalR events (one fetch on mount, re-fetch on reconnect) instead of polling every 60 seconds, with audience-aware delivery for tenant admins vs members.
* **MS-Graph style pagination** — All large list endpoints (sessions, events, audit, ops events, reports) now support opt-in `nextLink` pagination with HMAC-bound continuation tokens. Dashboard, Fleet Health, and MCP tools have been migrated to the new model.
* **MCP server** — A new `get_resource` tool, leaner `get_session_summary` payloads, and a security & capability audit covering OAuth proxy hardening, access-guard tightening, and pagination of the last three legacy diagnostic tools.
* **Backend security hardening** — Client-certificate chain trust is now pinned to the embedded Intune root certificate, and query-tenant guards moved to centralized middleware.
* **Web hardening & portal split** — Tightened Content Security Policy, hardened `/go` bootstrap error handling, and a host-based middleware now cleanly separates the public site from the authenticated portal.
* **Web performance** — In-flight request collapser (`dedupedAuthFetch`) eliminates duplicate parallel fetches, the dashboard load path was refactored
* **Delivery Optimization** — Download breakdowns now include Microsoft Connected Cache (MCC) and LinkLocal sources, applied symmetrically across all layers (collector, telemetry, dashboard).
* **WhiteGlove improvements** — Timeline now splits Part 1 / Part 2 at the Part-2 `agent_started` event (not the legacy `whiteglove_resumed`), Part-2 architectural cleanup landed on the agent side, summary dialog is suppressed on Part 1, and Modern Deployment EventID 1010 was added to the harmless-noise default list.
* **Bugfixes & polish** — `TenantIdResolver` fallbacks for non-Type-6 enrollments and Hybrid Join (CloudDomainJoin + MS-Organization-Access cert), event-driven TenantId wait via RegistryWatcher, software-inventory now accepts AAD/MSA SIDs and emits real registry paths, full-width dashboard toggle with `?span=` URL persistence, SLA violator link routes correctly, and many smaller fixes across pagination, audit fetching, and submit-diag sidebar entries.

## 2026-04-16 — Completion state machine, SLA & App Health dashboards, Ops Alerts, and Device Preparation groundwork

_Platform Update — 12:00 CET_

* **Session completion state machine** — The agent soon uses a dedicated `CompletionStateMachine` that combines multiple signals (ESP final exit, Hello, Desktop arrival) to decide when an enrollment is truly done. This fixes several cases where WhiteGlove and Hybrid Join sessions were misclassified or never marked complete.
* **SLA tracking dashboard** — New SLA monitoring page with per-tenant configuration and notification support when SLAs are breached.
* **App Health dashboard** — New global view of app deployment health with scoped drill-downs and a configurable column picker.
* **Ops Events & Ops Alerts** — Operational event log plus admin alerts for backend health, blob storage, runaway sessions, and excessive event counts per session.
* **Agent emergency / distress channel** — A separate low-overhead channel so the agent can still report critical errors when the normal telemetry path is impaired.
* **Enhanced analyze rule engine** — New `in` / `not_in` compare operators, `MarkSessionAsFailed` action, template variables, per-rule stats card, and a new ESP certificate-error analyze rule (`ANALYZE-ESP-002`).
* **Delivery Optimization** — OS-level DO collector, P2P totals in download progress, and DO usage stats in the geographic drill-down.
* **Vulnerability matching improvements** — Fuzzy (Jaro-Winkler) CPE matching, confidence levels, data freshness indicators, CVE mapping column in the vulnerability report, and WhiteGlove sessions now also get a vulnerability report.
* **Device Preparation (WDP v2) groundwork** — The agent now distinguishes Classic vs v2 Autopilot flow, and a device-association validator was added on the backend. Device Preparation support is still in active validation.
* **IME version history** — Intune Management Extension version history is tracked and surfaced via MCP; agents running on outdated IME versions trigger an alert.
* **Known Issues page** — Dedicated docs page for ongoing issues (replaces the inline list that used to live in this changelog).
* **MCP server** — Stateless endpoint, tools split into domain modules, new ops-events tool, tool-call telemetry, improved semantic + keyword search, and an integration test suite.
* **Security hardening** — Centralized tenant-isolation middleware, OData sanitizer, hardened agent config endpoint, cross-tenant fallback fixes, session-aware auto-unblock, and additional request-size / integrity guards on the self-update path.
* **Web performance & refactor** — Lazy session loading, response compression, more parallel fetches, and a large internal restructuring of the web app into hooks and utils for easier maintenance.
* **Bugfixes & UX polish** — Quick search, bootstrap scripts, webhook notifications, WhiteGlove timeline rendering, phase-timeline regressions, report upload size, summary dialog launch fallback, NTP / timezone defaults, and many more small fixes.

## 2026-03-30 — Updated bootstrapper script, agent crash detection, and quick search

_Platform Update — 12:00 CET_

* **Updated bootstrapper script (action recommended)** — The bootstrapper script (`Install-AutopilotMonitor.ps1`) now uses SHA-256 integrity verification for agent downloads instead of MD5. If you deployed the script via Intune, it is recommended to replace it with the latest version from the repository for improved security.
* **Agent crash detection** — The agent now detects and reports unexpected crashes with automatic recovery. Platform-level metrics (CPU, memory, disk) are collected alongside enrollment events for better diagnostics.
* **Global quick search** — A fuzzy search across sessions, devices, and users is now available from the navigation bar for fast lookups.
* **Rate limiting** — Per-user request rate limiting protects the backend from excessive API usage.
* **Bugfixes** — Vulnerability report rescan persistence, orphaned session handling, timezone parsing, and NTP clock-skew warnings improved.

## 2026-03-26 — Software inventory & vulnerability analysis, new agent signals, and settings overhaul

_Platform Update — 12:00 CET_

* **Software Inventory & Vulnerability Analysis** — The agent now discovers installed software across Registry, WMI, AppX/MSIX, and per-user sources and correlates it against NVD and CISA KEV databases. The dashboard shows a vulnerability report with CVSS scores and severity levels. Includes 240+ curated CPE mappings and strict AppX whitelist filtering.
* **SecureBoot & time sync** — The agent collects SecureBoot certificate details (with a new analyze rule), auto-detects the timezone, and checks NTP offset to catch time-related enrollment failures.
* **Security hardening** — Request size limits on all submission endpoints and symlink detection in diagnostic paths guard against DoS and path-traversal attacks.
* **Settings reorganization** — The sidebar now uses expandable sections for a cleaner navigation. Tenant settings were restructured and consolidated.
* **OOBE Config viewer** — A modal dialog decodes the OOBE configuration bitmask, showing each bit flag with description and confidence level, and detects the enrollment profile type.
* **FAQ page** — New Docs section covering supported scenarios, deployment, agent capabilities, and troubleshooting.

## 2026-03-19 — Navigation overhaul, session architecture, new agent signals, and community rules

_Platform Update — 12:00 CET_

* **Unified sidebar** — The entire navigation has been redesigned with a global sidebar. The old top nav is gone; settings and admin areas now have their own sidebar sections. Mobile layout also reworked.
* **Session index table** — Session storage has been fundamentally re-architected for better scalability and reliability.
* **New agent signals** — The agent now reports `agent_shutdown` (clean shutdown), `hardware_spec` (hardware inventory at enrollment), network interface changes, and clock skew deviations for better diagnostics.
* **Self-deploying mode detection** — The agent now automatically detects self-deploying scenarios and tracks the enrollment finalization process with dedicated events.
* **Notification providers** — The webhook notification system now supports three providers: Teams Legacy, Teams Workflow, and Slack — selectable per tenant.
* **Community rules** — A community rule set for gather and analyze rules has been added. Rules now have a JSON view, severity override, and centralized guardrails. New local admin analyze rule included.
* **Geographic drill-down** — The geographic performance view now supports drill-down to region and country level.
* **Mark as success** — Sessions can now be manually marked as successful, e.g. after manually resolved enrollments.
* **Feedback system** — An integrated feedback system with admin management allows direct feedback from within the portal.
* **Tenant settings UX** — The central save button in tenant settings has been replaced with individual section save buttons. A new Unrestricted Mode option disables most guardrails per tenant request.
* **Docs expanded** — New general documentation section, IME pattern explanation, and a public sites sidebar added.
* **Backend reliability** — Improved cache invalidation and retry logic for transient errors.

## 2026-03-10 — Security architecture, session timeline improvements, and new agent capabilities

_Platform Update — 12:00 CET_

* **Role-based access control** — Admin and Operator roles with role management in Settings. API authorization and policy enforcement middleware ensure proper access control across all endpoints.
* **Agent self-update** — Agents can now update themselves automatically, ensuring outdated versions in the field get replaced without manual intervention.
* **Bootstrap sessions** — New bootstrap session flow with explicit token enablement for initial device onboarding. (support for bootstrap tokens enabled by request)
* **Raw event timeline** — A new raw view of the event timeline with full search support, useful for deep-dive troubleshooting.
* **Enrollment summary dialog** — Optional summary dialog shown at the end of enrollment, with event timeline search and clickable phases in the phase tracker.
* **Original ESP tracking** — The agent now tracks the original ESP provisioning status to catch non-IME errors such as certificate failures.
* **Analyze & gather rules** — Added negative compare operators for analyze rules, XML and JSON gather options, and a built-in "old OS version" warning rule.
* **Email notifications** — Email notification (Welcome and instructions) for Joining the Private Preview.
* **Agent version management** — Block specific agent versions from connecting, along with expanded data retention configuration options.
* **Install progress** — The agent install progress page now shows download and install phases with elapsed time.
* **TPM info collection** — TPM details are now collected at enrollment time for improved hardware diagnostics.
* **Firewall compatibility** — The agent now sends a dedicated User-Agent header to simplify firewall allowlisting.

## 2026-03-01 — Ongoing improvements to Pre-Provisioning support (still testing)

_Pre-Provisioning (WhiteGlove) — 16:00 CET_

I'm continuously improving support for Pre-Provisioning (White Glove) scenarios. The session timeline should now better reflect the provisioning process better, and I'm working on improving the accuracy of event categorization and timing for these sessions. If you are using Pre-Provisioning and notice any discrepancies in the timeline or data, please share your Feedback with me via GitHub Issues. Your feedback is invaluable in helping me enhance support for these scenarios. Expect a "Report Session" button in the timeline view soon to make sharing feedback and logs easier!

## 2026-02-27 — Configurable Diagnostic Package, Gather Rule Examples, Updated Docs

_Features — 21:38 CET_

The configurable diagnostic package allows for more flexible data collection and analysis. Gather rule examples have been added to help users understand how to create their own rules. Documentation has been updated to reflect these changes and provide guidance on using the features.

## 2026-02-27 — First implementation of Pre-Provisioning support incl. session timeline visualization

_Architecture — 14:38 CET_

The session timeline now also supports sessions that started with Pre-Provisioning (aka White Glove) — including the provisioning process itself. This is a first implementation and only tested with a very basic scenario, so if you use Pre-Provisioning and see anything that looks off in the timeline, please check the logs and share them via GitHub Issues.

## 2026-02-26 — Reworked real-time event delivery and session timeline processing

_Architecture — 10:15 CET_

The way live session events reach the dashboard timeline was fundamentally reworked. This should make the timeline more reliable and accurate.
