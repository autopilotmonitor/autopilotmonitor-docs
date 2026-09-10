---
description: >-
  Support and service level for Autopilot Monitor Pro: eligibility, support
  hours and response targets, availability target, service credits, and the
  limits of our liability.
---

# Support & Service Level

***

## Scope and precedence

This page describes the support services and the service level for **Autopilot Monitor Pro**. It is provided by glueckkanja AG, \[registered seat], ("we", "us", "our") and forms part of the agreement under which you subscribed to Autopilot Monitor Pro.

Where this page and your individually signed agreement or order form differ, the agreement or order form prevails. For the processing of personal data, the data processing agreement (DPA) applies and takes precedence over this page. This page is governed by German law; the place of jurisdiction is the one stated in the agreement under which you purchased Autopilot Monitor Pro.

{% hint style="info" %}
The free **Community plan**, evaluation tenants, preview features, and self-hosted deployments of the open-source components are provided **as is**, without support and without a service level. See [Plans](../plans.md) for the differences between the plans.
{% endhint %}

## Support

### Eligibility

Customers with an active Autopilot Monitor Pro subscription are eligible for support. Support is included in the subscription fee; there is no separate support fee and no ticket quota.

### What support covers

Our support covers technical assistance for administrators:

* Technical questions about the features of Autopilot Monitor
* Handling of incidents affecting Autopilot Monitor, including the agent, the portal, the backend, and the MCP server

### What support does not cover

The following is outside the scope of the included support:

* Problems in Microsoft Intune, Entra ID, Windows Autopilot, Windows, or any other third-party product, even when Autopilot Monitor makes them visible
* Writing, adapting, or debugging your own [analyze rules](../rules/analyze-rules/) and [gather rules](../rules/gather-rules.md) beyond pointing you to the documentation
* Consulting, workshops, training, on-site work, and administration of your environment
* Feature development and changes to product behaviour, and modified builds, self-hosted deployments, or agent versions that are no longer supported (see [Agent Changelog](../changelog/agent-changelog.md))

### How to reach us

Open a ticket through our [support portal](https://products.glueckkanja.com/support/tickets/new?ticket_form=technical_support_request_\(autopilotmonitor\)).

Every request is tracked as a ticket, and that ticket is the record of the support case. We communicate in writing, by e-mail or through the web interface of our ticketing system. Where it helps, our support engineers may offer a Microsoft Teams call to work on an issue together with you.

### What we need from you

So that we can start work without an extra round trip, please include with your ticket:

* Your tenant ID and, for session-specific issues, the session ID or device serial number
* What you expected to happen, what happened instead, and when it started
* A [diagnostics package](../troubleshooting/diagnostics-and-log-collection.md) where the issue involves a device or an agent

The easiest way to send us all of this is directly from the portal: [**Report Session**](../troubleshooting/diagnostics-and-log-collection.md#reporting-a-session) on the affected session, or **Settings → Tenant → Submit Logs** for issues not tied to a session. Both send your comment, contact address and attachments to us, and Report Session adds the session's timeline and, optionally, its diagnostics package. If you use one of them, the list above is covered; just mention the report in your ticket.

Our response target starts once we have a ticket with enough information to reproduce or investigate the issue.

### Languages

Our engineers are happy to support you in:

* English
* German

### Support hours

* Monday to Friday
* 08:00 to 18:00, Europe/Berlin time (CET/CEST)
* Excluding public holidays in Hesse, Germany, and 24 and 31 December

### Response target

**Typically less than 8 hours for incidents**, counted within support hours.

The response time is the period between your report of an incident and the moment one of our support engineers starts working on it. This is a target, not a guaranteed response time, and no service credits are attached to it. We do not commit to a resolution time, because how long a fix takes depends on the cause, and often on third parties.

## Service level

### Service hours

The service is operated 24×7.

### Availability target

The target availability is **99.5% per calendar month**, which corresponds to roughly 3 hours 40 minutes of downtime in a 31-day month. Availability is calculated as:

`availability` = (`service period` − `downtime`) / `service period`

where

* `service period` is the calendar month concerned, and
* `downtime` is the accumulated time during which the service was unavailable.

### How availability is measured

The service is considered **unavailable** when, for reasons within our control, either

* administrators cannot sign in to the portal or the portal does not respond to valid requests, or
* the backend does not accept telemetry from agents that are correctly deployed and configured.

Downtime is measured by our own monitoring, is counted in full minutes, and starts at the earlier of our monitoring detecting the outage or your report reaching our support. Individual functions that are degraded while the portal and telemetry intake keep working are handled as incidents, not as downtime. The current state of the platform is shown on the [System Health](../portal-guide/audit-log-and-system-health.md#system-health) page, and notable events are documented under [Service Announcements](../troubleshooting/service-announcements.md).

### What does not count as downtime

* Planned maintenance that we announce at least 48 hours in advance through Service Announcements or in-portal notice
* Emergency maintenance needed to close a security vulnerability
* Outages of Microsoft services the platform depends on, including Microsoft Entra ID, Microsoft Graph, Microsoft Intune, and Microsoft Azure
* Causes on your side, for example firewall, proxy or DNS configuration, revoked consent, expired credentials, misconfigured tenant settings, or agent deployments that do not meet the [requirements](../getting-started/requirements-and-access.md)
* Preview and beta features, and features you have switched off yourself
* Force majeure, and other events outside our reasonable control
* Suspension of the service because of overdue payment or a breach of the agreement

### Service credits

If the measured availability in a calendar month falls below the target, you can claim the following **time credit**, that is, additional days of service applied at your next subscription renewal at no charge:

| Measured availability in the affected month | Time credit |
| ------------------------------------------- | ----------- |
| below 99.5% and 99.0% or higher             | 5 days      |
| below 99.0%                                 | 15 days     |

The two tiers are not cumulative: only the higher applicable time credit is granted.

### Reporting and claiming

Please report outages to our support as soon as you notice them, so that we can start work immediately. To claim a credit, contact our support **within 30 days after the end of the affected calendar month** and state the month, the outage, and the ticket the outage was reported under.

### Data loss

Even with careful operation and maintenance, data loss can never be fully ruled out. This includes the data collected by the Autopilot Monitor Agent. We do not commit to a specific recovery point or recovery time objective for Autopilot Monitor Pro, and we are under no obligation to restore lost data.

If **all** of your tenant's data in Autopilot Monitor Pro is lost, you can claim a time credit of **six months**, applied at your next subscription renewal at no charge. Contact our support within 30 days after you become aware of the loss.

Autopilot Monitor records past enrollments and is not the source of truth for your device estate, so please export what you need to retain; see [Settings Reference](../reference/settings.md) for data retention and offboarding.

### How time credits work

* A time credit is applied at your next subscription renewal: the renewed term is extended by the granted number of days or months at no additional charge, without changing your price, the scope of your subscription, or your billing cycle. We confirm every granted credit in writing, stating the amount and the renewal it applies to.
* Time credits are not paid out in cash, cannot be converted into a monetary credit or a discount, and are not transferable.
* Time credits are capped at 8 days per calendar month and 30 days per contract year for availability, and, separately, at six months per event of total data loss.
* A time credit lapses if you do not claim it within the periods stated above, or if the subscription is not renewed. If availability is missed repeatedly, you have a termination right instead, see below.

### Repeated failure to meet the availability target

If the measured availability falls below 99.5% in three consecutive calendar months, or below 95.0% in a single calendar month, you may terminate your Autopilot Monitor Pro subscription with 30 days' notice to the end of a month. We then refund the fees you have already paid for the unused remainder of the term on a pro-rata basis. The termination right expires if you do not exercise it within 30 days after the end of the month that triggered it.

### Limits of our liability

The time credits described above, together with the termination right for repeated failure to meet the availability target, are the complete and exclusive remedy for failure to meet the availability target and for loss of data. Time credits are compensation in the form of additional service, not damages. Beyond them we accept no liability for damages arising from unavailability or data loss, including indirect and consequential damages, lost profit, business interruption, and the cost of repeating or re-analysing enrollments.

This limitation does **not** apply to liability for intent or gross negligence, for injury to life, body or health, for damages under the German Product Liability Act, for the breach of a guarantee given by us, or to any liability that cannot be limited or excluded by law. Liability for the processing of personal data is governed by the data processing agreement.

## Version and changes

* **Version:** 0.1

We may change this page, for example when we add plans or features. Changes are announced under [Service Announcements](../troubleshooting/service-announcements.md) at least 30 days before they take effect, and they apply from your next subscription renewal. The version published when you place an order or when your subscription renews is the version that applies to that term. Earlier versions are available from our support on request.
