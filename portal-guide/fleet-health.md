---
type: Feature Guide
tags: [portal, fleet-health, metrics]
description: >-
  The fleet-wide health overview — success rates, failure patterns, where
  enrollment time goes, and how often a device needed more than one attempt.
---

# Fleet Health

Fleet Health zooms out from individual sessions to the whole fleet: is enrollment healthy *in general*, where does the time go, and where do the systematic problems hide? A **time-range selector** (7 / 30 / 90 days) drives the page — the one exception is [Time attribution](#time-attribution), which is fixed to the last 30 days — and all numbers refresh live as new sessions arrive.

## What you see

* **Headline stats** — **Success Rate** (color-coded: green ≥ 95 %, yellow ≥ 80 %, red below), **Avg. Enrollment Time**, **Failed**, **Incomplete**, and **Active Now**. The **Success Rate is honest**: its denominator is only sessions that reached a real verdict (Succeeded + Failed). **Incomplete** sessions — silent enrollments that never produced a completion or an explicit failure signal — are counted separately and **excluded from the failure rate**, so a fleet of laptops closed mid-ESP no longer looks like a fleet of broken enrollments. See [Sessions & Statuses](../concepts/sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions) for how a timeout is classified.
* **Enrollments Timeline** — a stacked bar chart per day (successes over failures) for spotting trends and bad days at a glance.
* **Ranked problem lists**:
  * **Top Failure Reasons** — what actually kills your enrollments, with proportional bars.
  * **Slowest Models** and **Top Failing Models** — hardware lines that drag down or break enrollments.
  * **Slowest Apps** (average and max install duration) and **Top Failing Apps** — including the top failure exit codes per app, e.g. `1603 (12x)`.
* **Health by Device Model** — every model with its success rate, device count, and a color-coded bar.

App durations are computed over **measured installs** only. An app that Intune reported as *skipped* (e.g. a WinGet update policy that didn't apply to the device) is not an install attempt: it is excluded from the average, the maximum, and the slowest ranking, and the skipped count is shown next to the app instead. This is why the *Slowest Apps* list carries a *measured installs* caption.

## Time attribution

*Where does enrollment time actually go?* This section splits the median enrollment into the six segments of the enrollment stack — **Device preparation**, **Apps (ESP)**, **Identity & Hello**, **User ESP**, **Desktop handoff**, and the honest **Unattributed** remainder — as one stacked bar per **enrollment class**. Classes (user-driven, pre-provisioning/White Glove, self-deploying, Device Preparation v2) are never mixed: their timelines are structurally different, so an average across them would be meaningless.

Unlike the rest of the page, this section always covers the **last 30 days**, independent of the time-range selector — a median needs a stable window, and a median of daily medians is not the median of the range.

* A class is only shown with numbers once it has **at least 20 clean sessions** in the window; below that it reads *"insufficient data (n=…)"* rather than a number that would move with every new device.
* Sessions whose data can't carry the split — a **flagged** session (e.g. the agent started late and missed early phases) or one **without a breakdown** — are excluded and counted next to the class, so you can see how much of your fleet the bar actually represents.
* Hovering a segment shows its median and p90.

**Top time-consuming blocking apps** ranks the apps that sit on the ESP's critical path — the ones the user is actually waiting for. Membership in the blocking set is read from the device's own ESP tracking data, not guessed from install order. *Removing it saves* is a **what-if upper bound**: the time the enrollment would have ended earlier if that app had not blocked, including the idle wait before it started. It is always worded *"up to"* with a p75 next to it — real savings are usually lower, because another app often takes over the critical path.

## First-time-right

The success rate answers *"did this enrollment work?"* — first-time-right answers *"did this **device** work, or did somebody wipe and retry it until it did?"* A device that fails twice and succeeds on the third run ends up enrolled and contributes one success to the rate, but it cost three enrollments and a technician's afternoon. That cost is invisible everywhere else in the portal.

* The **rate** is the share of completed [device journeys](../concepts/sessions-and-statuses.md#device-journeys--attempts) that succeeded on the first attempt, over the selected time range. It appears once there are **at least 20 completed journeys**, otherwise *"insufficient data (n=…)"*. Sessions from devices with a placeholder serial number are excluded and disclosed below the rate.
* The **weekly trend** shows the rate per calendar week (bar height = rate), so an improvement after a fixed image or a corrected app assignment is visible.
* **Attempts until success** is a histogram of how many terminal attempts each completed journey needed — one green bar for *1 attempt*, amber bars for everything worse.
* **Repeat devices** names up to ten devices whose current journey took two or more attempts — most attempts first, then most recent — with the failure reason from the most recent failed attempt and a link into that session. In the cross-tenant (aggregated) view this list is intentionally empty: select a tenant to see device-level detail.

## How to use it

Fleet Health is the page for the weekly look: a dropping success rate, one model climbing the failing list, or one app dominating the slow list is your cue to drill down — click through to the [Software](software-inventory-and-vulnerabilities.md) app detail for failing apps, or filter the [Dashboard](dashboard-and-sessions.md) by model or status for failing hardware. For contractual reporting against defined targets, use [SLA Compliance](sla-compliance.md) instead.

Two questions the page answers that the success rate alone cannot:

* *"Enrollment takes too long"* → **Time attribution**. If the *Apps (ESP)* segment dominates, the blocking-apps table names the candidates to take off the critical path (make an app non-blocking in the ESP profile rather than removing it, and it stops costing enrollment time while still being installed). A large *Unattributed* share means the timeline is thin for that class — usually an agent that started late.
* *"How much rework is my fleet actually causing?"* → **First-time-right**. Compare it with the success rate: a healthy success rate next to a low first-time-right rate means devices are being retried until they pass, and the *Repeat devices* list tells you which ones and why.
