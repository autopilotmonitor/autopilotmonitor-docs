---
okf_version: "0.1"
---

# Autopilot Monitor — Customer Documentation Bundle

Customer-facing product documentation for [Autopilot Monitor](https://autopilotmonitor.com), organized as an
[Open Knowledge Format (OKF)](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
knowledge bundle and published via GitBook at https://docs.autopilotmonitor.com.
Contributor-facing technical documentation lives in the main product repository (separate bundle).

# Getting Started

* [How Autopilot Monitor Works](getting-started/how-it-works.md) - Architecture in one page: on-device agent, backend, portal, and how telemetry flows.
* [Requirements & Access](getting-started/requirements-and-access.md) - Prerequisites, licensing, and who needs which permissions before onboarding.
* [Portal Setup](getting-started/portal-setup.md) - Tenant onboarding and admin consent in the portal.
* [Deploy the Agent](getting-started/deploy-the-agent.md) - Rolling out the agent with Intune (platform script bootstrap).
* [Your First Monitored Enrollment](getting-started/your-first-monitored-enrollment.md) - Walkthrough of a first end-to-end monitored Autopilot enrollment.

# Concepts

* [Sessions & Statuses](concepts/sessions-and-statuses.md) - What a session is, every status it can take, and how completion is decided.
* [Roles & Permissions](concepts/roles-and-permissions.md) - Portal roles, tenant scoping, and the permission model.
* [Agent Lifecycle & Security](concepts/agent-lifecycle-and-security.md) - How the agent installs, authenticates, runs, and removes itself.

# Portal Guide

* [Dashboard & Sessions](portal-guide/dashboard-and-sessions.md) - The main dashboard and session list.
* [Session Details & Diagnosis](portal-guide/session-details-and-diagnosis.md) - Timeline, events, rule findings, and diagnosing a single enrollment.
* [Progress Portal](portal-guide/progress-portal.md) - Real-time enrollment progress view for helpdesk and end users.
* [Fleet Health](portal-guide/fleet-health.md), [Geographic Performance](portal-guide/geographic-performance.md), [SLA Compliance](portal-guide/sla-compliance.md), [Usage Metrics](portal-guide/usage-metrics.md) - Fleet-wide analytics surfaces.
* [Software Inventory & Vulnerabilities](portal-guide/software-inventory-and-vulnerabilities.md) - Installed software and CVE exposure per enrollment.
* [Audit Log & System Health](portal-guide/audit-log-and-system-health.md) - Auditing portal actions and monitoring service health.

# Rules

* [How Rules Work Together](rules/overview.md) - Analyze rules vs gather rules and how they interact.
* [Analyze Rules](rules/analyze-rules/README.md) - Rule engine on session timelines: [concepts](rules/analyze-rules/concepts.md), [built-in rules reference](rules/analyze-rules/built-in-rules.md), [template rules](rules/analyze-rules/template-rules.md), [cookbook](rules/analyze-rules/cookbook.md).
* [Gather Rules](rules/gather-rules.md) - Collecting additional on-device data during enrollment.
* [IME Log Patterns](rules/ime-log-patterns.md) - Pattern matching on Intune Management Extension logs.

# Integrations

* [Notifications](integrations/notifications.md) - Notification channels (Teams, Slack, email, generic webhooks) and rule-driven notifications.
* [AI Integration (MCP)](integrations/ai-integration-mcp.md) - Connecting AI clients to the read-only MCP telemetry server.

# Reference

* [Settings Reference](reference/settings.md) - Every tenant/portal setting explained.
* [Network Endpoints](reference/network-endpoints.md) - The outbound HTTPS (443) hosts to allow on firewalls, proxies, and gateways.
* [Optional Graph Permissions](reference/optional-graph-permissions.md) - Opt-in tenant-side Graph permission grants for optional features via appRoleAssignment, without changing the published app manifest.
* [Agent Command-Line Parameters](reference/agent-command-line.md) - Agent CLI switches.
* [Bootstrap Script & Tokens](reference/bootstrap-script-and-tokens.md) - Bootstrap deployment script and enrollment tokens.

# Troubleshooting & Support

* [FAQ](troubleshooting/faq.md) - Frequently asked questions.
* [Common Problems](troubleshooting/common-problems.md) - Known issues and how to resolve them.
* [Diagnostics & Log Collection](troubleshooting/diagnostics-and-log-collection.md) - Collecting agent/device diagnostics for support.
* [Service Announcements](troubleshooting/service-announcements.md) - How service announcements reach the portal.

# Changelog

* [Agent Changelog](changelog/agent-changelog.md) and [Platform Changelog](changelog/platform-changelog.md) - Release history.

# Conventions for this bundle

* Every content page is a markdown file with YAML frontmatter; `type` is mandatory, `description` and `tags` are carried on all pages. GitBook uses the same frontmatter (`description`) for rendering.
* Use standard **relative** markdown links between documents. Do NOT use the spec's `/`-prefixed bundle-absolute form — GitBook and GitHub resolve those differently and navigation breaks.
* `README.md` + `SUMMARY.md` are GitBook structure files (site entry point and table of contents); `index.md` (this file) and `log.md` are OKF reserved files and are intentionally NOT listed in `SUMMARY.md`, so GitBook does not publish them as pages.
* Record notable additions and changes in [log.md](log.md).
