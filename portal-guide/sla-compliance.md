---
type: Feature Guide
tags: [portal, sla, compliance]
description: >-
  Track enrollment performance against your defined SLA targets — success rate,
  P95 duration, and app install success.
---

# SLA Compliance

If you've committed to enrollment service levels — internally or contractually as an MSP — this page reports against them over a 1/3/6-month window.

## Configuring targets

Targets are defined under **Settings → Tenant → SLA Targets** (the page links you there if none exist yet). Three targets are available, each optional — only configured targets are evaluated:

* **Enrollment Success Rate** (e.g. ≥ 95 %)
* **Max (P95) Enrollment Duration** (e.g. ≤ 60 minutes)
* **App Install Success Rate**

## What you see

* An **overall status banner** — *All SLA Targets Met* or *SLA Targets Breached* — with per-target check marks.
* **Gauge charts** per target comparing the current value to the target (the duration gauge is inverted — lower is better).
* **This week** stat cards (sessions, succeeded, failed, duration violations) and a **weekly trend** line chart of success rate over the selected months.
* **Top Failing Apps** and an **App Install Summary** when app data exists.
* The **SLA Violators** table — every offending session with device (linked), serial, duration, status, a **violation-type badge** (Duration / Failed / Both), and the failure reason. Or, on a good day, a green *No SLA violations*.

### Which period each number covers

| Element | Period |
| --- | --- |
| Banner, **Enrollment Success Rate** and **P95 Enrollment Duration** gauges | The last 30 days, rolling — finished enrollments only (succeeded or failed). |
| **App Install Success Rate** gauge, Top Failing Apps, App Install Summary | The current ISO week. Shown once the week has at least 5 install attempts; skipped apps don't count. |
| **This week** stat cards | The current ISO week (Monday to Sunday, UTC). |
| Weekly trend, SLA Violators | The selected 1/3/6-month window. |

The banner and the gauges use the same periods as the [SLA breach alerts](../integrations/notifications.md), so an alert and this page report the same value. The 30-day window moves with every day, so there is no reset at the start of a month.

A target whose period has no finished enrollment yet shows **No data** instead of a value — an empty period is neither on target nor breached, and it is left out of the banner's count.

{% hint style="info" %}
Metrics are cached for speed — the header shows when they were computed, and **Refresh** recomputes on demand.
{% endhint %}
