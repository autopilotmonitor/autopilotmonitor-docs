---
type: Troubleshooting
tags: [troubleshooting, device-validation, graph-permissions, intune, windows-365]
description: >-
  Cloud PC or Intune Enrollment Validation is on, but its optional Graph
  permission is not granted. What the notice means, and how to grant the
  permission or turn the validation off.
---

# Device Validation: Permission Missing

Two options under **Settings → Autopilot Validation** need an optional Microsoft Graph permission that the normal admin consent does not include:

| Validation | Permission | Feature in the grant script |
| --- | --- | --- |
| Windows 365 Cloud PC Validation | `CloudPC.Read.All` | `W365CloudPcValidation` |
| Intune Enrollment Validation | `DeviceManagementManagedDevices.Read.All` | `IntuneDeviceBinding` |

You can turn them on before the permission is granted. Until it is granted, the validation accepts no device.

## What you see

* **In Settings → Autopilot Validation:** As soon as one of these validations is on without its permission, a **Permission missing** notice appears below its switch, and the section's status reads **Permission missing**. Once devices have been affected for a while, the notice also says since when.
* **On the Dashboard:** A red notice appears only when devices were actually affected: **Action required: Some devices aren't being monitored**. If the affected validation is the only one you have turned on, it reads **Action required: Enrollment monitoring is interrupted**.

## What it means for your devices

* **Intune Enrollment Validation:** Devices that none of your other validations accept are held back and not monitored. Devices another validation accepts are not affected.
* **Windows 365 Cloud PC Validation:** Cloud PCs can't be validated, so they are rejected and not monitored. Other devices are not affected.

Your devices keep enrolling as usual, because Autopilot Monitor only watches. What is missing is the monitoring of the affected devices.

## When the Dashboard notice appears

Only when the missing permission has consequences and lasts. A device must have been kept out because of it, and the permission must still be missing at least 15 minutes later. Autopilot Monitor checks again when the next device connects, and by itself about every 2 hours.

## Grant the permission

1. Click **Grant permission** in the notice. It opens **Settings → Optional Graph capabilities** with the feature already ticked.
2. Copy the command and run it in Azure Cloud Shell or in PowerShell. Granting needs the **Global Administrator** or **Privileged Role Administrator** role in Microsoft Entra. See [Optional Graph Permissions](../reference/optional-graph-permissions.md).
3. Click **Refresh permission status**.

You don't need to change anything else. A device that connects after the grant is accepted, usually within a few minutes.

## Or turn the validation off

If you don't need the validation, click **Turn off** in the notice, or use its switch. This needs the Tenant Admin role in the portal.

{% hint style="warning" %}
If it is the only validation you have turned on, the confirmation warns you: with no validation enabled, Autopilot Monitor rejects all agent requests, and the Dashboard shows **Action required: No device validation is enabled**. Turn on another validation in that case, for example Autopilot Device Validation.
{% endhint %}

## How long the notice stays

* **After you turn the validation off:** The Dashboard notice goes away within a few minutes.
* **After the grant:** The notice in Settings goes away once the permission shows as granted. The Dashboard notice goes away after the next device connects, or at the latest after about 2 hours.
