---
type: How-to Guide
tags: [requirements, permissions, onboarding]
description: What you need before you start, and how to get access during the Private Preview.
---

# Requirements & Access

## Requirements

Autopilot Monitor plugs into an existing Windows Autopilot environment. You need:

* **Microsoft Intune** with **Windows Autopilot** configured — devices are registered as Autopilot devices and enroll via OOBE.
* **Microsoft Entra ID accounts** for portal sign-in. The portal uses your existing Entra ID identity; there are no separate credentials.
* **Windows 10 or Windows 11 (x64)** client devices enrolling through Autopilot. Both user-driven and pre-provisioning (White Glove) scenarios are supported.
* **Outbound HTTPS (TLS 1.2+)** from enrolling devices to the Autopilot Monitor backend and the agent download endpoint. No inbound connectivity is required — the agent uses standard HTTPS on port 443. For the exact hosts to allow on a firewall, proxy, or gateway, see [Network Endpoints](../reference/network-endpoints.md).
* Permission to create a **platform script** in the Microsoft Intune admin center (for agent deployment).

{% hint style="info" %}
The agent authenticates with the device's **MDM client certificate** (issued by Intune during enrollment). If your network uses TLS inspection, exclude the Autopilot Monitor backend endpoints from break-and-inspect — client-certificate authentication does not survive TLS interception.
{% endhint %}

## Private Preview access

Autopilot Monitor is currently in **Private Preview**. The service is available to a limited number of organizations while the core functionality is refined. Access is invite-only and managed per tenant.

### How to request access

1. Open the [Autopilot Monitor portal](https://www.autopilotmonitor.com) and sign in with your Microsoft Entra ID account.
2. If your organization has not been granted access yet, you will see a **Private Preview** waitlist screen — signing in automatically places your tenant on the waitlist.
3. Request activation via [LinkedIn](https://www.linkedin.com/in/oliver-kieselbach) or by opening a [GitHub issue](https://github.com/okieselbach/Autopilot-Monitor/issues). Tenants are enabled manually; requests are reviewed regularly.
4. Once approved, sign in again — you will be taken straight to the portal. Continue with [Portal Setup](portal-setup.md).

{% hint style="warning" %}
**Expect instability during Private Preview.** The backend, portal, and agent are under active development and receive frequent updates. Availability is not guaranteed, features and data structures may change, and session data may be cleared between major updates. Plan for these realities when relying on it during the preview.
{% endhint %}
