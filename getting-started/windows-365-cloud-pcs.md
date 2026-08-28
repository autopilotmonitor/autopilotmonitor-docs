---
type: Feature Guide
tags: [windows365, cloudpc, deployment, device-validation]
description: >-
  Monitor Windows 365 Cloud PC enrollments — why Cloud PCs are different, how
  to enable Cloud PC validation, and how to include Cloud PCs in the agent
  deployment.
---

# Windows 365 Cloud PCs

Autopilot Monitor can monitor the enrollment of **Windows 365 Cloud PCs**. Support is opt-in per tenant — nothing changes for your environment until you enable it.

## Why Cloud PCs are different

Compared to a physical Autopilot device, a Cloud PC differs in two structural ways:

1. **Provisioned headless.** The Windows 365 service runs OOBE, Intune enrollment, and the device phase before any user ever connects. The part worth monitoring — Account Setup with app installs, policies, and Windows Hello — only starts when the assigned user connects for the first time.
2. **Never Autopilot-registered.** Cloud PCs have no Windows Autopilot device identity, and their serial numbers cannot be pre-imported as Corporate Identifiers (the VMs are minted at provisioning time). Both standard [device validation methods](../reference/settings.md#enrollment-device-validation) therefore reject them.

Autopilot Monitor addresses both: a dedicated validation method for Cloud PCs, and a bootstrapper that recognizes Cloud PCs and installs the agent exactly in the first-connect window.

## Enable Cloud PC validation

Two steps, both one-time:

1. **Grant the optional Graph permission.** Cloud PC validation needs the read-only permission `CloudPC.Read.All`, which is not part of the default consent. Grant it as the feature `W365CloudPcValidation` using the add-on grant script — see [Optional Graph Permissions](../reference/optional-graph-permissions.md).
2. **Turn on the validation.** In **Settings → Enrollment Device Validation**, enable **Windows 365 Cloud PC Validation**.

The new method acts as a fallback after the Autopilot and Corporate Identifier lookups: the backend takes the Intune device id from the device's Intune-issued MDM certificate and requires a matching Cloud PC object in your tenant's Cloud PC inventory (Graph `virtualEndpoint/cloudPCs`). Only machines actually provisioned by the Windows 365 service have such an object, so enabling this does not open registration to any other device type.

## Include Cloud PCs in the agent deployment

The dynamic Autopilot device group recommended in [Deploy the Agent](deploy-the-agent.md#3-assign-to-a-device-group) does **not** contain Cloud PCs — they are not Autopilot-registered. Assign the same platform script additionally to a dynamic Entra ID device group with this membership rule:

```
(device.deviceModel -startsWith "Cloud PC") and (device.deviceModel -ne "Cloud PC for Agents")
```

(Assigning to *All devices* works too — the bootstrapper guards keep productive machines untouched.)

On the device, the bootstrapper positively identifies a Cloud PC via two local Windows 365 markers and installs the agent only in the first-connect window: exactly one user profile exists and it was created within the last 15 minutes. A Cloud PC waiting headless for its user, or a productive Cloud PC with an older profile, is skipped. The usual 12-hour uptime guard does not apply to Cloud PCs — they routinely run headless for days between provisioning and the user's first sign-in.

## What you see in the portal

* Sessions from Cloud PCs carry a **Cloud PC** flag, and session search can filter on it.
* The **"Not registered" devices** view badges Cloud PCs. If your Cloud PCs appear there, the validation method is not enabled (or the Graph permission is missing) — the badge hint points to both.
* The [Progress Portal](../portal-guide/progress-portal.md) follows a Cloud PC by the device name shown in the Windows App and lists only the first-connect steps its user can watch (sign-in, account setup, app installation, finalizing).

## Limitations

* The headless provisioning phase itself is not monitored — there is no user session, and the agent installs at first connect. Monitoring covers the first-connect enrollment (Account Setup).
* On Windows 365 Frontline in shared mode, a session start can look like a first connect, so a one-time agent install may occur. The agent then finds the enrollment already complete and removes itself; the deployment marker prevents any repeat.
