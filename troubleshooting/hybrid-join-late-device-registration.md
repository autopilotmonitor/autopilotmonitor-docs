---
type: Troubleshooting
tags: [troubleshooting, hybrid-join, enrollment, entra-connect]
description: Hybrid Join devices whose sessions start late or never — the agent is waiting for a device registration that your directory hasn't delivered yet.
---

# Hybrid Join: sessions start late — or never

You deploy Autopilot with **Hybrid Entra Join** (HAADJ), the device enrolls, but in the portal the session either never appears, appears long after the enrollment started, or shows a striking pattern: a couple of events right at the start, then a silent gap of 30, 60, 90 minutes, then normal activity.

That gap is not a monitoring glitch. **It is your enrollment, measured.** The agent is waiting for the same thing your user is waiting for in front of the Enrollment Status Page: the device's Entra registration. Every minute of that gap is a minute added to every single Hybrid Join enrollment in your environment — fixing it makes deployments faster for everyone, not just the dashboard prettier.

## How to recognize it

On the device, `%ProgramData%\AutopilotMonitor\Logs` shows the agent starting, failing to find a tenant identity, waiting, and giving up cleanly:

```
[WARN] TenantIdResolver: no TenantId resolvable from registry — device likely not yet AAD-joined / MDM-enrolled ...
[WARN]   - Type=6 ...: EnrollmentState=1, AADTenantID=<missing>, UPN=<redacted>
[WARN] TenantIdResolver: HKLM\SYSTEM\CurrentControlSet\Control\CloudDomainJoin\TenantInfo not found (device not AAD-joined yet).
[INFO] TenantIdAwaiter: TenantId not yet resolvable — waiting up to 600s for registry signal.
[WARN] TenantIdAwaiter: timeout after 600s — TenantId still not available, agent will exit cleanly and retry on next trigger.
[ERROR] V2 agent cannot start: TenantId not available (registry empty + no bootstrap config).
```

In the portal: a Hybrid session whose **duration is much larger than its visible activity** — the timeline holds a few bootstrap events, then nothing, then the regular event stream. The silent stretch is exactly the window in which the device had no Entra identity yet.

## What is actually happening

On a Hybrid Join deployment the identity chain is long: the device is joined to **on-premises AD** first (offline domain join), then it must be **discovered by Entra Connect sync**, complete its **Entra device registration**, and finish **Intune enrollment** — only then does a tenant identity exist in the registry for the agent to read.

The agent handles this: it waits up to 10 minutes for the registration to land, exits cleanly if it doesn't, and retries on every boot for up to 48 hours. The reboot built into the Hybrid Join flow usually gives it that second chance, and once it starts it **replays the IME log from the beginning**, so app installs from before its start are reconstructed. In a healthy environment the registration lands within one sync cycle and the session starts minutes after the device does.

When the registration takes an hour or more, the agent isn't the only thing stalled — ESP progress, policy delivery, and your user are stalled with it.

## What to check — in this order

1. **On an affected device: `dsregcmd /status`.**
   `AzureAdJoined : NO` long after the AD join completed is the finding. Error **`0x801c03f3`** ("device object not found") means Entra has never heard of this device — the sync never delivered it.

2. **Event Viewer → Applications and Services Logs → Microsoft → Windows → User Device Registration → Admin.**
   Event **306** carries the registration failure detail; **304**/**307** show the retry loop. The timestamps tell you precisely how long the device fought for its registration.

3. **Entra Connect: is the computer object in sync scope?**
   The OU your Autopilot Domain Join profile targets must be inside the Entra Connect sync scope. A device written to an un-synced OU never reaches Entra — this is the single most common cause.

4. **Entra Connect: is `userCertificate` populated and synced?**
   The device writes its registration certificate to the **`userCertificate`** attribute of its AD computer object; Entra Connect syncs it to create the Entra device object. Check the attribute on a stalled device's computer object, and confirm the attribute isn't filtered out of your sync rules.

5. **Sync latency.**
   The default Entra Connect cycle is **30 minutes**, and AD replication between the DC the Offline Domain Join connector writes to and the DC Entra Connect reads from adds on top. As a test, join a device and run `Start-ADSyncSyncCycle -PolicyType Delta` on the Entra Connect server — if the registration completes right after, latency is your whole problem.

Measure one number: **time from AD join to the device appearing in the Entra portal.** Minutes = healthy. An hour = every enrollment in your fleet is paying that hour.

Autopilot Monitor helps you here, too: the built-in device gather runs `dsregcmd /status` during Account Setup, and the analyze rules flag the known Hybrid Join failure codes (`0x801C03F3`, `0x80070774`, the `0x80004005` timeout) directly on the session.

## If the delay can't be fixed right away

The directory fix above is the real solution — these knobs only widen the agent's patience while you work on it:

* **Raise the wait.** The agent accepts `--tenant-id-wait <seconds>` at install time (default 600). Intune platform scripts take no parameters, so set it on the `--install` line inside your copy of the bootstrap script, e.g. `--install --tenant-id-wait 1800`.
* **Rely on the boot retry.** Even after a timeout the agent retries at every boot for 48 hours and backfills from the IME log — sessions from late-registering devices arrive late, but they arrive.
* **Bootstrap tokens** (Pro) authenticate the agent before any MDM certificate exists and remove the dependency entirely — see [Bootstrap Script & Tokens](../reference/bootstrap-script-and-tokens.md).

## Still stuck?

Collect a [diagnostics package](diagnostics-and-log-collection.md) and use **Report Session** on an affected session — the registration timestamps above are exactly the context that makes it quick to diagnose.
