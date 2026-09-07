---
type: Concept
tags: [metrics, statistics, percentiles]
description: >-
  What average, median, P90, P95 and P99 mean, how to read them, and which
  portal page uses which one over which sessions.
---

# Averages, Medians & Percentiles

Enrollment durations are shown in several forms across the portal: an **average**, a **median**, and percentiles such as **P90**, **P95** or **P99**. They answer different questions, and a fleet of enrollments almost never has a "normal" distribution — most devices finish in a similar time, a few take much longer. This page explains the five terms once, with one example, so every duration figure in the portal reads the same way.

## One example, five numbers

Ten devices enrolled at one site. Sorted from fastest to slowest, their enrollment durations in minutes were:

```
22  24  25  26  27  28  30  33  41  94
```

Nine of them finished within 41 minutes. One device took 94 minutes — a content download that stalled for an hour before recovering.

| Statistic | Value | How it is read |
|---|---|---|
| **Average** (mean) | **35 min** | Add all durations, divide by their count (350 / 10). The single 94-minute device lifts the average above what nine of the ten users actually experienced. |
| **Median** (P50) | **28 min** | The value in the middle of the sorted list: half of the enrollments were faster, half were slower. The 94-minute outlier does not move it. |
| **P90** | **41 min** | 90 out of 100 enrollments finished within this time. |
| **P95** | **94 min** | 95 out of 100 enrollments finished within this time. With only ten samples, this is the slowest one. |
| **P99** | **94 min** | 99 out of 100 enrollments finished within this time. |

A percentile is always read the same way: **P*n* is the time within which *n* percent of the enrollments finished.** The median is P50. The higher the percentile, the more it is about the slowest devices — the ones whose users call the service desk.

## Which number answers which question

* **"How long does enrollment take for a typical device?"** — the **median**. It ignores the few stragglers and is the honest headline figure. The Dashboard and Fleet Health lead with it for exactly that reason.
* **"What does the slow tail look like?"** — **P90**, **P95** or **P99**. A median of 28 minutes with a P95 of 94 minutes means most users are fine but every twentieth device has a real problem; a median of 28 with a P95 of 40 means the fleet is uniformly healthy. Service-level targets are set on P95 because it is what your unluckiest users still live through, without letting a single broken device define the number.
* **"What is the total effort, or the per-app cost?"** — the **average**. When time is divided further — minutes per app, bytes per second — the average is the number that can be divided; a median cannot. The Geographic Performance page uses averages for that reason, and states the basis next to them.

A rule of thumb: when average and median are far apart, the distribution has a long tail. That gap is a diagnostic signal in itself — the tail is where the problems are, and the percentiles tell you how heavy it is.

## Small samples

Percentiles need enough data to mean anything. With ten sessions, P95 and P99 are simply the slowest session; with three, the median is the middle one. Pages that rank or compare therefore apply minimum counts: Fleet Health's time attribution needs 20 clean sessions per enrollment class before it shows a number, Geographic Performance only lets locations with at least 3 sessions shape the fleet benchmark, and app-version comparisons need enough installs on both sides. Where a number is shown for a small sample, treat it as an indication, not a measurement.

## Where each statistic appears

Every page states which sessions its figures are computed over. The table below is the cross-reference; the linked pages carry the detail.

| Page | Duration figures | Computed over |
|---|---|---|
| [Dashboard](../portal-guide/dashboard-and-sessions.md#stats-at-the-top) | Median, P90 | Succeeded enrollments of the last 7 days |
| [Fleet Health](../portal-guide/fleet-health.md) | Median, P90; time attribution per enrollment class as median, P90 and P75 | Every session that is no longer running (succeeded, failed and incomplete), in the selected range; time attribution over clean sessions of the last 30 days |
| [Geographic Performance](../portal-guide/geographic-performance.md#how-the-numbers-are-calculated) | Average and P95 per location; a fleet benchmark for *vs Global* | Succeeded enrollments per location in the selected range; the benchmark averages the location averages |
| [Usage Metrics](../portal-guide/usage-metrics.md) | Average, median, P95, P99 | Every session that recorded a duration |
| [SLA Compliance](../portal-guide/sla-compliance.md) | P95 against your target | Every session with a duration in the evaluated week |
| [Software Inventory](../portal-guide/software-inventory-and-vulnerabilities.md) | Median and P95 install time per app and app version | Measured installs only — the final install attempt of each app, skipped installs excluded |

An enrollment duration runs from the agent's first observation on the device to the moment completion is detected (see [How completion is detected](sessions-and-statuses.md#how-completion-is-detected)). For pre-provisioned (White Glove) devices the pause between the technician phase and the user phase is not counted.
