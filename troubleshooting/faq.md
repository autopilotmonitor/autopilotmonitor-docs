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

**User-Driven** and **Pre-Provisioned** (White Glove) Windows Autopilot flows are fully supported, as are self-deploying/kiosk profiles. **Autopilot Device Preparation** is supported in early testing. **Windows 365 Cloud PCs** are supported as an opt-in per tenant — see the next question.

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

<summary>How do I deploy the agent?</summary>

Via an Intune platform script — the [Deploy the Agent](../getting-started/deploy-the-agent.md) guide covers it step by step, including the safety guards and a dry-run tester.

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

With the device's **MDM client certificate** issued by Intune, over TLS. The backend additionally validates each device against your Intune tenant (Autopilot registration or corporate identifiers) before accepting any data — only devices under your management can send telemetry. See [Authentication](../concepts/agent-lifecycle-and-security.md#authentication).

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
