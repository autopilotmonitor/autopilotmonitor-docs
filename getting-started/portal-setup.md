---
type: How-to Guide
tags: [portal, setup, onboarding]
description: >-
  One-time tenant setup: the first sign-in that makes you Tenant Admin, the
  setup guide on the dashboard, and the same steps done one by one.
---

# Portal Setup

Before the agent can send any data to the portal, a few one-time steps are required in the portal itself. The **setup guide** walks you through them in a few minutes, Intune included. You can also do each step on your own; see [Setting up without the guide](#setting-up-without-the-guide).

## Sign in: the first user becomes Tenant Admin

Open the portal and sign in with your Microsoft Entra ID account. The very first member of your organization to sign in is automatically granted **Tenant Admin** rights for your tenant; a guest account never is.

{% hint style="info" %}
The Tenant Admin can later promote other users via **Settings → Access Management**. Users without a role only see the **Progress Portal** — a simplified view for tracking a specific device by serial number — and have no access to session details, diagnostics, or configuration. See [Roles & Permissions](../concepts/roles-and-permissions.md) and, for colleagues who sign in later, [Signed in, but only the Progress Portal](../troubleshooting/progress-portal-only.md).
{% endhint %}

## The setup guide

As long as your tenant has no enrollment session, the dashboard shows a **Get started** checklist with four steps, and Tenant Admins find a **Setup guide** entry in the sidebar under the dashboard. Click **Start guided setup** (or **Continue setup**) to open the guide at the first open step. Tenant Admins can open it any time from the help menu (**?** → **Setup guide**).

| Step | | What it does |
| --- | --- | --- |
| Apply recommended settings | Recommended | Timezone, diagnostics package, software inventory. |
| Check your hardware | Recommended | Makes sure the brands and virtual machines you enroll are on the hardware whitelist. |
| Turn on device validation | **Required** | Without a validation method the backend rejects all agent data. |
| Deploy the agent with Intune | **Required** | One platform script; the step is done when the first enrollment arrives. |

A step counts as done when its setting is in place, also when you set it up without the guide. Every setting the guide changes stays under **Settings**, where you can change it again.

Tenant Admins complete the steps. Operators and Viewers see the checklist read-only.

### Step 1: Recommended settings

Three settings that get more out of every enrollment. All are checked; uncheck what you don't want, then click **Apply**. Settings that are already as recommended show **Already on** and stay untouched. **Show me where** opens the setting on its own page and highlights it.

* **Set the device timezone automatically** — the agent sets the timezone from the device's IP location, so enrollment timelines and logs show local time. It turns **Geo-Location Detection** on as well, if it was off. Leave it off if your devices go online through a proxy in another country.
* **Upload diagnostics packages** — the device's logs are attached to the session. Choose when: **When an enrollment fails** (Upload Mode *On Failure Only*, preselected) or **After every enrollment** (*Always*). Without a storage account of your own, the packages go to the built-in hosted storage, and the guide says so before you apply it: they leave your Azure tenant and are stored per tenant in the Autopilot Monitor backend. If you already set up your own storage account, it stays the destination.
* **Software inventory and vulnerability analyzer** — lists the software installed at enrollment start and end and matches it against known vulnerabilities.

**Skip this step** leaves everything as it is.

### Step 2: Check your hardware

Monitoring data is only accepted from manufacturers on the [Hardware Whitelist](../reference/settings.md#hardware-whitelist). The default list is `Dell*`, `HP*`, `Lenovo*` and `Microsoft Corporation`. Virtual machines report their own manufacturer, so add them if you test with them. The guide adds VMware, QEMU, Proxmox and Parallels with one click each, and the same for ASUS, Samsung, Acer, Fujitsu and Panasonic. Hyper-V virtual machines report `Microsoft Corporation` and are already covered. **Accept every manufacturer (\*)** turns the check off. Models and the full list stay under **Settings → Hardware Whitelist**.

### Step 3: Turn on device validation

The guide opens **Settings → Autopilot Validation** and highlights **Autopilot Device Validation**: the backend then only accepts data from devices registered as Windows Autopilot devices in your tenant. Click **Turn on and approve**. Microsoft asks you to approve one read-only permission (`DeviceManagementServiceConfig.Read.All`); afterwards you are back on the page and the setting is saved.

* **Using Autopilot device preparation?** Click **I use device preparation**: the guide highlights **Corporate Identifier Validation** and **Device Association Validation** instead. Turn on the one that matches the list you keep in Intune. Both use the same permission and the same approval. See [Autopilot Device Preparation](autopilot-device-preparation.md).
* **Can't approve it yourself?** Only a **Global Administrator** or **Privileged Role Administrator** can grant the permission. **Copy approval link** copies a link you can send to one of them. Once they approved it, click **Detect existing access** — with device preparation, the one under the validation you use — and the setting is turned on without a second approval.

### Step 4: Deploy the agent with Intune

The guide shows the deployment in four steps: download the loader script `Start-AutopilotMonitor.ps1`, add it as a platform script in Intune, assign it to your Autopilot devices (it offers the membership rule of a dynamic device group to copy), and enroll a test device. The full walkthrough with all script settings is [Deploy the Agent](deploy-the-agent.md). With Autopilot device preparation, deploy the agent as an MSI line-of-business app instead; the guide points there when only device-preparation validation is on.

After **Done, wait for my device**, the checklist waits for your first enrollment and updates as soon as a device starts enrolling. From the moment device validation is on, it also lists devices the backend turned away, with the reason:

* **Manufacturer not on your hardware whitelist** — **Accept this manufacturer** adds it to the whitelist; the next enrollment of such a device is accepted.
* **Not registered as an Autopilot device in this tenant** — check the device in Intune, or the validation method you chose.

**No device after an hour?** The checklist lists what to check; [No sessions appear in the portal](../troubleshooting/common-problems.md#no-sessions-appear-in-the-portal) has the details.

When the first enrollment arrives, the checklist gives way to **Your first enrollment is here** with a link to the session. The setup guide stays in the help menu. Once every step is done it shows **Setup is complete**; **Go through the steps again** starts it at step 1 with the settings your tenant has now.

### Hiding the checklist

**Hide** removes the checklist and the sidebar entry for everyone in your tenant. To bring them back, a Tenant Admin opens **?** → **Setup guide** and clicks **Show it there again**. After the first enrollment the checklist does not come back.

## Setting up without the guide

The same steps, done on the settings pages directly.

### Enable device validation

Open **Settings → Autopilot Validation** and enable **Autopilot Device Validation** — or, with Autopilot device preparation, **Corporate Identifier Validation** or **Device Association Validation**. At least one validation method is required before the agent is permitted to send any session data; without it, all agent uploads are rejected.

{% hint style="warning" %}
**Why is this required?** The device check ensures that only devices registered in your Intune tenant can register sessions, preventing unintended data from reaching your tenant. Enabling it requires admin consent for the Microsoft Graph permission `DeviceManagementServiceConfig.Read.All` (read-only), and it is your explicit confirmation that the agent may collect and transmit enrollment telemetry on behalf of your organization.

Granting this consent requires an Entra ID role that can consent tenant-wide to Microsoft Graph application permissions — **Global Administrator** or **Privileged Role Administrator**. If your account lacks these rights, ask someone who has them to run the consent, then use **Detect existing access** in the portal to activate the feature without repeating the consent redirect.
{% endhint %}

### Recommended configuration

Consider enabling these features to get the most out of Autopilot Monitor:

1. **Tenant → Contact** — set an address we can reach you on if something about your tenant needs attention. A shared team mailbox works better than a personal one. It is used for service matters only, never for marketing.
2. **Agent Settings → Geo Location Detection + Set Timezone Automatically** — sets the device timezone correctly during enrollment. Without a location sensor, Windows defaults to Pacific Standard Time; this fixes the "every English device thinks it's in Redmond" effect and feeds the Geographic Performance view.
3. **Agent Analyzers** — enable the **Local Admin Analyzer** (configure your expected admin accounts to detect unauthorized local admins), **Software Inventory**, and the **Vulnerability Analyzer** for security insights during enrollment.
4. **Diagnostics Package** — automatically gathers log files from devices for troubleshooting. You choose the upload destination: your own Azure Blob Storage (Container SAS URL — data never leaves your tenant) or the built-in hosted storage (no storage account needed). Upload Mode *On Failure Only* keeps the IME and agent logs of every failed enrollment, the raw evidence an AI investigation needs when no rule explains a failure.
5. **Tenant → Hardware Whitelist** — add the manufacturers you enroll beyond Dell, HP, Lenovo and Microsoft, including the virtual machines you test with.

All settings are described in detail in the [Settings Reference](../reference/settings.md).

### Deploy the agent

[Deploy the agent](deploy-the-agent.md) via Intune, and sessions will start appearing on the dashboard as soon as devices begin enrolling.
