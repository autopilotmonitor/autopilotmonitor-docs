---
description: >-
  Real-time monitoring, diagnostics, and automated analysis for Windows
  Autopilot enrollments.
---

# Welcome

**Autopilot Monitor** gives IT admins and MSPs real-time visibility into Windows Autopilot enrollments — device provisioning sessions, the Enrollment Status Page (ESP), app and script installations, and completion, all streamed live to a central portal. Automated analyze rules flag problems as they happen and tell you how to fix them, before your users ever notice.

{% hint style="info" %}
🚧 **This documentation is being built out.** Pages marked as under construction are on their way — the structure below shows what's coming.
{% endhint %}

## Where to go next

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>🚀 Getting Started</strong></td><td>Onboard your tenant, deploy the agent via Intune, and watch your first enrollment live.</td><td><a href="getting-started/how-it-works.md">how-it-works.md</a></td></tr><tr><td><strong>📐 Concepts</strong></td><td>Sessions, statuses, roles, and how the temporary enrollment agent works under the hood.</td><td><a href="concepts/sessions-and-statuses.md">sessions-and-statuses.md</a></td></tr><tr><td><strong>🧩 Rules</strong></td><td>Let built-in rules analyze every enrollment automatically — and build your own with the cookbook.</td><td><a href="rules/overview.md">overview.md</a></td></tr><tr><td><strong>🔔 Integrations</strong></td><td>Teams, Slack, and webhook notifications — plus AI-powered analysis via MCP.</td><td><a href="integrations/notifications.md">notifications.md</a></td></tr><tr><td><strong>📖 Reference</strong></td><td>Every setting, agent command-line parameter, and the bootstrap script explained.</td><td><a href="reference/settings.md">settings.md</a></td></tr><tr><td><strong>🛟 Troubleshooting</strong></td><td>FAQ, common problems, and how to collect diagnostics when something goes wrong.</td><td><a href="troubleshooting/faq.md">faq.md</a></td></tr></tbody></table>

## What is Autopilot Monitor?

Windows Autopilot enrollment is a black box: when a device gets stuck at the Enrollment Status Page, the only built-in answer is "wait, then reset." Autopilot Monitor opens that box. A lightweight, temporary agent runs on the device during enrollment, streams every meaningful signal — ESP phases, Win32/LOB app installs, PowerShell scripts, policies, performance, security posture — to the portal, and removes itself when enrollment completes.

On top of that live timeline, **analyze rules** evaluate every session automatically: they detect failed detections, dependency chains, disk exhaustion, certificate problems, unexpected local admins, and much more — each finding comes with a confidence score, an explanation, and concrete remediation steps. You can use the built-in rules, adapt templates, or write your own.
