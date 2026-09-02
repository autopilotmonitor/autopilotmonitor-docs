---
type: Integration Guide
tags: [notifications, channels, webhooks]
description: >-
  Enrollment notifications for Teams, Slack, Discord, or any system (e.g. Azure Logic Apps) via generic JSON
  webhook — setup per provider, triggers, and the payload reference.
---

# Notifications

Autopilot Monitor pushes enrollment events to your team the moment they happen — no one has to watch the portal. Notification channels are configured under **Settings → Tenant → Notifications**; you can define several named channels per tenant, each with its own provider.

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

### Discord

Create a webhook in your Discord channel (*Channel settings → Integrations → Webhooks → New Webhook*), copy the URL, select the *Discord* provider. Notifications render as an embed; because Discord webhooks cannot post buttons, links (e.g. to the session) appear as markdown links inside the message.

### Generic JSON webhook

Any HTTPS endpoint that accepts a JSON POST — a ticketing system, an automation platform, or an SMTP gateway (e.g. Postal) if you want notifications as email. This provider additionally supports **custom HTTP headers** (e.g. an `Authorization` header for your endpoint) and an optional **signing secret** for [HMAC request verification](#verifying-signed-requests).

#### Azure Logic Apps and Power Automate

Use the Generic JSON provider to feed an Azure Logic App or a Power Automate flow — there is no separate provider because the **When a HTTP request is received** trigger has no payload format of its own: it accepts any JSON body and lets you describe its shape on the receiving side.

1. Start the logic app or flow with the trigger **When a HTTP request is received**, leave the method at POST, and save — the trigger URL (including its `sig=` signature) is generated on save.
2. Paste that URL as the Webhook URL in Autopilot Monitor, select the *Generic JSON* provider, save, and click **Send Test Notification**. The request appears in the run history.
3. In the trigger, choose **Use sample payload to generate schema** and paste the body of that test run (or the payload from the [reference below](#generic-json-payload-reference)). The fields then show up as dynamic content in later steps; branch on `eventType` with a Switch action.

{% hint style="warning" %}
Leave the trigger's **Schema Validation** setting off, or keep the generated schema free of `required` and `additionalProperties: false`. Fields without a value are omitted from the payload, and new fields are added within the same `schemaVersion` — a strict schema would reject those requests with HTTP 400.
{% endhint %}

The trigger URL carries its own shared-access signature, so treat it like a password. The optional signing secret adds the HMAC headers described below, but Logic Apps has no built-in expression to verify an HMAC — use it only if a downstream step (e.g. an Azure Function) checks it. In Power Automate this trigger is a Premium connector; in Azure Logic Apps it is billed per execution.

## Triggers

| Toggle | Fires when |
| --- | --- |
| **Notify on Start** | An enrollment session starts |
| **Notify on Success** | A session completes successfully |
| **Notify on Failure** | A session ends in failure — keep this one on |

**Send Test Notification** posts a sample message to verify the configuration end to end.

Beyond the session triggers, the same webhook also carries **SLA breach/resolution alerts** (when [SLA targets](../portal-guide/sla-compliance.md) are configured), **consecutive-failure alerts**, and **hardware-rejection notices**.

## In-portal alerts

Some alerts are delivered as **bell notifications** in the portal header rather than through the webhook — they are about your configuration or your hardware, not about a single enrollment, and they are raised once per subject instead of per event:

| Alert | Raised when | Seen by |
| --- | --- | --- |
| **Rule fires more often than usual** | An analyze rule's [regression detection](../rules/analyze-rules/README.md#regression-detection) flags a sustained, statistically separated increase in its hit rate. Links straight to the rule card. | Tenant Admins |
| **App version installs slower** | A newly rolled-out app version's median install time (the final install attempt) rises sharply above the previous version's — at least twice as long and 5+ minutes more, with enough measured installs on both sides to be meaningful. Links to the [app's detail page](../portal-guide/software-inventory-and-vulnerabilities.md#per-app-deep-dive). Raised once per (app, version). | Tenant Admins |
| **Device with an incompatible TPM** | A device's TPM cannot perform the signature Windows requires for the agent's certificate authentication, so the agent can never authenticate from it. The fix is on your side — a TPM firmware update or device replacement. | Tenant Admins |
| **Hardware rejection** | A device was refused because it is outside the tenant's [Hardware Whitelist](../reference/settings.md#hardware-whitelist). Also sent to the webhook, if one is configured. | Tenant Admins |
| **SLA breach / resolution, consecutive failures** | Same events as the webhook alerts above. | All tenant members |

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

### Verifying signed requests

If you configure a **Signing Secret** on a generic JSON channel, every request carries two extra headers so your endpoint can verify it really came from Autopilot Monitor and was not tampered with:

| Header | Content |
| --- | --- |
| `X-AutopilotMonitor-Timestamp` | Unix timestamp (seconds) the request was signed at |
| `X-AutopilotMonitor-Signature` | `sha256=` followed by the lowercase hex HMAC-SHA256 of `"{timestamp}.{rawBody}"`, keyed with your signing secret |

To verify: concatenate the timestamp header, a `.`, and the **raw request body** (before any JSON parsing), compute HMAC-SHA256 with your secret, and compare against the signature header using a constant-time comparison. Reject requests whose timestamp is older than a few minutes to prevent replay.

```powershell
function Test-AutopilotMonitorSignature {
    param(
        [string]$Secret,           # the channel's signing secret
        [string]$Timestamp,        # X-AutopilotMonitor-Timestamp header value
        [byte[]]$RawBody,          # raw request body bytes, before any JSON parsing
        [string]$SignatureHeader   # X-AutopilotMonitor-Signature header value
    )
    $hmac = [System.Security.Cryptography.HMACSHA256]::new([Text.Encoding]::UTF8.GetBytes($Secret))
    $message = [Text.Encoding]::UTF8.GetBytes("$Timestamp.") + $RawBody
    $expected = 'sha256=' + (($hmac.ComputeHash($message) | ForEach-Object { $_.ToString('x2') }) -join '')
    [System.Security.Cryptography.CryptographicOperations]::FixedTimeEquals(
        [Text.Encoding]::UTF8.GetBytes($expected),
        [Text.Encoding]::UTF8.GetBytes($SignatureHeader))
}
```

The snippet targets PowerShell 7; on Windows PowerShell 5.1, replace the `FixedTimeEquals` call with `$expected -ceq $SignatureHeader` (a plain case-sensitive comparison, without the timing-attack hardening).

The signature covers the exact bytes of the body — verify before re-serializing the JSON, since any reformatting changes the digest.
