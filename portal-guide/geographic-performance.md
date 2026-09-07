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
* **Performance Map** — every location plotted as a marker; a **Color by** selector chooses what the color encodes, a legend under the map explains the active scale, and clicking a marker selects its row. See [Reading the map](#reading-the-map).
* **Global averages banner** — duration, minutes-per-app, throughput, and peer-to-peer share as the benchmark line: your own fleet in the selected range, never other customers.
* **Location Performance table** — per location: sessions, success rate, average and P95 duration (heat-colored vs. global), **App-Load-Score** (a normalized index — under 80 green/good, over 120 red/slow — so a site installing 40 apps isn't unfairly compared with one installing 10), throughput, **P2P %** (with a tooltip breaking peer bytes into LAN / group / internet sources), and **vs Global** (percent faster or slower than the benchmark line). Outlier rows carry a red border. A note under the table states what each figure is computed over; the details are in [How the numbers are calculated](#how-the-numbers-are-calculated).

## Reading the map

Every marker is one location. **Marker size** reflects how many sessions the location had in the selected time range (relative to the busiest location), a **blue ring** marks the location currently selected in the table, and hovering a marker shows all of its values regardless of the active coloring.

The **Color by** selector in the map header chooses which metric the marker color encodes. The legend under the map always describes the active scale, and the same thresholds color the badges in the Location Performance table, so map and table never disagree:

* **Enrollment duration** (default) — compared with the global average: green at or below it (up to 100%), yellow up to 120%, orange up to 150%, red beyond.
* **Success rate** — over finished enrollments only: green from 90%, yellow from 70%, red below. A location where nothing has finished yet is gray.
* **API latency** — the median agent-to-backend round trip on absolute thresholds: green under 250 ms, yellow under 500 ms, orange under 800 ms, red beyond. Latency encodes network distance to the backend region, so this view shows which sites would benefit from a closer region or a shorter route.
* **DO peer efficiency** — the share of content that came from Delivery Optimization peers: green from 50%, yellow from 10%, dark gray when peer caching barely contributes.
* **App-Load-Score** — the normalized per-app duration: green under 80 (faster than the global median), dark gray from 80 to 120 (around it), red above 120 (slower).

Light gray always means "nothing to measure yet" for the active metric. The selection applies to the current page view; the map opens on Enrollment duration every time.

The legend doubles as a filter: click a bucket to show only the locations in it, click further buckets to add them, and click **Show all** (or the last selected bucket again) to return to the full map. The map view and marker sizes stay put while you filter, and the Location Performance table keeps listing every location. Switching **Color by** clears the filter.

## How the numbers are calculated

Every figure on this page comes from the sessions of **your own tenant** in the selected time range, grouped the way you chose. Nothing is compared with other customers.

* **Avg Duration** (per location) — the average enrollment duration of the **succeeded** sessions at that location. Failed and still-running sessions do not enter it. **P95** is the 95th percentile of the same sessions: 95 out of 100 succeeded enrollments at that location finished within this time. Averages, medians and percentiles are explained in [Averages, Medians & Percentiles](../concepts/averages-medians-and-percentiles.md).
* **Global Averages (Benchmark)** — the line above the table is the reference every *vs Global* figure is measured against. Its **Avg Duration** is the average of the per-location Avg Duration values, taken over locations with at least 3 sessions, so every location counts once and a single large site does not define the benchmark for everyone else. **Median** is the median of those same per-location values: half of your locations are faster than it, half slower. *Avg Min/App* and *Avg Throughput* follow the same location-by-location rule; *API Latency* and *DO P2P* are computed over all sessions or all bytes instead, because a per-location average would hide the busy sites.
* **vs Global** — the location's Avg Duration compared with the benchmark's Avg Duration, as a percentage: *+20 %* means enrollments at this location take on average a fifth longer than at a typical location in your fleet, *−15 %* means they finish faster. It is an average-against-average comparison; P95 and median play no part in it. A dash means the location has no succeeded enrollment yet, sits exactly on the benchmark, or there is no benchmark yet (fewer than 3 sessions everywhere).
* **Outliers** — a location is flagged when its Avg Duration lies more than 2 standard deviations away from the benchmark, in either direction, and it has at least 3 sessions. This is the same threshold as the *Outliers Detected* card.
* **App-Load-Score** — the location's minutes per app divided by the fleet's **median** minutes per app, times 100. 100 is the typical location; under 80 is fast, over 120 slow.
* **API Latency** — the median agent-to-backend round trip per location; its *vs global* tooltip compares that median with the median across all sessions, so a single distorted session never skews either side.

## Drilling in

Clicking a location opens its **session list**: summary cards for that site (sessions, success/failure, average duration, DO peer share) and the individual sessions with per-session peer-caching percentages — each row opens the full [session detail](session-details-and-diagnosis.md).

**Typical findings:** a site with high duration *and* low P2P % usually has a Delivery Optimization / firewall problem; high duration *with* good P2P % points at the WAN link or a proxy; a single-site failure cluster with normal speed elsewhere is usually local infrastructure (certificates, VPN, DNS).
