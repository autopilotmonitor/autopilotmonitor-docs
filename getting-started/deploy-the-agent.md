---
type: How-to Guide
tags: [agent, deployment, intune]
description: >-
  Deploy the monitoring agent as an Intune platform script — safe to assign
  broadly thanks to built-in pre-requisite guards.
---

# Deploy the Agent

The agent is deployed with a small PowerShell loader script distributed as an Intune **platform script**. It downloads the current bootstrapper, verifies our signature on it, and lets it install and register the agent — no manual steps on the device, and nothing to re-upload later.

## Safe to assign broadly

Before installing anything, the bootstrapper runs a series of pre-requisite guards. The agent is only installed when **all** of them pass — devices that don't meet the criteria are skipped silently. That makes it safe to assign the script to *All devices*: already-enrolled machines in daily use are never touched.

| Guard | What it checks |
| --- | --- |
| **No previous deployment** | A registry marker `HKLM:\SOFTWARE\AutopilotMonitor\Deployed` is the only artifact that survives agent self-destruct. It permanently prevents the bootstrapper and agent from ever running again on the same device. |
| **No real user profiles** | Combines WMI (`Win32_UserProfile`) and filesystem checks under `C:\Users` (system profiles like `defaultuser*`, `Public`, `Default` are excluded). Real user profiles mean the device is already in productive use. |
| **No previous user logon** | Checks `LastLoggedOnUser` in the LogonUI registry — during the device phase of ESP, no real user has signed in yet. |
| **Within bootstrap window** | Device uptime must be under 12 hours. Prevents installation on devices that have been powered on for a long time without enrolling. Sleep/standby does not reset this timer. |
| **Agent not already installed** | If the agent binary is already present, installation is skipped. |

One deliberate exemption, with two triggers: a single freshly created user profile does not block the install when the device is verifiably mid-enrollment rather than in productive use. That is the case during a **Windows Backup for Organizations** restore (the device positively reports OOBE as in progress) and on a **Windows 365 Cloud PC** at the assigned user's first connect (the device positively identifies as a Cloud PC via local Windows 365 markers; the uptime guard is also waived there, since Cloud PCs run headless for days before the first sign-in — see [Windows 365 Cloud PCs](windows-365-cloud-pcs.md)). Any other combination keeps the original skip behavior.

### Try it first: dry run

Want to verify which devices would receive the agent? Run this read-only check in PowerShell on any machine — it evaluates all guards and transparently reports the install decision without changing anything:

```powershell
irm 'https://download.autopilotmonitor.com/agent/Test-ShouldBootstrapAgent.ps1' | iex
```

The script source is public: [Test-ShouldBootstrapAgent.ps1 on GitHub](https://github.com/okieselbach/Autopilot-Monitor/blob/main/scripts/Bootstrap/Test-ShouldBootstrapAgent.ps1).

## Everything you deploy is signed

Scripts and agent binaries carry an Authenticode signature from **glueckkanja AG**, timestamped so it stays valid beyond the certificate's lifetime.

* The loader refuses to run a bootstrapper that is not signed by us — a tampered or substituted download never executes on your devices.
* You can check any file before you upload it: `Get-AuthenticodeSignature .\Start-AutopilotMonitor.ps1` must report `Valid` and `glueckkanja AG`.
* The agent's executables and libraries are signed too, so application control policies (WDAC, AppLocker) can allow them by publisher.
* If your policy requires signed scripts, **Enforce script signature check** can be turned on in the Intune script settings.

## Deployment steps

### 1. Download the loader script

Download [`Start-AutopilotMonitor.ps1`](https://download.autopilotmonitor.com/agent/Start-AutopilotMonitor.ps1) from the Autopilot Monitor download host — current version: ![Latest loader version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fdownload.autopilotmonitor.com%2Fagent%2Fversion.json&query=%24.loaderVersion&label=Loader&prefix=v&color=2563eb).

Or fetch it from a PowerShell prompt:

```powershell
irm 'https://download.autopilotmonitor.com/agent/Start-AutopilotMonitor.ps1' -OutFile .\Start-AutopilotMonitor.ps1
```

This is the only file you upload to Intune. On each device it fetches the current bootstrapper, checks that we signed it, and runs it. The loader carries none of the logic that changes over time, so the copy in Intune stays valid — an older loader keeps working.

The agent (current version ![Latest agent version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fdownload.autopilotmonitor.com%2Fagent%2Fversion.json&query=%24.version&label=Agent&prefix=v&color=2563eb) — see the [Agent Changelog](../changelog/agent-changelog.md)) is downloaded and hash-verified along the way; you never handle the agent binary yourself.

Sources are public on [GitHub](https://github.com/okieselbach/AutopilotMonitor/tree/main/scripts/Bootstrap). The signed copies are served from the download host — deploy those.

{% hint style="info" %}
**Prefer a single script?** Assigning [`Install-AutopilotMonitor.ps1`](https://download.autopilotmonitor.com/agent/Install-AutopilotMonitor.ps1) directly still works and installs exactly the same agent — current version: ![Latest bootstrapper version](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fdownload.autopilotmonitor.com%2Fagent%2Fversion.json&query=%24.bootstrapVersion&label=Bootstrapper&prefix=v&color=2563eb). The difference is upkeep: that file changes when the bootstrapper changes, so you re-upload it in Intune to stay current. Updates worth acting on are flagged in the [Platform Changelog](../changelog/platform-changelog.md); [the FAQ](../troubleshooting/faq.md) explains why an outdated copy can silently skip enrollments.
{% endhint %}

### 2. Create a platform script in Intune

In the **Microsoft Intune admin center**, go to **Devices → Scripts and remediations → Platform scripts** and click **+ Add → Windows 10 and later**.

| Setting | Value |
| --- | --- |
| Name | `Start Autopilot Monitor` |
| Script | Upload the downloaded `.ps1` file |
| Run this script using the logged on credentials | **No** (runs as SYSTEM) |
| Enforce script signature check | **No** — or **Yes**, the file is [signed](#everything-you-deploy-is-signed) |
| Run script in 64 bit PowerShell Host | **Yes** |

### 3. Assign to a device group

Assign the script to the device group that covers your Autopilot devices. The two common choices:

* **Recommended:** a **dynamic Entra ID device group** targeting only Autopilot-registered hardware, using the membership rule:

```
(device.devicePhysicalIds -any _ -startsWith "[ZTDId]")
```

* **All devices** — the built-in Intune group. The guards make this safe for already-provisioned machines, but without a filter the script also reaches every other managed Windows device (Teams Rooms devices, kiosks, …). Prefer the dynamic Autopilot group unless you have a reason not to.

{% hint style="info" %}
**Monitoring Windows 365?** Cloud PCs are not Autopilot-registered, so the dynamic Autopilot group does not include them. Assign the script to an additional Cloud PC group — membership rule and setup on [Windows 365 Cloud PCs](windows-365-cloud-pcs.md).
{% endhint %}

{% hint style="info" %}
**Using Autopilot Device Preparation?** There the platform script runs only *after* the app phase — too late for full monitoring. Deploy the agent via the MSI line-of-business app instead; setup on [Autopilot Device Preparation](autopilot-device-preparation.md).
{% endhint %}

### 4. Done

Once the script runs on an enrolling device, the agent installs itself, creates a scheduled task under SYSTEM, and begins monitoring immediately. The session appears on your dashboard within seconds of the agent starting.

Next: walk through [Your First Monitored Enrollment](your-first-monitored-enrollment.md).
