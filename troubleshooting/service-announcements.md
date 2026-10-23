---
type: Feature Guide
tags: [announcements, service-health]
description: >-
  Dated service announcements, newest first — breaking changes from external
  updates, platform incidents and maintenance, their impact, and any action
  needed.
---

# Service Announcements

Documented issues caused by external changes (e.g. Microsoft updates) and platform incidents — each entry describes the impact, what still works, and whether action is needed. Newest first.

## 2026-10-22 — **Change:** The Bootstrap Leaves Nothing Behind

A device the bootstrapper skips now keeps nothing of Autopilot Monitor, and the loader downloads and checks the bootstrapper in a folder only SYSTEM and administrators can change. The loader also runs only a current bootstrapper, and the deployment-marker and last-logon guards now work in Intune's default 32-bit PowerShell host as well.

If you assign the loader (`Start-AutopilotMonitor.ps1`), upload the new version once (recommended); older loaders keep working. For Device Preparation, replace bootstrap MSI 1.0, which runs the bootstrapper without checking our signature (action required); replace 1.1 like the loader (recommended). A copy of the bootstrapper (`Install-AutopilotMonitor.ps1`) assigned directly gets the cleanup with version 2.7. See [Deploy the Agent](../getting-started/deploy-the-agent.md) and [Autopilot Device Preparation](../getting-started/autopilot-device-preparation.md).

**Update, later the same day.** Installer 2.8 fixes the bootstrap MSI channel: since installer 2.7 the agent did not start on devices bootstrapped by the MSI. Devices that ran the MSI today got no agent and keep an unused agent folder: delete `C:\ProgramData\AutopilotMonitor` as administrator to remove it; a reset device enrolls with the agent again. Devices with the platform script were not affected. After a security review, loader 1.4 and bootstrap MSI 1.3 replace today's loader 1.3 and MSI 1.2; upload those instead.

**Update 2026-10-23.** Bootstrap MSI 1.3 left its loader file under `C:\Program Files\AutopilotMonitor` on every device it ran on. MSI 1.4 keeps the copy under ProgramData and the bootstrapper removes it; upload 1.4 in place of 1.3. A device that receives the update loses the file; on a device that keeps 1.3, delete the folder as administrator.

## 2026-10-20 — **Change:** Push Notifications to Paired Devices

Tenant Admins can now add a Push channel under **Settings → Notifications**. Admins and Operators then pair a phone or browser from their signed-in portal session and receive the channel's alerts there — and a ping when an enrollment they watch with **Notify me when done** finishes. Each alert is encrypted end to end for the receiving device; the push service of its platform (Apple, Google, Microsoft or Mozilla) only passes on ciphertext and delivery metadata. The push page on the phone needs no sign-in; pushes carry only lock-screen details, and everything else stays in the portal behind your sign-in. Push stays off until a Tenant Admin adds the channel.

How to set it up and what a push contains: [Push Notifications](../integrations/push-notifications.md). Where the data goes: [Data Flows & External Services](../trust/data-flows.md).

## 2026-10-13 — **Change:** Every File of the Agent Package Is Verified Before It Runs

Each agent package now carries a catalog signed by glueckkanja AG that lists every file in it. The bootstrapper, the bootstrap-token script and the agent's self-updater check this catalog on the device and install a package only when the catalog carries our signature and every file is present and unchanged, with nothing added. Until now the bootstrapper verified the signatures of our own binaries only. Third-party libraries enter the package only with their vendor's own signature, so every binary in it is signed. The package is also smaller: a library Windows already provides is no longer included.

If you assign the loader (`Start-AutopilotMonitor.ps1`), there is nothing to do — every device fetches the current bootstrapper. A copy of the bootstrapper (`Install-AutopilotMonitor.ps1`) assigned directly keeps working, but it does not perform the new check; replace it with the loader as described in [Deploy the Agent](../getting-started/deploy-the-agent.md).

## 2026-07-18 — **Resolved:** Infrastructure Maintenance Completed Successfully

The scheduled infrastructure maintenance was completed successfully on **Saturday, 18 Jul** — well ahead of the announced window. The platform is fully operational again: portal, ingestion API, realtime updates, and the MCP server are all back to normal, and agents have resumed syncing. Enrollment data collected on devices during the window was preserved and uploaded as designed — no data was lost.

There is nothing to do on your side, and everything should behave exactly as before. If you notice any irregularities, please open a [GitHub issue](https://github.com/okieselbach/Autopilot-Monitor/issues).

## 2026-07-14 — **Scheduled:** Infrastructure Maintenance — Platform Unavailable This Weekend

Autopilot Monitor undergoes planned infrastructure maintenance over the coming weekend, from **Sat 18 Jul, 00:00** until **Mon 20 Jul, 00:00 CEST (UTC+2)** (expected). During this window the platform is **not available**: the portal cannot be reached and the ingestion API is offline, so agents cannot send data.

There is nothing to do on your side. Agents continue collecting locally during the maintenance and re-sync automatically once the platform is back online — no enrollment data is lost. If the work finishes ahead of schedule, the platform will come back sooner; the completion will be announced here and in the portal.

## 2026-05-19 — **Resolved:** Safe Deletion Procedures in Place

During the early preview phase, a maintenance cleanup went wrong and removed a number of historical events from older session timelines. As a consequence, both deletion paths — admin-UI session deletion and behind-the-scenes maintenance cleanup — were rebuilt around one guarantee: every deletion creates a backup first and offers a way back.

* **Admin-UI deletions** collect a full manifest of everything belonging to the session before removing anything, run in resumable steps, and can be **reversed afterwards** (completely or partially), with counts corrected automatically.
* **Maintenance cleanups** run through a guided procedure with several stop points: a verified sample first, a full backup before any delete, an automatic stop if the volume is much larger than expected, and verified test rounds before bulk deletion. The backup allows one-by-one or full restore.

No action needed.

## 2026-04-16 — **Known Issue:** Agent Changes in Progress — Possible Detection Issues

The agent underwent active changes; during that period detection and classification issues could occur (sessions not completing correctly, events misclassified, sessions falsely classified as White Glove). Resolved by subsequent agent updates.

## 2026-04-06 — **Resolved:** Delivery Optimization Data Restored via OS-Level Collection

Autopilot Monitor now collects Delivery Optimization data directly from the OS (`Get-DeliveryOptimizationStatus`), bypassing the IME log entirely. This restores DO metrics — bytes from peers, peer-caching %, download progress — for all devices, **including those running IME ≥ 1.101**. The OS-level collector works alongside IME log parsing; when both provide data for the same app, the IME log path wins. No action needed.

## 2026-04-05 — **Breaking:** IME 1.101.x Removes Delivery Optimization Telemetry from Logs

Starting with IME **1.101.x**, Microsoft no longer writes DO telemetry to the IME log (the `[DO TEL] = {…}` JSON is gone; only a bare success line remains). The detailed stats — BytesFromPeers, PeerCaching %, LAN/Group peers — simply aren't written anymore, so they cannot be parsed from logs on those devices. This is a Microsoft-side change, not an Autopilot Monitor bug.

**Impact:** sessions from IME ≥ 1.101 devices recorded before 2026-04-06 lack DO metrics. Superseded by the OS-level collection above.
