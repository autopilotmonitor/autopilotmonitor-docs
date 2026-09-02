---
type: Reference
tags: [plans, licensing, features]
description: >-
  The available plans of Autopilot Monitor — the free Community plan today,
  the Pro plan coming soon.
---

# Plans

Autopilot Monitor will be available in two plans. Your tenant's current plan, and the side-by-side comparison below, are shown in the portal under **Settings → Tenant → Plan**; the same comparison is public at [autopilotmonitor.com/plans](https://autopilotmonitor.com/plans).

## Community — *available now*

The Community plan is what this documentation describes: the full product as it exists today. It is **free — and stays free**. That's the point of a community plan: it is the free way to use Autopilot Monitor, publicly available to every organization.

* **Access:** self-service — sign in with your work account; new tenants are activated after a short activation step. See [Requirements & Access](getting-started/requirements-and-access.md).
* **Features:** the complete current feature set — live session monitoring, the full [rules engine](rules/overview.md) including custom rules, fleet analytics, notifications, and diagnostics. The [AI integration (MCP)](integrations/ai-integration-mcp.md) is included with **per-account and organization-wide usage limits tied to your tenant's usage plan**.
* **Data retention:** session and telemetry data is retained for up to **90 days**.
* **Production use is fine** — the Community plan is meant for real fleets, not just labs. What you accept in return: community-based support, and later on, certain capabilities will be Pro-only.
* **Support:** community-based via [GitHub issues](https://github.com/okieselbach/Autopilot-Monitor/issues); rules and IME patterns are community-maintained.
* **Maintained by** Oliver Kieselbach as an open community contribution, and operated by glueckkanja AG — without commitments as to availability or support. See the [Terms of Use](https://autopilotmonitor.com/terms) and the [Security & Privacy FAQ](trust/security-faq.md).
* **Active development:** frequent updates, no availability guarantees, and data structures may change — see [Requirements & Access](getting-started/requirements-and-access.md#getting-access-tenant-activation).

## Pro — *coming soon*

A commercial plan for organizations that need more than the Community plan can promise — reliability commitments and priority support, plus higher operating limits. Pro includes **everything in Community**, plus:

* **Extended data retention** — 365 days (vs 90)
* **Higher portal & agent API rate limits**
* **Larger AI (MCP) usage quota**
* **Delegated (MSP) administration** across tenants — manage multiple customer tenants from one place; see [Roles & Permissions](concepts/roles-and-permissions.md)
* **OOBE bootstrap sessions** — run the agent already before MDM enrollment (activated on request); see [Bootstrap Script & Tokens](reference/bootstrap-script-and-tokens.md)
* **Unrestricted Mode** for advanced data collection (activated on request); see [Settings](reference/settings.md)
* **Reliability commitments & priority support**

It is aimed at larger fleets and managed service providers.

Pro is contracted with **glueckkanja AG**, a German company certified to ISO/IEC 27001 — which operates Autopilot Monitor for both plans, and is the counterparty for the agreement, the data processing agreement, and the support commitments. Local data residency in an additional region (for example a US deployment) is something we can accommodate in a Pro context; ask if you need it.

{% hint style="info" %}
🚧 **Coming soon.** Pricing and timeline will be announced here. Pro is planned to be sold through two channels — **direct purchase** (online, credit card or invoice) and the **Microsoft commercial marketplace** — and a one-time **30-day Pro trial** will be startable by tenant administrators from Settings → Tenant → Plan. None of these are open yet; the portal shows them as "coming soon". If the Pro plan is interesting for your organization — or you have requirements it must cover — reach out via [LinkedIn](https://www.linkedin.com/in/oliver-kieselbach) or a [GitHub issue](https://github.com/okieselbach/Autopilot-Monitor/issues); early feedback directly shapes what it becomes.
{% endhint %}

## At a glance

| | Community | Pro |
| --- | --- | --- |
| **Availability** | Now — publicly available, free | Coming soon |
| **Price** | Free — always | To be announced |
| **Feature set** | Full current feature set — AI (MCP) within usage limits | Everything in Community, plus the Pro capabilities above |
| **Data retention** | Up to 90 days | Up to 365 days |
| **Portal & agent API rate limits** | Standard | Higher |
| **AI (MCP) usage quota** | Standard | Larger |
| **Delegated (MSP) administration** | — | Included |
| **OOBE bootstrap sessions / Unrestricted Mode** | — | Included, activated on request |
| **Support** | Community (GitHub) | Priority support with reliability commitments |
| **Operator & counterparty** | glueckkanja AG — no commitments | glueckkanja AG under written agreement |
| **Maintainer** | Oliver Kieselbach (open community contribution) | Oliver Kieselbach |
| **Data processing agreement** | On request | Part of the agreement |
| **Intended for** | Labs **and** production fleets — with community support | Organizations needing support commitments and Pro-only capabilities, MSPs |

Both plans run on the same infrastructure, in the same region, with the same security model — see the [Security & Privacy FAQ](trust/security-faq.md). The plan changes limits, support, and the contractual counterparty, not how your data is protected.
