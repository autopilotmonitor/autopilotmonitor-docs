---
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

**User-Driven** and **Pre-Provisioned** (White Glove) Windows Autopilot flows are fully supported, as are self-deploying/kiosk profiles. **Autopilot Device Preparation** is supported in early testing.

</details>

<details>

<summary>Is Autopilot Monitor free?</summary>

Yes — the **Community Edition is free and stays free**. It is currently in Private Preview (access after approval, see [Requirements & Access](../getting-started/requirements-and-access.md)). A commercial **Enterprise Edition** is coming later for organizations that need more — see [Editions](../editions.md).

</details>

<details>

<summary>Where is my data stored?</summary>

Session data is stored in Azure, **West Europe** region (Azure Functions + Azure Table Storage). Data retention is [configurable per tenant](../reference/settings.md#data-management) (default 90 days), and a tenant can [offboard](../reference/settings.md#danger-zone) — irreversibly deleting all its data — at any time. Diagnostics packages can be kept entirely in **your own** Azure Blob Storage.

</details>

## Setup & Agent

<details>

<summary>How do I deploy the agent?</summary>

Via an Intune platform script — the [Deploy the Agent](../getting-started/deploy-the-agent.md) guide covers it step by step, including the safety guards and a dry-run tester.

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

The completion signal was missed — for example the device rebooted before the agent could detect the final state. The session resolves automatically: the backend marks stalled sessions **Failed – Timed Out** after the configured session timeout (default 5 hours). You can also mark it succeeded/failed manually via [Admin Mode](../concepts/roles-and-permissions.md#admin-mode). See [Sessions & Statuses → Timeouts](../concepts/sessions-and-statuses.md#timeouts--what-happens-to-stuck-sessions).

</details>

<details>

<summary>Where are the agent's own log files?</summary>

`%ProgramData%\AutopilotMonitor\Logs` on the device — startup, event collection, and backend communication are all logged there. Log verbosity is a [tenant setting](../reference/settings.md#agent-parameters). Note the logs are removed by self-destruct unless **Keep Log File** is enabled.

</details>
