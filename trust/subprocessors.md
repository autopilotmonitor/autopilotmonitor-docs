---
type: Reference
tags: [security, privacy, gdpr, subprocessors, data-flows, trust]
timestamp: 2026-07-21
description: >-
  Every third party involved in operating Autopilot Monitor, what data reaches
  it, and which flows exist only because a customer configured them.
---

# Sub-processors and Third-Party Data Flows

**Last reviewed: 21 July 2026.**

This page lists every third party that can be involved when you use Autopilot Monitor, grouped by whether your data actually flows to it. It reflects the sub-processors in use as of the review date; changes are announced before they take effect (see [Changes to this list](#changes-to-this-list)).

Read the groups carefully — they are not equivalent. Only the first group processes customer data on our behalf.

## 1. Sub-processors

Third parties that store or process customer data as part of operating the service.

| Sub-processor | Role | Data | Location |
| --- | --- | --- | --- |
| **Microsoft Corporation** (Microsoft Azure) | Cloud infrastructure — compute, storage, real-time messaging, container hosting, operational telemetry | All customer data: sessions, events, configuration, audit logs, backups, and any hosted diagnostics | **Germany West Central** (portal front-end static assets: West Europe) |

That is the entire list. There is no analytics vendor, no CRM, no third-party error tracker, no AI provider, and no offshore support desk in the path of your data.

{% hint style="info" %}
Note what is **not** in this table: no email provider stores your telemetry, and no notification service is a sub-processor. Autopilot Monitor sends alerts to destinations *you* nominate (group 5) — it does not route your data through an intermediary to get there.
{% endhint %}

## 2. Service communications and platform operations

Used to send messages about the service. **No enrollment telemetry flows through either of these.**

| Provider | Role | Data | Notes |
| --- | --- | --- | --- |
| **Resend** | Transactional email for onboarding — the Private Preview approval message sent when a tenant is activated | The recipient administrator's email address and the tenant's domain name | Onboarding only. Autopilot Monitor does **not** route notification alerts through Resend, and there is no other email provider in the product. |
| **Telegram** | Alerting channel for the platform operators, so incidents are noticed and acted on quickly | Operational alert content: event type, severity, a short message, and the tenant ID the event relates to | **Operator-only infrastructure monitoring.** It is not a tenant feature, cannot be configured by customers, and carries no enrollment telemetry, device data, or personal data — only the platform's own health and incident signals. |

{% hint style="info" %}
Telegram is listed here for completeness, not because your data reaches it. It is how the operators find out that something is wrong at 2am — the same role a pager plays elsewhere.
{% endhint %}

## 3. Reference data sources — inbound only

We pull public reference data *from* these services to enrich your view. **No customer data is sent to them.** Your software inventory is never uploaded for lookup; matching happens inside our own backend against a cached copy of the public data.

| Source | Purpose |
| --- | --- |
| **National Vulnerability Database (NVD)**, NIST | CVE and CPE records for vulnerability correlation. This product uses the NVD API but is not endorsed or certified by the NVD. |
| **CISA Known Exploited Vulnerabilities (KEV) Catalog** | Which vulnerabilities are actively exploited in the wild. |
| **Microsoft Security Response Center (MSRC)** | Microsoft-specific vulnerability details. |

## 4. Microsoft Graph — your own tenant

The service queries **Microsoft Graph in your tenant**, using the permissions you granted at onboarding, to verify that a device claiming to report telemetry is genuinely a registered Autopilot device in your tenant, and to resolve friendly names for scripts and applications. This is a lookup against your own directory, not a disclosure to a third party. See [Optional Graph Permissions](../reference/optional-graph-permissions.md) for what is required versus optional.

## 5. Destinations you choose

These flows exist **only** because a customer configured them, are off by default, and can be removed at any time. They are under your control and your agreements, not ours.

| Destination | When it applies | What is sent |
| --- | --- | --- |
| **Your own Azure storage account** | Diagnostics upload with the default `CustomerSas` destination | The diagnostics package — agent logs, IME logs, session info. This is the default: the payload never reaches our infrastructure. |
| **Microsoft Teams, Slack, or a generic webhook endpoint** | Notification channels you configure | Alert payloads — session, device, and finding details. Webhook targets pass an SSRF guard before any request is made. |
| **Your AI assistant / LLM vendor** | If a user connects an AI client through the [MCP integration](../integrations/ai-integration-mcp.md) | Whatever that assistant queries. The platform itself makes no LLM calls; this transfer is initiated by your user, to your vendor, under your agreement. MCP access is per-user and must be enabled by an administrator. |
| **IP geolocation service** (`ipinfo.io`, fallback `ifconfig.co`) | Geolocation, a tenant setting that is **on by default** | The device's outbound connection reaches the service, which returns approximate location. The session stores country, region, city, and approximate coordinates; the outbound IP is stored separately as a diagnostic event. Disable geolocation to stop this entirely — see the [Security & Privacy FAQ](security-faq.md#is-the-devices-ip-address-stored). |

{% hint style="info" %}
To eliminate group 5 entirely: leave diagnostics upload off or on `CustomerSas`, configure no notification channels, enable no MCP users, and switch off geolocation. The service is fully functional with all of them disabled — only Geographic Performance goes dark, because it has nothing to plot.
{% endhint %}

## Changes to this list

New sub-processors are announced here and in [Service Announcements](../troubleshooting/service-announcements.md) before they are introduced. If you have a signed data processing agreement, notification follows the terms of that agreement.

# Citations

* [Security & Privacy FAQ](security-faq.md) — data residency, isolation, encryption, and retention.
* [Network Endpoints](../reference/network-endpoints.md) — the outbound hosts to allow on firewalls and proxies.
* [Notifications](../integrations/notifications.md) — configuring notification channels.
* [Diagnostics & Log Collection](../troubleshooting/diagnostics-and-log-collection.md) — diagnostics upload modes.
