---
type: Feature Guide
tags: [announcements, service-health]
description: >-
  Dated announcements — breaking changes caused by external updates, incidents,
  and their resolutions.
---

# Service Announcements

Documented issues caused by external changes (e.g. Microsoft updates) and platform incidents — each entry describes the impact, what still works, and whether action is needed. Newest first.

## 2026-07-14 — **Scheduled:** Infrastructure Maintenance — Platform Unavailable This Weekend

Autopilot Monitor undergoes planned infrastructure maintenance over the coming weekend, from **Sat 18 Jul, 00:00** until **Mon 20 Jul, 00:00 CEST (UTC+2)** (expected). During this window the platform is **not available**: the portal cannot be reached and the ingestion API is offline, so agents cannot send data.

There is nothing to do on your side. Agents continue collecting locally during the maintenance and re-sync automatically once the platform is back online — no enrollment data is lost. If the work finishes ahead of schedule, the platform will come back sooner; the completion will be announced here and in the portal.

## 2026-05-19 — **Resolved:** Safe Deletion in Place — Cleanup Incident Follow-up

Following the cleanup incident from 2026-04-16, the announced safeguards are now in place. Both deletion paths — admin-UI session deletion and behind-the-scenes maintenance cleanup — now always create a backup first and offer a way back:

* **Admin-UI deletions** collect a full manifest of everything belonging to the session before removing anything, run in resumable steps, and can be **reversed afterwards** (completely or partially), with counts corrected automatically.
* **Maintenance cleanups** run through a guided procedure with several stop points: a verified sample first, a full on-disk backup before any delete, an automatic stop if the volume is much larger than expected, and two verified test rounds before bulk deletion. The backup allows one-by-one or full restore.

The events lost on 2026-04-16 cannot be recovered. What these procedures guarantee is that any future deletion has a backup and a way back. No action needed.

## 2026-04-16 — **Breaking:** Event Data Loss Due to Faulty Cleanup Operation

A faulty cleanup operation deleted a significant number of events. Affected sessions may show missing events, incomplete phases, or incorrect completion states; the deleted data cannot be restored. Operational safeguards to block and verify cleanup operations before execution were announced (and delivered — see the 2026-05-19 entry).

## 2026-04-16 — **Info:** Transparency Note — Root Cause of the Event Data Loss

In full transparency: during a storage migration to a new, more performant layout, an attempt to remove orphaned event entries from the platform's early days went wrong — part of the filter logic was not applied as expected, and a larger number of historical event entries were deleted before the issue was caught.

The impact is mainly limited to older session timelines: events are most valuable during and shortly after the enrollment, and the default retention already removes sessions after 90 days. Even so, the impact is real. At the time there was no separate staging environment, so larger refactorings had to run on the live system; additional operational safeguards and procedures were put in place as a consequence (see 2026-05-19). Thank you for the understanding, patience, and trust while building the platform together with early adopters.

## 2026-04-16 — **Known Issue:** Agent Changes in Progress — Possible Detection Issues

The agent underwent active changes; during that period detection and classification issues could occur (sessions not completing correctly, events misclassified, sessions falsely classified as White Glove). Resolved by subsequent agent updates.

## 2026-04-06 — **Resolved:** Delivery Optimization Data Restored via OS-Level Collection

Autopilot Monitor now collects Delivery Optimization data directly from the OS (`Get-DeliveryOptimizationStatus`), bypassing the IME log entirely. This restores DO metrics — bytes from peers, peer-caching %, download progress — for all devices, **including those running IME ≥ 1.101**. The OS-level collector works alongside IME log parsing; when both provide data for the same app, the IME log path wins. No action needed.

## 2026-04-05 — **Breaking:** IME 1.101.x Removes Delivery Optimization Telemetry from Logs

Starting with IME **1.101.x**, Microsoft no longer writes DO telemetry to the IME log (the `[DO TEL] = {…}` JSON is gone; only a bare success line remains). The detailed stats — BytesFromPeers, PeerCaching %, LAN/Group peers — simply aren't written anymore, so they cannot be parsed from logs on those devices. This is a Microsoft-side change, not an Autopilot Monitor bug.

**Impact:** sessions from IME ≥ 1.101 devices recorded before 2026-04-06 lack DO metrics. Superseded by the OS-level collection above.
