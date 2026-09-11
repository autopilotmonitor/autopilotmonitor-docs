---
type: Reference
description: >-
  The available plans of Autopilot Monitor — the free Community plan and the
  Pro plan, how to buy Pro and how to try it first.
tags:
  - plans
  - licensing
  - features
---

# Plans

Autopilot Monitor is available in two plans. Your tenant's current plan, and the side-by-side comparison below, are shown in the portal under **Settings → Tenant → Plan**; the same comparison is public at [autopilotmonitor.com/plans](https://www.autopilotmonitor.com/plans).

## Community — _available now_

The Community plan is what this documentation describes: the full product as it exists today. It is **free — and stays free**. That's the point of a community plan: it is the free way to use Autopilot Monitor, publicly available to every organization.

* **Access:** self-service — sign in with your work account; new tenants are activated after a short activation step. See [Requirements & Access](getting-started/requirements-and-access.md).
* **Features:** the complete current feature set — live session monitoring, the full [rules engine](rules/overview.md) including custom rules, fleet analytics, notifications, and diagnostics. The [AI integration (MCP)](integrations/ai-integration-mcp.md) is included with **per-account and organization-wide usage limits tied to your tenant's usage plan**.
* **Data retention:** session and telemetry data is retained for up to **90 days**.
* **Production use is fine** — the Community plan is meant for real fleets, not just labs. What you accept in return: community-based support, and later on, certain capabilities will be Pro-only.
* **Support:** community-based via [GitHub issues](https://github.com/okieselbach/Autopilot-Monitor/issues); rules and IME patterns are community-maintained.
* **Maintained by** Oliver Kieselbach as an open community contribution, and operated by glueckkanja AG — without commitments as to availability or support. See the [Terms of Use](https://www.autopilotmonitor.com/terms) and the [Security & Privacy FAQ](trust/security-faq.md).
* **Active development:** frequent updates, no availability guarantees, and data structures may change — see [Requirements & Access](getting-started/requirements-and-access.md#getting-access-tenant-activation).

## Pro — _available now_

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

### Buy Pro

Pro is sold through two channels — both lead to the same Pro plan on your existing tenant, and your data stays where it is:

* **Microsoft Marketplace** — [open the listing](https://marketplace.microsoft.com/en-us/product/saas/glueckkanja-gabag.autopilot-monitor-transactable-prod?tab=Overview), billed through your Azure subscription.
* **Cleverbridge** — [go to checkout](https://www.cleverbridge.com/306/purl-Autopilot-Monitor-Buy-Y), by credit card, PayPal or bank transfer.

Prerequisites, payment and subscription management per channel are described under [How to Purchase](troubleshooting-and-support/how-to-purchase/README.md); the list price is shown at [autopilotmonitor.com/plans](https://www.autopilotmonitor.com/plans).

### Try Pro first

A tenant administrator can start a one-time, free **30-day Pro trial** under **Settings → Tenant → Plan**. Pro — the trial included — needs a contact address and a company name under **Settings → Tenant → Contact** first, so we can reach and identify you for support; the Plan section tells you what is still missing. When the trial ends, the tenant returns to Community automatically.

Questions about Pro, or requirements it must cover? Reach out via [LinkedIn](https://www.linkedin.com/in/oliver-kieselbach) or a [GitHub issue](https://github.com/okieselbach/Autopilot-Monitor/issues).

## At a glance

|                                                 | Community                                                         | Pro                                                                       |
| ----------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Availability**                                | Now — publicly available, free                                    | Now — Microsoft Marketplace or Cleverbridge; free 30-day trial            |
| **Price**                                       | Free — always                                                     | See [autopilotmonitor.com/plans](https://www.autopilotmonitor.com/plans)  |
| **Feature set**                                 | Full current feature set — AI (MCP) within usage limits           | Everything in Community, plus the Pro capabilities above                  |
| **Data retention**                              | Up to 90 days                                                     | Up to 365 days                                                            |
| **Portal & agent API rate limits**              | Standard                                                          | Advanced                                                                  |
| **AI (MCP) usage quota**                        | Small                                                             | Advanced                                                                  |
| **Delegated (MSP) administration**              | —                                                                 | Included (2 managed tenants; additional tenants as add-ons — each additional slot also extends the AI (MCP) budgets) |
| **Managed by a Pro organization**               | Tenant is on Pro (badge "Pro (MSP)") for as long as it is managed | Same — the badge shows the delegation as the source                       |
| **OOBE bootstrap sessions / Unrestricted Mode** | —                                                                 | Included, activated on request                                            |
| **Support**                                     | Community (GitHub)                                                | Priority support with reliability commitments                             |
| **Operator & counterparty**                     | glueckkanja AG — no commitments                                   | glueckkanja AG under written agreement                                    |
| **Maintainer**                                  | Oliver Kieselbach (open community contribution)                   | Oliver Kieselbach                                                         |
| **Data processing agreement**                   | [Published DPA](legal/data-privacy-agreement-dpa/README.md), accepted at sign-up | Same DPA                                                                  |
| **Intended for**                                | Labs **and** production fleets — with community support           | Organizations needing support commitments and Pro-only capabilities, MSPs |

Both plans run on the same infrastructure, in the same region, with the same security model — see the [Security & Privacy FAQ](trust/security-faq.md). The plan changes limits, support, and the contractual counterparty, not how your data is protected.
