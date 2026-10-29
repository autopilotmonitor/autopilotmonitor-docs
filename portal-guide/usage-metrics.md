---
type: Feature Guide
tags: [portal, usage, metrics]
description: >-
  Tenant usage at a glance — session volumes and success rate, deployment
  types, apps and scripts per session, portal users, duration percentiles, and
  hardware.
---

# Usage Metrics

Usage Metrics is the informational overview of how your tenant uses Autopilot Monitor — useful for capacity questions, adoption reporting, and a quick feel for the numbers behind the fleet.

## What you see

* **Sessions** — total / today / last 7 days / last 30 days, and succeeded / failed / in progress / success rate.
* **Deployment Types** — your enrollments split three ways, each with counts, shares and bars: Autopilot or Device Preparation, Microsoft Entra joined or hybrid joined, and the Autopilot deployment mode (user-driven, self-deploying or pre-provisioned). The modes add up to the Autopilot count.
* **Apps & Scripts** — average apps per session (plus total unique apps), average platform-script and remediation-script executions per session.
* **Users** — unique portal users (90 days), daily logins, and 7-/30-day active users.
* **Performance** — average, median, **P95**, and **P99** enrollment durations. The percentiles are the honest numbers for planning: the median tells you the typical experience, P95/P99 tell you what your unluckiest users live through. See [Averages, Medians & Percentiles](../concepts/averages-medians-and-percentiles.md) for how to read them.
* **Hardware** — top manufacturers and top models, ranked by enrollment sessions. Each row shows its sessions, their share of all sessions, and the distinct devices behind them. A device that was reset and enrolled again adds a session each time but stays one device. Devices are told apart by serial number; a session without a usable serial number counts as its own device.

Apart from the total enrollments since signup, the counts cover the last 90 days. For a different window, add `?days=` to the page address (up to 365 days on Pro).

The page is read-only; metrics are cached with a computed-at note and a **Refresh** button.
