---
description: >-
  One-time tenant setup: first sign-in, enabling Autopilot Device Validation,
  and the recommended initial configuration.
---

# Portal Setup

Before the agent can send any data to the portal, a few one-time steps are required in the portal itself.

## 1. Sign in — the first user becomes Tenant Admin

Open the portal and sign in with your Microsoft Entra ID account. The very first user to sign in for your organization is automatically granted **Tenant Admin** rights for your tenant.

{% hint style="info" %}
The Tenant Admin can later promote other users via **Settings → Access Management**. Users without a role only see the **Progress Portal** — a simplified view for tracking a specific device by serial number — and have no access to session details, diagnostics, or configuration. See [Roles & Permissions](../concepts/roles-and-permissions.md).
{% endhint %}

## 2. Enable Autopilot Device Validation

Navigate to **Settings → Configuration** and enable **Autopilot Device Validation**. This is required before the agent is permitted to send any session data — without it, all agent uploads are rejected.

{% hint style="warning" %}
**Why is this required?** The Autopilot device check ensures that only devices registered in your Intune tenant can register sessions, preventing unintended data from reaching your tenant. Enabling it requires admin consent for the Microsoft Graph permission `DeviceManagementServiceConfig.Read.All` (read-only), and it is your explicit confirmation that the agent may collect and transmit enrollment telemetry on behalf of your organization.
{% endhint %}

## 3. Ready

That's it — the portal can now receive data. [Deploy the agent](deploy-the-agent.md) via Intune, and sessions will start appearing on the dashboard as soon as devices begin enrolling.

## Recommended configuration

After the basic setup, consider enabling these features to get the most out of Autopilot Monitor:

1. **Agent Settings → Geo Location Detection + Set Timezone Automatically** — sets the device timezone correctly during enrollment. Without a location sensor, Windows defaults to Pacific Standard Time; this fixes the "every device thinks it's in Redmond" effect and feeds the Geographic Performance view.
2. **Agent Analyzers** — enable the **Local Admin Analyzer** (configure your expected admin accounts to detect unauthorized local admins), **Software Inventory**, and the **Vulnerability Analyzer** for security insights during enrollment.
3. **Diagnostics Package** — automatically gathers log files from devices for troubleshooting. This requires your own Azure Blob Storage account (Container SAS URL) as the upload destination; you keep full control of the collected data.

All settings are described in detail in the [Settings Reference](../reference/settings.md).
