---
type: Reference
tags: [settings, configuration]
description: >-
  Every tenant setting explained — access, device validation, notifications,
  agent behavior, diagnostics, data retention, and offboarding.
---

# Settings Reference

The **Settings / Configuration** area (Tenant Admins; Operators can view and manage most settings) controls how Autopilot Monitor behaves for your tenant. Settings are grouped into **Tenant**, **Agent**, **Maintenance**, and **Reporting**.

{% hint style="info" %}
Two groups are **optional features unlocked per tenant on request** (open a [GitHub issue](https://github.com/okieselbach/Autopilot-Monitor/issues)): **Bootstrap Sessions** and **Unrestricted Mode**. They only appear in your settings after the platform operators have activated them for your tenant.
{% endhint %}

## Tenant

### Access Management

| Setting | Description |
| --- | --- |
| Team Members & Roles | Add members by UPN, assign **Admin** or **Operator**, enable/disable accounts, and grant bootstrap-token management. The first user to sign in becomes Admin automatically. See [Roles & Permissions](../concepts/roles-and-permissions.md). |

### Enrollment Device Validation

| Setting | Default | Description |
| --- | --- | --- |
| Autopilot Device Validation | Disabled | Only devices registered as Windows Autopilot devices in your Intune tenant may register sessions. Requires admin consent for the Graph permission `DeviceManagementServiceConfig.Read.All` (read-only). |
| Corporate Identifier Validation | Disabled | Validates devices against Intune Corporate Device Identifiers (manufacturer, model, serial). Same Graph permission. |

{% hint style="warning" %}
**At least one validation method must be enabled** — with both off, the backend rejects all agent requests. This is the consent gate described in [Portal Setup](../getting-started/portal-setup.md).
{% endhint %}

### Hardware Whitelist

| Setting | Default | Description |
| --- | --- | --- |
| Allowed Manufacturers | `Dell*, HP*, Lenovo*, Microsoft Corporation` | Comma-separated manufacturer names permitted to register sessions; wildcards supported. `*` allows all. |
| Allowed Models | `*` | Same mechanics for device models, e.g. `Latitude*,EliteBook*`. |

### Notifications

| Setting | Description |
| --- | --- |
| Notification Provider | **Microsoft Teams (Workflow Webhook)** — recommended, free, no Power Automate Premium needed · **Microsoft Teams (Legacy Connector)** — deprecated by Microsoft, existing configs keep working · **Slack** (Incoming Webhook) · **Generic JSON webhook** — stable JSON payload for ticketing/automation systems, supports custom HTTP headers (e.g. `Authorization`). |
| Webhook URL | The URL generated during provider setup. |
| Notify on Start / Success / Failure | Which session events trigger a notification. Keep *Failure* on so broken enrollments surface without watching the portal. |
| Send Test Notification | Fires a sample message to verify the configuration. |

Setup walkthroughs per provider: [Notifications](../integrations/notifications.md).

### SLA Targets

Define your enrollment SLA thresholds (e.g. target duration and success rate); the [SLA Compliance](../portal-guide/sla-compliance.md) view reports against them.

### Bootstrap Sessions *(optional feature — on request)*

| Setting | Description |
| --- | --- |
| Bootstrap Tokens | Create tokens that let devices register **before** device validation applies (pre-MDM scenarios). Each token has a short code + URL + ready-to-use PowerShell command, a validity of 1 h to 7 days, a status (Active / Expired / Revoked), and a usage count. Tokens can be revoked at any time. Details: [Bootstrap Script & Tokens](bootstrap-script-and-tokens.md). |

## Agent

### Agent Parameters

| Setting | Default | Description |
| --- | --- | --- |
| Self-Destruct on Complete | Enabled | The agent removes its scheduled task and files after enrollment. Recommended — the agent is temporary by design. |
| Keep Log File | Disabled | Preserve the agent's local log through self-destruct (troubleshooting). |
| Reboot on Complete | Disabled | Reboot when enrollment completes; **Reboot Delay** 0–3600 s (default 10 s). |
| Geo-Location Detection | Enabled | IP-based public IP / approximate location / ISP lookup at enrollment time. Disable if outbound third-party requests are prohibited. |
| Set Timezone Automatically | Disabled | Sets the device timezone from the geolocation result (`tzutil /s`). Requires Geo-Location Detection. |
| IME Pattern Match Log | Disabled | Writes matched IME log lines to `%ProgramData%\AutopilotMonitor\Logs\ime_pattern_matches.log` — the debugging tool for [IME Log Patterns](../rules/ime-log-patterns.md). |
| Show Script Output (stdout) | Enabled | Shows PowerShell script stdout in the timeline. Disable if scripts may print sensitive data; stderr is always shown. |
| Log Level | Info | Agent's own log verbosity: Info · Debug · Verbose · Trace. Raise only while diagnosing an agent issue. |
| Show Enrollment Summary | Disabled | Shows the end user a visual summary dialog after enrollment (success and failure). Requires the SummaryDialog companion. Sub-settings: **Auto-Close Timeout** (default 60 s; 0 = manual close), **Launch Retry Timeout** (default 120 s — retries while credential UI like Windows Hello locks the desktop), **Branding Image URL** (540 × 80 px banner). |

### Agent Collectors

| Setting | Default | Description |
| --- | --- | --- |
| Performance Collector | Enabled, 30 s | Periodic CPU/memory/disk/network snapshots (interval 30–300 s). Core collectors (enrollment tracking, Windows Hello detector) are always active. |
| Hello Wait Timeout | 30 s | How long to wait for the Windows Hello wizard after ESP exit (30–300 s) before proceeding with completion. |

### Agent Analyzers

| Setting | Default | Description |
| --- | --- | --- |
| Local Admin Analyzer | Enabled | Detects pre-enrollment local admin accounts (a known Autopilot bypass) at enrollment start and completion. **Allowed Local Accounts** defines your expected accounts; built-in Windows accounts are always allowed. Feeds the ANALYZE-ID-002 rule. |
| Software Inventory & Vulnerability Analyzer | Disabled · *Experimental* | Collects installed software at start and completion, detects during-enrollment deltas, and correlates against NVD CVEs, CISA KEV, and MSRC. Feeds the [Software Inventory](../portal-guide/software-inventory-and-vulnerabilities.md) view and ANALYZE-ID-003. |

### Diagnostics Package

| Setting | Default | Description |
| --- | --- | --- |
| Upload destination | Your own Azure Blob Storage | **Your own Azure Blob Storage** — agents upload via a Container SAS URL you provide; data never leaves your Azure tenant. **Hosted storage** (opt-in) — uploads to the built-in storage under a per-tenant prefix using short-lived (15 min), blob-scoped, write-only tokens minted per upload; no long-lived credentials. Switching destinations always requires an explicit admin action. |
| Blob Storage Container SAS URL | — | Only for the own-storage destination. Needs **Read, Write, Create** at container level. Stored securely in the backend, never sent to devices — agents request a short-lived upload URL per upload. The portal shows a green/amber/red expiry indicator for the SAS. |
| Upload Mode | Off | **Off** · **Always** · **On Failure Only** (recommended when storage cost matters). |
| Additional Log Paths | — | Extra files/wildcard patterns for the diagnostics ZIP, on top of the platform-wide global paths. Environment variables are expanded; wildcards only in the last path segment; `%LOGGED_ON_USER_PROFILE%` targets the signed-in user's `AppData\Local`/`Roaming`; **Include Subfolders** collects recursively. Paths are validated against an agent-side allow-list of known log locations. |

### Unrestricted Mode *(optional feature — on request)*

Relaxes the [Gather Rule guardrails](../rules/gather-rules.md#security-guardrails): any registry path, WMI query, and PowerShell/system command becomes available; file and diagnostics paths open up except `C:\Users` (always blocked for privacy). Dangerous operations (downloads, user creation, boot manipulation, persistence mechanisms) remain hard-blocked. Requires platform-side activation before the toggle is visible.

## Maintenance

### Data Management

| Setting | Default | Description |
| --- | --- | --- |
| Data Retention Period | 90 days (7–180) | Sessions and events are deleted automatically after this period. |
| Session Timeout | 5 hours (1–12) | *In Progress* sessions inactive past this threshold are reclassified from the evidence rather than blanket-failed: if Device Setup already finished they become **Awaiting User**, otherwise they eventually settle as **Incomplete** (a non-completion, **not** counted as a failure), and a session that later completes is reconciled to **Succeeded**. See [Sessions & Statuses](../concepts/sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions). Set it to match or slightly exceed your ESP timeout. |

### Danger Zone

| Action | Description |
| --- | --- |
| Offboard Tenant | **Irreversibly deletes all tenant data** — sessions, events, rules, audit logs, configuration, and all member accounts — and signs you out. Requires typing `OFFBOARD` to confirm. |

## Reporting

| Page | Description |
| --- | --- |
| MCP Usage | Per-tenant usage reporting for the [AI integration (MCP)](../integrations/ai-integration-mcp.md) — request volumes per user and tool. Visible when MCP access is enabled. |
