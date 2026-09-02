---
type: Overview
tags: [autopilot-monitor, introduction]
description: >-
  Real-time monitoring, diagnostics, and automated analysis for Windows
  Autopilot enrollments.
---

# Welcome

**Autopilot Monitor** gives IT admins and MSPs real-time visibility into Windows Autopilot enrollments. Without it, enrollment is a black box: when a device gets stuck at the Enrollment Status Page (ESP), the only built-in answer is "wait, then reset." Autopilot Monitor opens that box. A lightweight, temporary agent runs on the device during enrollment, streams every meaningful signal — ESP phases, Win32/LOB app installs, PowerShell scripts, policies, performance, security posture — live to a central portal, and removes itself when enrollment completes.

On top of that live timeline, **analyze rules** evaluate every session automatically — failed detections, dependency chains, disk exhaustion, certificate problems, unexpected local admins, and much more. Each finding comes with a confidence score, an explanation, and concrete remediation steps, so problems are flagged before your users ever notice. Use the built-in rules, adapt templates, or write your own.

{% hint style="info" %}
Autopilot Monitor is **publicly available** — the **Community plan** is free; sign in with your work account at [www.autopilotmonitor.com](https://www.autopilotmonitor.com) and your tenant is activated after a short activation step. See [Plans](plans.md) for what's available today and what's coming, and [www.autopilotmonitor.com/about](https://www.autopilotmonitor.com/about) for the product overview and the people behind it.
{% endhint %}

## The questions you have during a rollout

The short version of what Autopilot Monitor can do, phrased the way the questions come up when devices are enrolling. Longer answers are one click away.

### Can I follow a Windows Autopilot enrollment live?

Yes. Once the agent is [deployed via Intune](getting-started/deploy-the-agent.md), every enrolling device appears on the [dashboard](portal-guide/dashboard-and-sessions.md) as it starts, and its [session](portal-guide/session-details-and-diagnosis.md) shows ESP phases, app downloads and installs, reboots, errors, and performance snapshots as they happen — no refreshing, no touching the device. Start with [Your First Monitored Enrollment](getting-started/your-first-monitored-enrollment.md).

### Why did this enrollment fail, and how do I find out without the device?

Open the session: [analyze rules](rules/analyze-rules/README.md) have already flagged what went wrong — app install error codes, detection-rule failures that break the ESP, blocking-app timeouts, content download and proxy failures, TPM attestation and MDM enrollment error codes, hybrid join problems, failed Windows updates, low disk space or battery (see the [built-in rules reference](rules/analyze-rules/built-in-rules.md)). [Guided diagnosis](portal-guide/session-details-and-diagnosis.md#guided-diagnosis) names the primary suspect with a copyable quick fix, and [diagnostics collection](troubleshooting/diagnostics-and-log-collection.md) brings the agent and IME logs to you.

### Can failed enrollments be analyzed automatically instead of reading IME logs by hand?

Yes. Dozens of built-in, community-maintained rules evaluate every session automatically and report confidence-scored findings with remediation steps. Adapt [template rules](rules/analyze-rules/template-rules.md), [build your own](rules/analyze-rules/cookbook.md), add [IME log patterns](rules/ime-log-patterns.md), or use [gather rules](rules/gather-rules.md) to capture registry, file, or WMI data on any event. [Regression detection](rules/analyze-rules/README.md#regression-detection) tells you when a rule starts firing more often than usual.

### How do I get alerted when an enrollment fails?

Configure [notifications](integrations/notifications.md) for Microsoft Teams, Slack, Discord, or a generic JSON webhook, and pick the triggers: enrollment start, success, or failure. The same channel carries SLA breach and resolution alerts, consecutive-failure alerts, and hardware rejection notices.

### Is there reporting on my Autopilot deployments — success rate, duration, failing apps?

Yes. [Fleet Health](portal-guide/fleet-health.md) shows the success rate, average enrollment time, a daily timeline, top failure reasons, the slowest and most-failing models and apps, and a first-time-right rate that reveals devices that needed several attempts. [SLA Compliance](portal-guide/sla-compliance.md) reports against your own targets; [Software Inventory & Vulnerabilities](portal-guide/software-inventory-and-vulnerabilities.md) shows what was installed and what is affected by known CVEs.

### Why does enrollment take so long, and which app is slowing it down?

Every finished session gets a [time attribution](portal-guide/session-details-and-diagnosis.md#time-attribution) bar that shows where the minutes went and which apps blocked the ESP. [Fleet Health](portal-guide/fleet-health.md#time-attribution) aggregates the same data per enrollment class and ranks the apps that cost the most time, and [Geographic Performance](portal-guide/geographic-performance.md) finds slow sites and shows how much content came from Delivery Optimization peers.

### Can end users or field technicians check a device's status without portal access?

Yes. The [Progress Portal](portal-guide/progress-portal.md) shows a device's enrollment status by serial number — status, progress bar, steps, and what is currently downloading or installing — live and strictly read-only. No portal role is needed; the serial number is the access key.

### Can a managed service provider monitor several customer tenants?

Yes. [Delegated administration](concepts/roles-and-permissions.md) gives an MSP read-only access to a defined set of customer tenants from one login, with fleet analytics scoped to exactly those tenants, secrets redacted, and every grant written to the customer's audit log.

### Can I ask an AI assistant about my enrollments?

Yes. Connect Claude Desktop, VS Code with Claude, or any MCP client to the [Autopilot Monitor MCP server](integrations/ai-integration-mcp.md) and ask questions like "show me all failed enrollments from the last 24 hours". Access follows your portal role and is scoped to your tenant.

### Where is my data stored, and what does the agent do on the device?

In Germany, in Azure Germany West Central; see [Where is my data stored?](troubleshooting/faq.md#general) and the [Security & Privacy FAQ](trust/security-faq.md). The agent runs as a scheduled task only for the duration of the enrollment and removes itself on completion — see [Agent Lifecycle & Security](concepts/agent-lifecycle-and-security.md).

## Where to go next

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>🚀 Getting Started</strong></td><td>Onboard your tenant, deploy the agent via Intune, and watch your first enrollment live.</td><td><a href="getting-started/how-it-works.md">how-it-works.md</a></td></tr><tr><td><strong>📐 Concepts</strong></td><td>Sessions, statuses, roles, and how the temporary enrollment agent works under the hood.</td><td><a href="concepts/sessions-and-statuses.md">sessions-and-statuses.md</a></td></tr><tr><td><strong>🧩 Rules</strong></td><td>Let built-in rules analyze every enrollment automatically — and build your own with the cookbook.</td><td><a href="rules/overview.md">overview.md</a></td></tr><tr><td><strong>🔔 Integrations</strong></td><td>Teams, Slack, Discord, and webhook notifications — plus AI-powered analysis via MCP.</td><td><a href="integrations/notifications.md">notifications.md</a></td></tr><tr><td><strong>📖 Reference</strong></td><td>Every setting, agent command-line parameter, and the bootstrap script explained.</td><td><a href="reference/settings.md">settings.md</a></td></tr><tr><td><strong>🛟 Troubleshooting</strong></td><td>FAQ, common problems, and how to collect diagnostics when something goes wrong.</td><td><a href="troubleshooting/faq.md">faq.md</a></td></tr></tbody></table>
