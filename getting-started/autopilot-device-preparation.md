---
type: Feature Guide
description: >-
  Monitor Windows Autopilot Device Preparation enrollments — why the flow is
  different, how to deliver the agent early via the MSI line-of-business app,
  and how to validate devices with device association
tags:
  - device-preparation
  - autopilot
  - deployment
  - device-validation
  - msi
---

# Autopilot Device Preparation

Autopilot Monitor supports **Windows Autopilot Device Preparation** — Microsoft's newer enrollment flow, often referred to as Autopilot v2. Sessions are detected as Device Preparation automatically, monitored end to end, and shown with their own phase model in the portal.

{% hint style="info" %}
Device Preparation support is **young and actively evolving**. If you monitor Device Preparation enrollments and something is missing, confusing, or broken, please [open a GitHub issue](https://github.com/okieselbach/AutopilotMonitor/issues) — feedback from real environments directly shapes what gets built next.
{% endhint %}

## Why Device Preparation is different

Two structural differences matter for monitoring:

1. **Resources install in three fixed phases.** There is no classic Enrollment Status Page. The Device Preparation page works through the resources selected in the Device Preparation policy in this order:
   1. policies, line-of-business apps and **Microsoft 365 Apps**
   2. **PowerShell scripts**
   3. **Win32, Microsoft Store and Enterprise App Catalog apps**

   Each phase starts only after the previous one succeeded. An agent delivered by a platform script therefore arrives after Microsoft 365 Apps have installed.
2. **Devices are never Autopilot-registered.** Device Preparation does not use Windows Autopilot device identities, so [Autopilot Device Validation](../reference/settings.md#enrollment-device-validation) can never match these devices. Validate them through **device association** or **Corporate Identifiers** instead — see below.

## Deploy the agent as an MSI line-of-business app

To get the agent onto the device in the first phase, Autopilot Monitor provides a small MSI that Intune delivers over the MDM channel. As a line-of-business app it installs before Microsoft 365 Apps, so the agent monitors all three phases.

The MSI contains no agent logic: it is packaging around the same signed loader used for the platform script, which downloads the current bootstrapper, verifies our signature on it, and runs it. It passes through exactly the same [pre-requisite guards](deploy-the-agent.md#safe-to-assign-broadly) and always installs the current agent, so unlike an uploaded script copy there is nothing in Intune to keep up to date.

1. **Download the MSI:** [`AutopilotMonitor-Bootstrap.msi`](https://download.autopilotmonitor.com/agent/AutopilotMonitor-Bootstrap.msi). The package is Authenticode-signed by glueckkanja AG and carries a build provenance attestation — check either with `Get-AuthenticodeSignature` or `gh attestation verify AutopilotMonitor-Bootstrap.msi --repo okieselbach/AutopilotMonitor` before you upload it.
2. **Add it in Intune:** in the **Microsoft Intune admin center**, go to **Apps → Windows → + Add**, choose the app type **Line-of-business app**, and upload the MSI.
3. **Assign it as Required to the right group:** target the **device group configured in your Device Preparation policy** (the group devices are joined to at enrollment time — Intune's _enrollment time grouping_). Because the device becomes a member of that group during enrollment, a Required assignment to exactly this group reaches the device in its very first sync.

{% hint style="warning" %}
The group assignment is the part that goes wrong most often: assigning the MSI to any other device group means it is not targeted at enrollment time and installs too late — use the same group that is selected in the Device Preparation policy itself.
{% endhint %}

## Alternative: the platform script

The platform script from [Deploy the Agent](deploy-the-agent.md) works with Device Preparation too, with less coverage. It runs in the script phase, so the agent misses the installation of line-of-business apps and Microsoft 365 Apps. It monitors the remaining scripts, the Win32 and Store app phase and the rest of setup.

To run during setup, the script must be assigned to the Device Preparation device group **and** selected in the Device Preparation policy. A script that is only assigned runs after the deployment has finished.

You can keep both channels assigned. The deployment marker guarantees that only one of them installs the agent.

## Device association

[Windows Autopilot device association](https://learn.microsoft.com/autopilot/device-preparation/device-association/overview) binds a device to your tenant _before_ it enrolls: you pre-associate the device in Intune, a TPM-backed marker is written to the device's UEFI, and Intune marks the device corporate-owned on its own — no corporate identifier upload needed. Autopilot Monitor supports it since Microsoft's release of the feature:

1. **Associate your devices** in Intune under **Devices → Enrollment → Device association**, as described in Microsoft's guide. The **Devices** tab there lists every pre-associated and associated device with its state and assigned Device Preparation policy.
2. **Turn on the validation:** in **Settings → Enrollment Device Validation**, enable **Device Association Validation**. Devices are matched against that list by serial number. It uses the same read-only Graph permission as Autopilot Device Validation — no additional consent.

Sessions accepted this way show **Device Association** as the validation method in the session details.

## Corporate Identifiers

If you do not use device association, register the devices as **corporate device identifiers** and let Autopilot Monitor validate against those:

1. **Upload identifiers to Intune** under **Devices → Enrollment → Corporate device identifiers**, as a CSV of type **manufacturer, model and serial number**. Use this CSV type — the manually enterable serial-number-only identifier is not the supported path for Windows devices.
2. **Turn on the validation:** in **Settings → Enrollment Device Validation**, enable **Corporate Identifier Validation**. It uses the same read-only Graph permission as Autopilot Device Validation — no additional consent.

Formatting differences between your CSV and what the device reports (upper/lower case, hyphens in serial numbers) are handled: values are matched the way Intune normalizes them.

## What you see in the portal

* Sessions carry a **Device Preparation** label and can be filtered by enrollment type in session search.
* The timeline follows the Device Preparation flow rather than ESP phases.
* The [Progress Portal](../portal-guide/progress-portal.md) shows Device Preparation devices with the same shorter flow, including live app progress — end users and helpdesk staff can follow a device by serial number without a portal role.
* The end-of-enrollment summary includes the apps delivered during Device Preparation — including **Microsoft 365 Apps**, which installs through its own MDM channel rather than the Intune Management Extension.

## Limitations

* The short window between enrollment start and the MSI installing (the first sync) is not yet captured live; coverage of that early window is being improved.
* Apps that are _not_ part of the Device Preparation policy install after the user reaches the desktop, like on any Intune-managed device. The agent keeps tracking those installs for as long as it runs, but a machine can genuinely still be "finishing up" after the session completes.
