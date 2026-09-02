---
type: FAQ
tags: [faq, troubleshooting]
description: Frequently asked questions about setup, the agent, the portal, and data handling.
---

# FAQ

## General

<details>

<summary>What is Autopilot Monitor?</summary>

Autopilot Monitor gives IT admins real-time visibility into Windows Autopilot enrollments. A lightweight, temporary agent runs on devices during enrollment and streams events to a web portal where you can watch progress live, diagnose failures with automated rule analysis, and review historical sessions. See [How Autopilot Monitor Works](../getting-started/how-it-works.md).

</details>

<details>

<summary>Which Autopilot scenarios are supported?</summary>

**User-Driven** and **Pre-Provisioned** (White Glove) Windows Autopilot flows are fully supported — for **Entra joined** and **Hybrid joined** devices alike — as are self-deploying/kiosk profiles. **Autopilot Device Preparation** is supported — including [device association](../getting-started/autopilot-device-preparation.md#device-association) as a validation method — and uses its own agent delivery channel — see [Autopilot Device Preparation](../getting-started/autopilot-device-preparation.md). **Windows 365 Cloud PCs** are supported as an opt-in per tenant — see the next question. The full overview: [supported enrollment scenarios](../getting-started/how-it-works.md#supported-enrollment-scenarios).

</details>

<details>

<summary>Does Autopilot Monitor support Windows 365 Cloud PCs?</summary>

Yes, as an opt-in per tenant. Cloud PCs are provisioned headless and are never Autopilot-registered, so they need their own validation method and a small deployment addition: enable **Windows 365 Cloud PC Validation** in Settings (requires the optional `CloudPC.Read.All` add-on permission) and assign the bootstrapper script to a group that includes your Cloud PCs. Monitoring then covers the first-connect enrollment (Account Setup) of the assigned user. The full setup is on [Windows 365 Cloud PCs](../getting-started/windows-365-cloud-pcs.md).

</details>

<details>

<summary>Is Autopilot Monitor free?</summary>

Yes — the **Community plan is free and stays free**, and it is publicly available: sign in with your work account and your tenant is activated after a short activation step (see [Requirements & Access](../getting-started/requirements-and-access.md)). A commercial **Pro plan** is coming later for organizations that need more — see [Plans](../plans.md).

</details>

<details>

<summary>Where is my data stored?</summary>

**In Germany.** All customer data and all compute that touches it run in Azure **Germany West Central**: Azure Functions, Table Storage, Blob Storage, queues, SignalR, the MCP container app, and Application Insights / Log Analytics. Your data is not replicated to another region.

One component sits outside that region — the Static Web App in West Europe that serves the portal front-end. It delivers the HTML, JavaScript, and images that make up the portal and **stores no customer data**; every request for actual data goes from your browser to the API in Germany West Central.

Data retention is [configurable per tenant](../reference/settings.md#data-management) (default 90 days), and a tenant can [offboard](../reference/settings.md#danger-zone) — irreversibly deleting all its data — at any time. Diagnostics packages can be kept entirely in **your own** Azure Blob Storage. For the full data-residency answer, see [Trust & Security](../trust/security-faq.md).

</details>

## Setup & Agent

<details>

<summary>Why are there two "Autopilot Monitor" applications in my Entra tenant?</summary>

Autopilot Monitor is moving to a new multi-tenant app registration. The application with ID `886ab5e2-6144-442c-80cc-9b28e0667731` is the new one; `1a400946-62c1-4ab4-aa37-f730ac89704d` is the previous registration your tenant consented to at onboarding. Migrating is a one-time admin consent under **Settings → Autopilot Validation**, and after migrating the previous application can be deleted from your tenant. See [App Registration Migration](app-registration-migration.md).

</details>

<details>

<summary>How do I deploy the agent?</summary>

Via an Intune platform script — the [Deploy the Agent](../getting-started/deploy-the-agent.md) guide covers it step by step, including the safety guards and a dry-run tester. For **Autopilot Device Preparation** enrollments the agent is instead delivered as a small MSI line-of-business app, because the platform script would arrive after the app phase there — see [Autopilot Device Preparation](../getting-started/autopilot-device-preparation.md).

</details>

<details>

<summary>Do I need to update the bootstrap script in Intune?</summary>

Yes — check it from time to time. The agent itself always stays current (the script downloads the latest release fresh on every device), but the bootstrap script is a static copy frozen at the moment you uploaded it to Intune — and the script keeps evolving: new install guards, exemptions for new Windows capabilities, and endpoint changes ship regularly.

An outdated copy still fails soft and never breaks an enrollment, but it can **silently skip devices that a current version would monitor**. Example: **Windows Backup for Organizations** restores a user profile during OOBE — older script versions read that profile as "device already in productive use" and skip the install, so the enrollment never shows up in the portal.

To check: compare the current version badge on [Deploy the Agent](../getting-started/deploy-the-agent.md#1-download-the-bootstrapper-script) with your Intune copy (the version is in the script header, and every run logs `Bootstrap script version`). Updates worth acting on are flagged **action recommended** in the [Platform Changelog](../changelog/platform-changelog.md). Updating takes under a minute: upload the new `.ps1` over your existing platform script in Intune.

</details>

<details>

<summary>Does the agent run permanently on the device?</summary>

No. The agent only exists during the enrollment window: it self-destructs after completion, stops at its 6-hour maximum lifetime, and — as an unconditional backstop — removes itself 48 hours after installation no matter what. It never runs as a persistent background service. See [Agent Lifecycle & Security](../concepts/agent-lifecycle-and-security.md).

</details>

<details>

<summary>What data does the agent collect?</summary>

Enrollment telemetry only: ESP phases, app and script installations, IME log entries, device information, performance snapshots, and security posture. It does **not** collect personal user data, user files, or browsing history. Optional collectors (geo location, software inventory, diagnostics package) are controlled by your tenant settings. See [the full list](../concepts/agent-lifecycle-and-security.md#what-data-is-collected).

</details>

<details>

<summary>How does the agent authenticate?</summary>

With the device's **MDM client certificate** issued by Intune, over TLS. The backend additionally validates each device against your Intune tenant (Autopilot registration, device association, corporate identifiers, or Cloud PC inventory) before accepting any data — only devices under your management can send telemetry. See [Authentication](../concepts/agent-lifecycle-and-security.md#authentication).

</details>

## Portal & Features

<details>

<summary>What are Gather Rules and Analyze Rules?</summary>

**Gather Rules** collect custom evidence from devices during enrollment (registry, WMI, files, allow-listed commands). **Analyze Rules** automatically evaluate every session and surface findings with explanations and remediation steps. They work best together — see [How Rules Work Together](../rules/overview.md).

</details>

<details>

<summary>Can an AI build custom rules for me?</summary>

Yes — with [AI Integration (MCP)](../integrations/ai-integration-mcp.md) connected, your assistant has the complete rule schemas and allowlists, can validate a draft, and can **dry-run an analyze rule against a real session from your tenant** before anything goes live. See [AI-Assisted Rule Authoring](../rules/ai-assisted-rule-authoring.md).

</details>

<details>

<summary>Can I export or download diagnostics data?</summary>

Yes — when the [Diagnostics Package](diagnostics-and-log-collection.md) feature is enabled, each session detail view offers a download of the collected log bundle. Session reports and timeline exports are available from the session's **Report** action.

</details>

<details>

<summary>Can I follow a Windows Autopilot enrollment live?</summary>

Yes. Every enrolling device appears on the [dashboard](../portal-guide/dashboard-and-sessions.md) as it starts, and its [session](../portal-guide/session-details-and-diagnosis.md) shows ESP phases, app downloads and installs, reboots, errors, and performance snapshots as they happen — no refreshing, no touching the device.

</details>

<details>

<summary>Why did an enrollment fail, and how do I find out without the device?</summary>

Open the session: [analyze rules](../rules/analyze-rules/README.md) have already flagged app install error codes, detection-rule failures that break the ESP, blocking-app timeouts, download and proxy failures, TPM attestation and MDM enrollment error codes, hybrid join problems, failed Windows updates, and low disk space or battery (full list: [built-in rules](../rules/analyze-rules/built-in-rules.md)). [Guided diagnosis](../portal-guide/session-details-and-diagnosis.md#guided-diagnosis) names the primary suspect with a copyable quick fix; [diagnostics collection](diagnostics-and-log-collection.md) brings the agent and IME logs to you.

</details>

<details>

<summary>Are failed enrollments analyzed automatically?</summary>

Yes. Dozens of built-in, community-maintained rules evaluate every session automatically and report confidence-scored findings with remediation steps. You can adapt [template rules](../rules/analyze-rules/template-rules.md), [write your own](../rules/analyze-rules/cookbook.md), and add [IME log patterns](../rules/ime-log-patterns.md); [regression detection](../rules/analyze-rules/README.md#regression-detection) tells you when a rule starts firing more often than usual.

</details>

<details>

<summary>How do I get alerted when an enrollment fails?</summary>

Configure [notifications](../integrations/notifications.md) for Microsoft Teams, Slack, Discord, or a generic JSON webhook and pick the triggers: start, success, or failure. The same channel carries SLA breach and resolution alerts, consecutive-failure alerts, and hardware rejection notices.

</details>

<details>

<summary>Is there reporting on success rate, enrollment duration, and failing apps?</summary>

Yes. [Fleet Health](../portal-guide/fleet-health.md) shows the success rate, average enrollment time, a daily timeline, top failure reasons, the slowest and most-failing models and apps, and a first-time-right rate. [SLA Compliance](../portal-guide/sla-compliance.md) reports against your own targets; [Geographic Performance](../portal-guide/geographic-performance.md) compares sites.

</details>

<details>

<summary>Why does enrollment take so long, and which app is slowing it down?</summary>

Every finished session gets a [time attribution](../portal-guide/session-details-and-diagnosis.md#time-attribution) bar that shows where the minutes went and which apps blocked the ESP. [Fleet Health](../portal-guide/fleet-health.md#time-attribution) aggregates the same data per enrollment class and ranks the apps that cost the most time.

</details>

<details>

<summary>Can end users or field technicians check a device's status without portal access?</summary>

Yes. The [Progress Portal](../portal-guide/progress-portal.md) shows a device's enrollment status by serial number — status, progress bar, steps, and what is currently downloading or installing — live and strictly read-only. No portal role is needed; the serial number is the access key.

</details>

## Troubleshooting

<details>

<summary>The agent is deployed but I don't see any sessions.</summary>

Work through the checklist in [Common Problems → No sessions appear](common-problems.md#no-sessions-appear-in-the-portal).

</details>

<details>

<summary>A session shows "In Progress" but the enrollment already finished.</summary>

The completion signal was missed — for example the device rebooted before the agent could detect the final state. The session resolves automatically: after the configured session timeout (default 5 hours) the backend reclassifies it from its evidence to **Awaiting User** or **Incomplete** (a timeout is no longer treated as a failure), and reconciles it to **Succeeded** if a genuine completion later arrives. You can also mark it succeeded/failed manually via [Admin Mode](../concepts/roles-and-permissions.md#admin-mode). See [Sessions & Statuses → Timeouts](../concepts/sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions).

</details>

<details>

<summary>Where are the agent's own log files?</summary>

`%ProgramData%\AutopilotMonitor\Logs` on the device — startup, event collection, and backend communication are all logged there. Log verbosity is a [tenant setting](../reference/settings.md#agent-parameters). Note the logs are removed by self-destruct unless **Keep Log File** is enabled.

</details>
