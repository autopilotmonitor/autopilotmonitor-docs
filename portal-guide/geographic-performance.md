---
type: Feature Guide
tags: [portal, geographic, performance]
description: >-
  Enrollment performance by location — find the slow sites, understand why, and
  see how well Delivery Optimization peer caching works per location.
---

# Geographic Performance

When one office keeps reporting "enrollment takes forever" while others are fine, this page turns the anecdote into data. It groups sessions by **city, region, or country** (your choice), over a 7/30/90-day window.

{% hint style="info" %}
Location data comes from the agent's **Geo-Location Detection** ([tenant setting](../reference/settings.md#agent-parameters), IP-based). A coverage note on the page shows how many sessions actually carry location data.
{% endhint %}

## What you see

* **Summary cards** — Locations Tracked, **Outliers Detected** (locations more than 2 standard deviations from the mean), Fastest/Slowest Location, and fleet-wide **DO Peer Efficiency** (what share of content came from Delivery Optimization peer caching instead of the internet).
* **Performance Map** — every location plotted and colored against the global average duration; click to select.
* **Global averages banner** — duration, minutes-per-app, throughput, and peer-to-peer share as the benchmark line.
* **Location Performance table** — per location: sessions, success rate, average and P95 duration (heat-colored vs. global), **App-Load-Score** (a normalized index — under 80 green/good, over 120 red/slow — so a site installing 40 apps isn't unfairly compared with one installing 10), throughput, **P2P %** (with a tooltip breaking peer bytes into LAN / group / internet sources), and **vs Global** (percent faster or slower). Outlier rows carry a red border.

## Drilling in

Clicking a location opens its **session list**: summary cards for that site (sessions, success/failure, average duration, DO peer share) and the individual sessions with per-session peer-caching percentages — each row opens the full [session detail](session-details-and-diagnosis.md).

**Typical findings:** a site with high duration *and* low P2P % usually has a Delivery Optimization / firewall problem; high duration *with* good P2P % points at the WAN link or a proxy; a single-site failure cluster with normal speed elsewhere is usually local infrastructure (certificates, VPN, DNS).
