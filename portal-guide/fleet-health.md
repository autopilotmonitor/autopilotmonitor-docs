---
description: >-
  The fleet-wide health overview — success rates, failure patterns, and the
  slowest models and apps across all your enrollments.
---

# Fleet Health

Fleet Health zooms out from individual sessions to the whole fleet: is enrollment healthy *in general*, and where do the systematic problems hide? A **time-range selector** (7 / 30 / 90 days) applies to everything on the page, and all numbers refresh live as new sessions arrive.

## What you see

* **Headline stats** — **Success Rate** (color-coded: green ≥ 95 %, yellow ≥ 80 %, red below), **Avg. Enrollment Time**, **Failed**, **Incomplete**, and **Active Now**. The **Success Rate is honest**: its denominator is only sessions that reached a real verdict (Succeeded + Failed). **Incomplete** sessions — silent enrollments that never produced a completion or an explicit failure signal — are counted separately and **excluded from the failure rate**, so a fleet of laptops closed mid-ESP no longer looks like a fleet of broken enrollments. See [Sessions & Statuses](../concepts/sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions) for how a timeout is classified.
* **Enrollments Timeline** — a stacked bar chart per day (successes over failures) for spotting trends and bad days at a glance.
* **Ranked problem lists**:
  * **Top Failure Reasons** — what actually kills your enrollments, with proportional bars.
  * **Slowest Models** and **Top Failing Models** — hardware lines that drag down or break enrollments.
  * **Slowest Apps** (average and max install duration) and **Top Failing Apps** — including the top failure exit codes per app, e.g. `1603 (12x)`.
* **Health by Device Model** — every model with its success rate, device count, and a color-coded bar.

## How to use it

Fleet Health is the page for the weekly look: a dropping success rate, one model climbing the failing list, or one app dominating the slow list is your cue to drill down — click through to the [Software](software-inventory-and-vulnerabilities.md) app detail for failing apps, or filter the [Dashboard](dashboard-and-sessions.md) by model or status for failing hardware. For contractual reporting against defined targets, use [SLA Compliance](sla-compliance.md) instead.
