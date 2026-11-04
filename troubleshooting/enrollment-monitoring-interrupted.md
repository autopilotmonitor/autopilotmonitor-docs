---
type: Troubleshooting
tags: [troubleshooting, entra-id, consent, device-validation]
description: >-
  The portal says enrollment monitoring is interrupted: Microsoft Entra rejects
  Autopilot Monitor's access to your tenant. What it means, how to restore
  access, and what to check afterwards.
---

# Enrollment Monitoring Is Interrupted

The **Dashboard** shows a red notice, **Action required: Enrollment monitoring is interrupted**. **Settings → Autopilot Validation** shows the same notice with a **Restore access** button.

## What the notice means

Before Autopilot Monitor accepts data from a device, it checks with Microsoft Graph that the device belongs to your tenant (see [Enrollment Device Validation](../reference/settings.md#enrollment-device-validation)). It does that through the Autopilot Monitor enterprise application in your tenant. Microsoft Entra currently refuses that access, so devices can't be validated.

Your devices keep enrolling as usual, because Autopilot Monitor only watches. What stops is the monitoring: new enrollments are not monitored until access is back.

## When it appears

Only when the problem lasts. A single failed check shows nothing, and neither does a short disturbance at Microsoft. The notice appears once Microsoft Entra still refuses access at least 15 minutes after the first refusal.

The check runs when a device connects. If no device is connecting, Autopilot Monitor asks again by itself about every 2 hours, so the notice can take that long to appear.

## Restore access

Open **Settings → Autopilot Validation**. The notice there shows the error code and when Autopilot Monitor first noticed the refusal. What to do depends on the code:

| Code | What happened | What to do |
| --- | --- | --- |
| `AADSTS7000229`, `AADSTS700016` | The Autopilot Monitor enterprise application is not in your tenant. Usually someone deleted it in the Microsoft Entra admin center. | Click **Restore access** and accept the Microsoft consent prompt. This creates the application again with its default permission. |
| `NoApplicationRole` | The enterprise application is there, but holds no permission. That is what remains when it was deleted and a user then accepted the sign-in prompt, which grants sign-in only. | Click **Restore access** and accept the Microsoft consent prompt. |
| `AADSTS7000112` | The enterprise application is disabled in your tenant. | In the Microsoft Entra admin center, open **Enterprise applications → Autopilot Monitor → Properties** and set **Enabled for users to sign-in?** to **Yes**. Then click **Detect existing access**. |
| `AADSTS53003` | A Conditional Access policy blocks the application. | Change the policy so it no longer blocks Autopilot Monitor. Then click **Detect existing access**. |
| Any other code | | Microsoft explains each code at [login.microsoftonline.com/error](https://login.microsoftonline.com/error). If that does not get you further, write to **support@autopilotmonitor.com** and include the code. |

The Microsoft consent prompt needs the **Global Administrator** or **Privileged Role Administrator** role in Microsoft Entra. The **Restore access** button is available to Tenant Admins. If you don't hold one of those Entra roles, ask someone who does, and add them under **Settings → Access Management** if they have no portal role yet.

**Detect existing access** checks right away whether the access works again, without a consent prompt. Neither button changes your validation settings.

{% hint style="info" %}
A fresh consent can take a minute to propagate on Microsoft's side. The portal retries the check automatically. If it still reports a problem, wait a minute and click **Detect existing access**.
{% endhint %}

## After access is back

* **Monitoring resumes on its own.** Agents that are still running send what they collected, and new enrollments are monitored again. An enrollment that ended during the interruption stays missing or incomplete in the portal.
* **Grant optional add-on permissions again.** If the enterprise application was deleted, the [optional Graph permissions](../reference/optional-graph-permissions.md) granted on it are gone too, for example `CloudPC.Read.All` or `DeviceManagementManagedDevices.Read.All`. The consent does not bring them back. Until you grant them again, devices that only pass through such a validation are not accepted, and Settings shows **Permission missing** below that validation's switch. See [Device Validation: Permission Missing](device-validation-permission-missing.md).
* **Check settings you made on the enterprise application.** A deleted application also loses its **Assignment required** setting and its app role assignments. See [settings on the enterprise application](app-registration-migration.md#before-you-migrate-settings-on-the-enterprise-application).

## How long the notice stays after the repair

* **After the consent in the portal:** It goes away in Settings right after the check succeeds. The Dashboard can take a few more minutes.
* **After a repair in the Microsoft Entra admin center:** It goes away after the next device connects, or when you click **Detect existing access**. If you do neither and no device connects, it can take up to about 2 hours. The Dashboard can take a few more minutes here too.

## If your tenant still ran on the previous app

If your tenant had not yet moved to the new app registration and the previous application (`1a400946-…`) was deleted, use the same **Restore access** button. The consent goes to the new application, and your tenant switches to it automatically. See [App Registration Migration](app-registration-migration.md).

## If you removed the application on purpose

If your organization no longer uses Autopilot Monitor, remove the agent's platform script from Intune. Otherwise new devices keep installing the agent, which keeps trying to connect.

To delete your data as well, use **Offboard Tenant** in the [Danger Zone](../reference/settings.md#danger-zone) settings. Signing in for this creates the enterprise application again with the sign-in permission only, so delete it after offboarding.
