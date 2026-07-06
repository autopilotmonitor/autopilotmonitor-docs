---
description: >-
  Enrollment notifications for Teams, Slack, or any system via generic JSON
  webhook — setup per provider, triggers, and the payload reference.
---

# Notifications

Autopilot Monitor pushes enrollment events to your team the moment they happen — no one has to watch the portal. One webhook is configured per tenant under **Settings → Tenant → Notifications**, with a choice of provider.

## Providers

### Microsoft Teams — Workflow Webhook *(recommended)*

1. In Teams, open the target channel → **Manage channel** → **Workflows**.
2. Add the template *"Post to a channel when a webhook request is received"* and copy the generated URL.
3. Paste it as the Webhook URL in Autopilot Monitor and select the *Teams (Workflow Webhook)* provider.

Workflow webhooks are free and don't require a Power Automate Premium license.

### Microsoft Teams — Legacy Connector *(deprecated)*

The legacy Office 365 Connector (MessageCard) format. Microsoft has deprecated this method — existing configurations keep working, but switch to Workflow Webhooks when you can.

### Slack

Create an **Incoming Webhook** in your Slack workspace (*Apps → Incoming Webhooks → Add*, pick the channel), copy the URL, select the *Slack* provider.

### Generic JSON webhook

Any HTTPS endpoint that accepts a JSON POST — a ticketing system, an automation platform, or an SMTP gateway (e.g. Postal) if you want notifications as email. This provider additionally supports **custom HTTP headers** (e.g. an `Authorization` header for your endpoint).

## Triggers

| Toggle | Fires when |
| --- | --- |
| **Notify on Start** | An enrollment session starts |
| **Notify on Success** | A session completes successfully |
| **Notify on Failure** | A session ends in failure — keep this one on |

**Send Test Notification** posts a sample message to verify the configuration end to end.

Beyond the session triggers, the same webhook also carries **SLA breach/resolution alerts** (when [SLA targets](../portal-guide/sla-compliance.md) are configured), **consecutive-failure alerts**, and **hardware-rejection notices**.

## Generic JSON payload reference

The generic provider sends a stable, versioned payload — `schemaVersion` is only bumped on breaking changes:

```json
{
  "schemaVersion": "1.0",
  "eventType": "enrollment_failed",
  "severity": "Error",
  "title": "Enrollment failed",
  "summary": "…",
  "themeColor": "D13438",
  "primaryUrl": "https://…/sessions/{id}",
  "facts":    [ { "name": "Device", "value": "…" } ],
  "sections": [ { "title": "…", "text": "…" } ],
  "actions":  [ { "type": "openUrl", "title": "…", "url": "…" } ]
}
```

* `eventType` values: `enrollment_started`, `enrollment_succeeded`, `enrollment_failed`, `preprovisioning_resumed`, `preprovisioning_completed`, `preprovisioning_failed`, `sla_breach`, `sla_resolved`, `consecutive_failures`, `hardware_rejected`, `test`.
* `severity` is one of `Info`, `Success`, `Warning`, `Error`.
* `primaryUrl` is the first link action — a session link for enrollment alerts, a portal link otherwise; inspect `actions[]` (each carries a title) to disambiguate. Fields without a value are omitted rather than sent as `null`.

{% hint style="info" %}
**Routing tip:** key your automation on `eventType`, not on `title` — titles are human-facing and may be refined over time; `eventType` and the schema are the contract.
{% endhint %}
