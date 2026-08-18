---
type: How-to Guide
tags: [requirements, permissions, onboarding]
description: What you need before you start, and how tenant activation works.
---

# Requirements & Access

## Requirements

Autopilot Monitor plugs into an existing Windows Autopilot environment. You need:

* **Microsoft Intune** with **Windows Autopilot** configured — devices are registered as Autopilot devices and enroll via OOBE.
* **Microsoft Entra ID accounts** for portal sign-in. The portal uses your existing Entra ID identity; there are no separate credentials.
* **Windows 10 or Windows 11 (x64)** client devices enrolling through Intune — user-driven, pre-provisioning (White Glove), self-deploying, and Device Preparation flows, with Entra join or Hybrid join. See the [supported enrollment scenarios](how-it-works.md#supported-enrollment-scenarios).
* **Outbound HTTPS (TLS 1.2+)** from enrolling devices to the Autopilot Monitor backend and the agent download endpoint. No inbound connectivity is required — the agent uses standard HTTPS on port 443. For the exact hosts to allow on a firewall, proxy, or gateway, see [Network Endpoints](../reference/network-endpoints.md).
* Permission to create a **platform script** in the Microsoft Intune admin center (for agent deployment).

{% hint style="info" %}
The agent authenticates with the device's **MDM client certificate** (issued by Intune during enrollment). If your network uses TLS inspection, exclude the Autopilot Monitor backend endpoints from break-and-inspect — client-certificate authentication does not survive TLS interception.
{% endhint %}

## Getting access — tenant activation

Autopilot Monitor is **publicly available** — the Community plan is free, and there is no signup form. Every new organization goes through a short activation step after its first sign-in.

### How it works

1. Open the [Autopilot Monitor portal](https://www.autopilotmonitor.com) and sign in with your Microsoft Entra ID account.
2. On the first sign-in from a new organization you will see an **activation** screen — your tenant is queued for activation automatically; there is nothing to fill in.
3. Activation usually completes within a couple of minutes. The activation screen checks automatically and takes you straight into the portal; you can also leave an email address to be notified.
4. Once activated, continue with [Portal Setup](portal-setup.md). If activation takes longer than expected, reach out via [LinkedIn](https://www.linkedin.com/in/oliver-kieselbach) or a [GitHub issue](https://github.com/okieselbach/Autopilot-Monitor/issues).

{% hint style="warning" %}
**Expect frequent change.** The backend, portal, and agent are under continuous, active development and receive frequent updates. On the Community plan, availability is not guaranteed, features and data structures may change, and session data may be cleared between major updates. Plan for these realities — production use is fine with that trade-off understood.
{% endhint %}
