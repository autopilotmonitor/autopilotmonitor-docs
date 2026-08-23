---
type: Changelog
tags: [platform, changelog]
description: >-
  Significant platform changes — portal, backend, and data-flow updates,
  newest first.
---

# Platform Changelog

This changelog tracks significant platform changes — architecture updates, data flow changes, and anything else that might briefly affect the UI or monitoring data. If something looks off, check here first. A recent entry might explain it.

Found a bug or want to give feedback? [Open a GitHub Issue](https://github.com/okieselbach/Autopilot-Monitor/issues) — it helps more than you might think.

## August 2026

* **Diagnostics package contents are now visible** — Settings → Agent → Diagnostics Package lists the built-in collection and the platform-wide paths read-only above your own entries, one line per path. See [Diagnostics & Log Collection](../troubleshooting/diagnostics-and-log-collection.md#the-diagnostics-package).
* **RealmJoin Watcher and Keep Awake moved to Agent Settings** — Both toggles now live under Settings → Agent → Agent Settings instead of Agent Analyzers. See [Settings Reference](../reference/settings.md#agent).
* **Discord notifications** — Discord is now available as a notification provider; enrollment alerts arrive as embeds in your Discord channel. See [Notifications](../integrations/notifications.md#discord).
* **Signed webhook requests** — Generic JSON webhooks can now carry an HMAC signature so your endpoint can verify each request really came from Autopilot Monitor. See [Verifying signed requests](../integrations/notifications.md#verifying-signed-requests).
* **App install times now measure the actual install attempt** — App durations now measure the final install attempt instead of the span since the app was first observed; multi-pass apps carry a ×2 marker, and download time is reported separately. Values recorded before the change keep the old measurement, so averages step down visibly around the changeover.
* **Analyze rules can now run during the enrollment** — Rules can declare additional **evaluation triggers** — at the WhiteGlove seal or when specific events arrive — instead of only running at enrollment end. Early findings appear live as **Preliminary** and are confirmed or resolved by the final analysis. See [Concepts → Evaluation triggers](../rules/analyze-rules/concepts.md#evaluation-triggers-when-a-rule-runs).
* **App version duration regression alerts** — When a newly rolled-out app version installs markedly slower than its predecessor, the app's detail page shows a regression banner and tenant admins get a one-time notification. Version charts now include median install duration by version. See [Software](../portal-guide/software-inventory-and-vulnerabilities.md#per-app-deep-dive).
* **Fleet context on analysis findings** — Each finding on a session now shows in how many enrollments the rule fired over the last 14 days; **View sessions** opens the dashboard filtered to those enrollments. See [Session Details](../portal-guide/session-details-and-diagnosis.md#session-detail).
* **Gather rules: guardrail-aware enabling and clearer attribution** — WMI gather rules can now select specific properties of an allow-listed class (requires the latest agent). Custom rules whose target is not allow-listed can no longer be enabled and show a **Blocked on devices** badge; new custom rules record the creating admin as **Author**.
* **Windows 365 Cloud PC support** — Cloud PC first-connect enrollments can now be monitored via the new opt-in **Windows 365 Cloud PC Validation** check — enable it in Settings and deploy the latest bootstrapper script. Rejected Cloud PCs in *Devices Not Registered* now carry a Cloud PC badge.
* **AI-assisted rule authoring** — An AI assistant connected via MCP can draft an analyze or gather rule, validate it against the real schemas and guardrails, and dry-run it against one of your sessions — without changing anything in your tenant. See [AI-Assisted Rule Authoring](../rules/ai-assisted-rule-authoring.md).
* **Test log patterns with the agent's real engine** — Log-parser patterns can now be tested against pasted sample log lines using the exact matching engine the agent runs on the device. See the [worked example](../rules/ai-assisted-rule-authoring.md#example-from-a-log-file-to-timeline-events).
* **Fixed: self-deploying (kiosk) enrollments shown as Awaiting User or Incomplete** — A self-deploying device whose agent goes quiet after Device Setup is now reconciled to Succeeded instead of waiting for a user that never signs in; affected sessions have been corrected. See [Sessions & Statuses](../concepts/sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions).
* **Fixed: enrollment durations inflated by time-zone-skewed log timestamps** — Session start times are now guarded against stray log timestamps from a different time zone; affected sessions have been corrected.
* **Fixed: Pre-Provisioning completion webhook fired twice** — It now fires once. Notification titles also switched from emojis to neutral status indicators.
* **Fixed: disabling device validation did not stick** — Turning off Autopilot or Corporate Identifier validation in Settings now actually saves.
* **Mobile & UX polish** — Public-site navigation now works on phones, the dashboard session list gained a **Load all** option, plus pagination, dialog, and dark-mode fixes.

## Late July 2026

* **Publicly available — sign-in with tenant activation** — Autopilot Monitor left the invite-only phase: any organization can sign in with a work account; new tenants are activated automatically after a short activation step — no access request needed. See [Requirements & Access](../getting-started/requirements-and-access.md).
* **Updated bootstrapper script (action recommended)** — The bootstrapper script (`Install-AutopilotMonitor.ps1`) now downloads the agent from the dedicated distribution endpoint `download.autopilotmonitor.com` and supports enrollments that restore settings via **Windows Backup for Organizations** (previously such devices were skipped as "already in use"). If you deployed it via Intune, replace it with the latest version from the repository — see [Deploy the Agent](../getting-started/deploy-the-agent.md). Existing deployments keep working.
* **Time attribution — where enrollment time goes** — Finished enrollments now show a breakdown of where the time went — device preparation, apps, identity & Hello, user ESP, desktop handoff — including reboot time and the ESP-blocking apps on the critical path. Fleet Health shows the same split as fleet-wide medians. See [Fleet Health](../portal-guide/fleet-health.md#time-attribution) and [Session Details](../portal-guide/session-details-and-diagnosis.md#time-attribution).
* **First-time-right — wipe-and-retry is now visible** — Enrollment attempts of the same device are grouped into a [device journey](../concepts/sessions-and-statuses.md#device-journeys--attempts): Fleet Health shows how many devices needed a second run, the weekly trend, and the repeat devices with their last failure reason. See [Fleet Health](../portal-guide/fleet-health.md#first-time-right).
* **Device history on the session** — A session whose device enrolled before now opens with *"Attempt N for this device"* and an expandable list of that device's previous enrollments.
* **Rule regression detection** — When an analyze rule starts firing far above its own 28-day baseline, its card shows a **↑ Regression** badge and tenant admins get a notification — including the OS build, model, or agent version the affected sessions concentrate on, where one exists. See [Analyze Rules](../rules/analyze-rules/README.md#regression-detection).
* **On-demand log collection** — **Collect Logs** on a running session asks the agent to build and upload a diagnostics package right away — no waiting for the enrollment to finish; a package typically arrives within a minute. See [Session Details](../portal-guide/session-details-and-diagnosis.md#on-demand-log-collection).
* **App durations count real installs only** — Apps that Intune reported as *skipped* no longer count as 0-second installs; duration stats and the slowest-apps ranking cover measured installs only, with skipped counts shown separately.
* **Rates count finished work only** — Success rate and app failure rate are now computed over finished enrollments and installs, so sessions still in progress no longer skew the numbers.
* **Install rows say where they come from** — Rows in Install Progress that are not plain Intune app installs now carry a **Click-to-Run** or **RealmJoin** pill, so an app and the installer it triggers no longer look like a duplicate.
* **Incompatible-TPM devices are surfaced** — Devices whose TPM cannot sign with RSA-PSS (and can therefore never be monitored on Windows 11 25H2+) now appear in the Hardware Whitelist with remediation guidance, plus a one-time notification.
* **Gather rules: phase-aware and quieter** — Rules can now collect once at a phase's start or end, be scoped to specific enrollment phases, and — for interval rules — emit only when the collected result actually changes (the default for new rules). See [Gather Rules](../rules/gather-rules.md).
* **Fixed: toggling a custom gather rule no longer clears its definition** — Toggles now only change the enabled state instead of wiping title, target, and parameters.
* **Fixed: portal could freeze on navigation** — Tabs no longer hang on a spinner for minutes after navigating; the hosting plan was also upgraded.
* **Portal recovers from deployments** — If a portal update ships while a tab is open, the app now recovers with a single automatic reload instead of a broken page.
* **Mobile & dark-mode polish** — Better readability for rule cards and phase timelines on small screens, tap-to-expand for truncated labels, and dark-mode contrast fixes.

## Early July 2026

* **Missing Autopilot profile is now flagged** — Devices that go through OOBE without their Autopilot deployment profile (not assigned, or the assignment hadn't propagated yet) are highlighted with a warning — across tenants, these enrollments fail far more often than normal ones.
* **Evidence for why the profile was missing** — The agent records Windows' own deployment-service verdict, the Autopilot diagnostic registry values, and a reachability check of the deployment service.
* **Device Validation shown on the session** — Session Info now shows which check admitted the device to the platform: Autopilot registration, Corporate Identifier, or Bootstrap token.
* **Pre-install errors are backfilled** — Autopilot errors logged before the agent was installed (e.g. TPM attestation retries that later succeeded) now appear in the session timeline, marked as backfilled.
* **New built-in rules for documented Autopilot known issues** — TPM attestation error codes, the hybrid-join 0x80004005 timeout (including which Windows update fixes it), and clock-skew warnings known to break attestation and ESP.
* **Two new session states — timeout is no longer a failure** — A silent session is now classified as Awaiting User (still waiting on the account phase) or Incomplete (expired, but not a failure) instead of always being marked Failed. See [Sessions & Statuses](../concepts/sessions-and-statuses.md).
* **Late completions now reconcile to Succeeded** — If a genuine completion signal arrives later, the session is upgraded to Succeeded even from Failed, Incomplete, or Awaiting User.
* **Honest Fleet Health** — Success Rate now excludes Incomplete sessions instead of counting them as failures; Incomplete gets its own stat card.
* **Windows Update during OOBE is now visible** — The agent detects a quality/cumulative update installing during enrollment and reports it, graded by two new analyze rules. See [Built-in Rules](../rules/analyze-rules/built-in-rules.md#device).
* **Self-maintaining rule lifecycle** — Gather rules now get the same automatic sunset/cleanup as analyze rules, and community rules are no longer mistaken for removed ones.

## June 2026

* **Software hub with self-service vulnerability exposure** — App installs, installed-software inventory, and CVE/KEV vulnerability exposure are now combined in one tabbed Software hub, with per-tenant Fleet Exposure available to tenant admins.
* **Faster Fleet Health on large fleets** — Fleet Health now loads from server-aggregated metrics instead of the browser, so it opens far faster on big fleets.
* **"Not registered" devices overview** — A new view lists devices rejected over the last 14 days because they weren't in the tenant's Autopilot or Corporate Identifier registry.
* **Richer session detail** — Session Info now shows Reboots, Enrollment Type, Join Type, and last-contact time.
* **Office & RealmJoin install rows** — Microsoft 365 Apps and RealmJoin packages now appear as their own rows in Install Progress alongside Intune apps.
* **Expanded MCP tools** — New software-inventory and app-install-metrics tools, richer audit-log filters, and role-aware tool access.
* **Clearer error reporting** — Event and error views now distinguish a detection failure from an install failure from a "likely stuck" enrollment.
* **Deep links survive re-authentication** — Opening a shared link in a new tab now returns you there after login instead of the dashboard.
* **Health dashboard MCP card** — The Health dashboard now shows MCP Server status alongside the SignalR quota card.
* **Device auto-block on runaway sessions** — Devices emitting an excessive number of session events can now be auto-blocked with one click.
* **CPU architecture** — Device hardware now reports CPU architecture (`x86` / `x64` / `ARM` / `ARM64`).
* **Security hardening** — Stricter header and query-filter validation, plus a patched dependency advisory.

## Mid May 2026

* **Safe session cascade-delete with restore window** — Session deletion is now crash-safe and includes a 33-day restore window.
* **Async tenant offboarding** — Tenant offboarding now runs as a background cascade with a progress countdown — you can close the tab while it finishes.
* **Optional Graph add-on — real script names** — Tenant admins can opt in to a scoped Graph permission so Intune Platform Script and Remediation names show in session timelines instead of bare IDs.
* **Full Intune Remediation lifecycle** — Detection, remediation, and post-detection are now shown as a single Remediation cycle card with a live "running" indicator.
* **SLA notification spam fix** — SLA breach notifications now fire exactly once per breach, with a configurable cooldown.
* **Dashboard stats overhaul** — Dashboard cards are now server-aggregated over the full 7-day window, so they no longer drift as the session table grows.
* **Agent V2 robustness** — Several completion and recovery fixes, including Hello-disabled enrollments no longer deadlocking.
* **Fail-soft runtime handoff** — A blocked WMI process creation (e.g. by Defender ASR) can no longer strand a freshly-bootstrapped device.
* **Intune dual-stack certificate fix** — On devices with both MDM and MMP-C certificates, the agent now picks the correct one for mTLS, resolving some certificate-related enrollment failures.
* **"Likely stuck" app-install hedge** — When ESP times out on Apps, the in-flight app is now shown as "likely stuck" instead of silently disappearing.
* **Notifications & UX polish** — Opt-in "enrollment started" webhook, an always-visible notification bell, and sidebar usage telemetry.
* **Bugfixes & polish** — Various fixes throughout the whole platform.

## Early May 2026

* **Agent V2 rollout (action recommended)** — The agent was rebuilt on a new internal architecture for more reliable session detection. Replace your Intune bootstrapper script with the latest version from the repository.
* **Submit Logs page** — Send diagnostic files to the Autopilot Monitor team without needing an active session.
* **Real-time notifications** — The notification bell now updates instantly via push instead of polling.
* **Graph-style pagination** — Large list endpoints (sessions, events, audit, reports) now support cursor-based pagination.
* **MCP server improvements** — New `get_resource` tool, leaner payloads, and a security hardening pass.
* **Backend security hardening** — Client-certificate trust is now pinned to the embedded Intune root certificate.
* **Web hardening** — Tightened Content Security Policy and a cleaner separation of the public site from the authenticated portal.
* **Web performance** — Duplicate parallel fetches are now collapsed, speeding up dashboard load.
* **Delivery Optimization** — Download breakdowns now include Microsoft Connected Cache (MCC) and LinkLocal sources.
* **WhiteGlove improvements** — Timeline now splits Part 1 / Part 2 at the correct event, and the summary dialog no longer shows after Part 1.
* **Bugfixes & polish** — Tenant ID resolution fallbacks, software-inventory fixes, a full-width dashboard option, and various smaller fixes.

## April 2026

* **Session completion state machine** — The agent now combines multiple signals (ESP exit, Hello, Desktop arrival) to decide when an enrollment is truly done, fixing several WhiteGlove and Hybrid Join misclassifications.
* **SLA tracking dashboard** — New SLA monitoring page with per-tenant configuration and breach notifications.
* **App Health dashboard** — New global view of app deployment health with scoped drill-downs.
* **Ops Events & Ops Alerts** — Operational event log plus admin alerts for backend health, blob storage, and runaway sessions.
* **Agent emergency / distress channel** — A separate low-overhead channel so the agent can still report critical errors when normal telemetry is impaired.
* **Enhanced analyze rule engine** — New compare operators, a mark-as-failed action, and template variables for rules.
* **Delivery Optimization** — OS-level download tracking with peer-to-peer totals.
* **Vulnerability matching improvements** — Fuzzy CPE matching, confidence levels, and WhiteGlove sessions now get vulnerability reports too.
* **Device Preparation groundwork** — The agent now distinguishes Classic vs. v2 Autopilot flow; full support is still in active validation.
* **IME version history** — Intune Management Extension version history is tracked, with alerts for outdated versions.
* **Known Issues page** — Dedicated docs page for ongoing issues.
* **MCP server** — Now a stateless endpoint with domain-organized tools and improved search.
* **Security hardening** — Centralized tenant-isolation middleware and additional request-integrity guards.
* **Web performance** — Lazy session loading, response compression, and an internal restructuring for maintainability.
* **Bugfixes & UX polish** — Quick search, bootstrap scripts, webhook notifications, and many smaller fixes.

## Late March 2026

* **Updated bootstrapper script (action recommended)** — The bootstrapper script (`Install-AutopilotMonitor.ps1`) now uses SHA-256 integrity verification for agent downloads instead of MD5. If you deployed it via Intune, replace it with the latest version from the repository.
* **Agent crash detection** — The agent now detects and reports unexpected crashes with automatic recovery, alongside platform-level metrics (CPU, memory, disk).
* **Global quick search** — A fuzzy search across sessions, devices, and users is now available from the navigation bar.
* **Rate limiting** — Per-user request rate limiting protects the backend from excessive API usage.
* **Bugfixes** — Vulnerability report rescan persistence, orphaned session handling, and NTP clock-skew warnings improved.
* **Software Inventory & Vulnerability Analysis** — The agent discovers installed software across Registry, WMI, AppX/MSIX, and per-user sources and correlates it against NVD and CISA KEV databases, shown as a vulnerability report with CVSS scores.
* **SecureBoot & time sync** — The agent collects SecureBoot certificate details, auto-detects the timezone, and checks NTP offset to catch time-related enrollment failures.
* **Security hardening** — Request size limits on submission endpoints and symlink detection in diagnostic paths.
* **Settings reorganization** — The sidebar now uses expandable sections; tenant settings were restructured and consolidated.
* **OOBE Config viewer** — A modal dialog decodes the OOBE configuration bitmask and detects the enrollment profile type.
* **FAQ page** — New Docs section covering supported scenarios, deployment, and troubleshooting.

## Mid March 2026

* **Unified sidebar** — The entire navigation has been redesigned with a global sidebar; mobile layout also reworked.
* **Session index table** — Session storage has been fundamentally re-architected for better scalability and reliability.
* **New agent signals** — The agent now reports clean shutdown, hardware inventory, network interface changes, and clock skew deviations.
* **Self-deploying mode detection** — The agent automatically detects self-deploying scenarios and tracks enrollment finalization with dedicated events.
* **Notification providers** — Webhook notifications now support Teams Legacy, Teams Workflow, and Slack — selectable per tenant.
* **Community rules** — A community rule set for gather and analyze rules has been added, with a JSON view and severity override.
* **Geographic drill-down** — The geographic performance view now supports drill-down to region and country level.
* **Mark as success** — Sessions can now be manually marked as successful, e.g. after manually resolved enrollments.
* **Feedback system** — An integrated feedback system allows direct feedback from within the portal.
* **Tenant settings UX** — Individual section save buttons replace the central save button; a new Unrestricted Mode option disables most guardrails per tenant request.
* **Docs expanded** — New general documentation section and IME pattern explanation.
* **Backend reliability** — Improved cache invalidation and retry logic for transient errors.

## Early March 2026

* **Role-based access control** — Admin and Operator roles with role management in Settings.
* **Agent self-update** — Agents can now update themselves automatically, replacing outdated versions in the field without manual intervention.
* **Bootstrap sessions** — New bootstrap session flow with explicit token enablement for initial device onboarding.
* **Raw event timeline** — A new raw view of the event timeline with full search support for deep-dive troubleshooting.
* **Enrollment summary dialog** — Optional summary dialog shown at the end of enrollment.
* **Original ESP tracking** — The agent tracks the original ESP provisioning status to catch non-IME errors such as certificate failures.
* **Analyze & gather rules** — Negative compare operators for analyze rules, XML/JSON gather options, and a built-in "old OS version" warning rule.
* **Email notifications** — Welcome and instructions email when a tenant joins the service.
* **Agent version management** — Block specific agent versions from connecting, plus expanded data-retention configuration.
* **Install progress** — The agent install progress page now shows download and install phases with elapsed time.
* **TPM info collection** — TPM details are now collected at enrollment time.
* **Firewall compatibility** — The agent sends a dedicated User-Agent header to simplify firewall allowlisting.
* **Pre-Provisioning (White Glove) improvements** — Ongoing accuracy improvements to the session timeline for Pre-Provisioning scenarios.

## Late February 2026

* **Configurable Diagnostic Package** — More flexible diagnostic data collection, example gather rules, and updated documentation.
* **Pre-Provisioning (White Glove) session timeline** — First implementation of session timeline support for Pre-Provisioning enrollments.
* **Real-time event delivery rework** — Reworked how live session events reach the dashboard timeline, for better reliability and accuracy.
