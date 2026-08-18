---
type: Feature Guide
tags: [device-preparation, autopilot, deployment, device-validation, msi]
description: >-
  Monitor Windows Autopilot Device Preparation enrollments — why the flow is
  different, how to deliver the agent early via the MSI line-of-business app,
  and how to validate devices with Corporate Identifiers.
---

# Autopilot Device Preparation

Autopilot Monitor supports **Windows Autopilot Device Preparation** — Microsoft's newer enrollment flow, often referred to as Autopilot v2. Sessions are detected as Device Preparation automatically, monitored end to end, and shown with their own phase model in the portal.

{% hint style="info" %}
Device Preparation support is **young and actively evolving**. If you monitor Device Preparation enrollments and something is missing, confusing, or broken, please [open a GitHub issue](https://github.com/okieselbach/AutopilotMonitor/issues) — feedback from real environments directly shapes what gets built next.
{% endhint %}

## Why Device Preparation is different

Two structural differences matter for monitoring:

1. **The processing order is reversed.** There is no classic Enrollment Status Page. The Device Preparation page works through the resources selected in the Device Preparation policy in a fixed order: **apps first, then PowerShell scripts**. A PowerShell script therefore runs only *after* the app phase — delivered that way, the agent would arrive too late to watch the apps install, and if the app phase fails hard, the script channel may never run at all.
2. **Devices are never Autopilot-registered.** Device Preparation does not use Windows Autopilot device identities, so [Autopilot Device Validation](../reference/settings.md#enrollment-device-validation) can never match these devices. **Corporate Identifiers** are the intended validation path — see below.

## Deploy the agent as an MSI line-of-business app

To get the agent onto the device *before* the app phase, Autopilot Monitor provides a small MSI that Intune delivers over the MDM channel. Line-of-business apps install with the first Intune sync — early enough to monitor the Device Preparation app phase from the start.

The MSI contains no agent logic: it is a thin runner that downloads and executes the current server-hosted bootstrap script. It passes through exactly the same [pre-requisite guards](deploy-the-agent.md#safe-to-assign-broadly) and always installs the current agent, so unlike an uploaded script copy there is nothing in Intune to keep up to date.

1. **Download the MSI:** [`AutopilotMonitor-Bootstrap.msi`](https://download.autopilotmonitor.com/agent/AutopilotMonitor-Bootstrap.msi). Every build is published with a signed provenance attestation — verify with `gh attestation verify AutopilotMonitor-Bootstrap.msi --repo okieselbach/AutopilotMonitor` if you want to check what you are uploading.
2. **Add it in Intune:** in the **Microsoft Intune admin center**, go to **Apps → Windows → + Add**, choose the app type **Line-of-business app**, and upload the MSI.
3. **Assign it as Required to the right group:** target the **device group configured in your Device Preparation policy** (the group devices are joined to at enrollment time — Intune's *enrollment time grouping*). Because the device becomes a member of that group during enrollment, a Required assignment to exactly this group reaches the device in its very first sync.

{% hint style="warning" %}
The group assignment is the part that goes wrong most often: assigning the MSI to any other device group means it is not targeted at enrollment time and installs too late — use the same group that is selected in the Device Preparation policy itself.
{% endhint %}

You can keep the platform script from [Deploy the Agent](deploy-the-agent.md) assigned as well — the deployment marker guarantees that only one channel ever installs the agent, so the two coexist safely.

## Enable Corporate Identifier validation

Because Device Preparation devices are never Autopilot-registered, register them as **corporate device identifiers** and let Autopilot Monitor validate against those:

1. **Upload identifiers to Intune** under **Devices → Enrollment → Corporate device identifiers**, as a CSV of type **manufacturer, model and serial number**. Use this CSV type — the manually enterable serial-number-only identifier is not the supported path for Windows devices.
2. **Turn on the validation:** in **Settings → Enrollment Device Validation**, enable **Corporate Identifier Validation**. It uses the same read-only Graph permission as Autopilot Device Validation — no additional consent.

Formatting differences between your CSV and what the device reports (upper/lower case, hyphens in serial numbers) are handled: values are matched the way Intune normalizes them.

## What you see in the portal

* Sessions carry a **Device Preparation** label and can be filtered by enrollment type in session search.
* The timeline follows the Device Preparation flow rather than ESP phases.
* The end-of-enrollment summary includes the apps delivered during Device Preparation — including **Microsoft 365 Apps**, which installs through its own MDM channel rather than the Intune Management Extension.

## Limitations

* The short window between enrollment start and the MSI installing (the first sync) is not yet captured live; coverage of that early window is being improved.
* Apps that are *not* part of the Device Preparation policy install after the user reaches the desktop, like on any Intune-managed device. The agent keeps tracking those installs for as long as it runs, but a machine can genuinely still be "finishing up" after the session completes.
