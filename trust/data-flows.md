---
type: Reference
tags: [security, privacy, data-flows, architecture, trust]
timestamp: 2026-07-21
description: >-
  Where data actually travels when you use Autopilot Monitor — what the service
  stores, what it only reads from, and which connections exist because you
  configured them.
---

# Data Flows and External Services

**Last reviewed: 21 July 2026.**

A technical map of every outbound connection Autopilot Monitor makes, grouped by what actually happens to your data. Read the groups carefully — they are not equivalent, and most of them carry no customer data at all.

{% hint style="info" %}
This page describes the architecture. The **data processing agreement** is the authoritative document for the contractual side — which parties are engaged, on what terms, and how changes are handled. It is available on request; see the [Security & Privacy FAQ](security-faq.md#can-i-get-a-data-processing-agreement-dpa--avv).
{% endhint %}

## 1. Where your data is stored

| Service | Role | Data | Location |
| --- | --- | --- | --- |
| **Microsoft Azure** | The platform itself — compute, storage, real-time messaging, container hosting, operational telemetry | All customer data: sessions, events, configuration, audit logs, backups, and any hosted diagnostics | **Germany West Central** (portal front-end static assets: West Europe) |

That is the whole of it. Your telemetry is not copied to an analytics platform, a CRM, a third-party error tracker, or an AI provider, and no external support desk holds a copy.

## 2. Messages the service sends

Used to deliver messages *about* the service. **Neither carries enrollment telemetry.**

| Service | When it is used | What it receives |
| --- | --- | --- |
| **Resend** | Onboarding only — the approval message sent when a tenant is activated for the Private Preview | The recipient administrator's email address and the tenant's domain name |
| **Telegram** | Operator alerting, so incidents get noticed and acted on quickly | Platform health signals: event type, severity, a short message, and the tenant ID an event relates to. No device data, no enrollment telemetry, no personal data |

Telegram is **operator-only infrastructure**, not a tenant feature — it cannot be configured or used by customers. It is listed here because it exists in the open-source code and you would find it anyway; it is how the people running the service learn that something is wrong.

Autopilot Monitor does **not** route notification alerts through an email provider. There is no other mail service in the product.

## 3. Data the service reads in — nothing goes out

Public reference data is pulled *from* these sources to enrich your view. **Nothing about your environment is sent to them.** Your software inventory is never uploaded for lookup; matching happens inside the backend against a cached copy of the public data.

| Source | Purpose |
| --- | --- |
| **National Vulnerability Database (NVD)**, NIST | CVE and CPE records for vulnerability correlation. This product uses the NVD API but is not endorsed or certified by the NVD. |
| **CISA Known Exploited Vulnerabilities (KEV) Catalog** | Which vulnerabilities are actively exploited in the wild. |
| **Microsoft Security Response Center (MSRC)** | Microsoft-specific vulnerability details. |

## 4. Lookups inside your own tenant

The service queries **Microsoft Graph in your tenant**, using the permissions granted at onboarding, to verify that a device reporting telemetry really is a registered Autopilot device of yours, and to resolve friendly names for scripts and applications. This reads your own directory; it discloses nothing to anyone else. See [Optional Graph Permissions](../reference/optional-graph-permissions.md) for what is required and what is optional.

## 5. Destinations you choose

These connections exist **only because someone configured them**. They point at systems you nominate, and you can remove them at any time.

| Destination | When it applies | What is sent |
| --- | --- | --- |
| **Your own Azure storage account** | Diagnostics upload with the default `CustomerSas` destination | The diagnostics package — agent logs, IME logs, session info. This is the default: the payload never reaches our infrastructure. |
| **Microsoft Teams, Slack, or a generic webhook endpoint** | Notification channels you configure | Alert payloads — session, device, and finding details. Webhook targets pass an SSRF guard before any request is made. |
| **Your AI assistant** | If a user connects an AI client through the [MCP integration](../integrations/ai-integration-mcp.md) | Whatever that assistant queries. The platform itself makes no AI calls; this transfer is initiated by your user, to your vendor, under your agreement with them. MCP access is per-user and must be enabled by an administrator. |
| **IP geolocation service** (`ipinfo.io`, fallback `ifconfig.co`) | Geolocation, a tenant setting that is **on by default** | The device's outbound connection reaches the service, which returns approximate location. The session stores country, region, city, and approximate coordinates; the outbound IP is stored separately as a diagnostic event. Disable geolocation to stop this entirely — see the [Security & Privacy FAQ](security-faq.md#is-the-devices-ip-address-stored). |

{% hint style="info" %}
To eliminate group 5 entirely: leave diagnostics upload off or on `CustomerSas`, configure no notification channels, enable no MCP users, and switch off geolocation. The service is fully functional with all of them disabled — only Geographic Performance goes dark, because it has nothing to plot.
{% endhint %}

# Citations

* [Security & Privacy FAQ](security-faq.md) — data residency, isolation, encryption, retention, and contracting.
* [Network Endpoints](../reference/network-endpoints.md) — the outbound hosts to allow on firewalls and proxies.
* [Notifications](../integrations/notifications.md) — configuring notification channels.
* [Diagnostics & Log Collection](../troubleshooting/diagnostics-and-log-collection.md) — diagnostics upload modes.
