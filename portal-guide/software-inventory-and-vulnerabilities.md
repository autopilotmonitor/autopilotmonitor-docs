---
type: Feature Guide
tags: [portal, software-inventory, vulnerabilities, cve]
description: >-
  App install health, the software inventory across enrolled devices, and
  CVE/KEV vulnerability correlation.
---

# Software Inventory & Vulnerabilities

The **Software** area is a three-tab hub with a shared 7/30/90-day time range.

## Installs

The app-install health view across all enrollments:

* Stat cards (total apps, total installs, average failure rate) and a **Delivery Optimization rollup** — peer offload percentage, bytes saved via peers and Connected Cache, and a sourcing breakdown.
* A sortable **apps table**: app (with type badge — Win32, MSI, WinGet, …), installs, succeeded/failed, failure rate (red at ≥ 20 %), average install time (the final install attempt — see [Fleet Health](fleet-health.md) for what counts), and a **trend** arrow (improving/worsening in percentage points).

### Per-app deep dive

Clicking an app opens its detail page — the place to answer *"is this app hurting our enrollments, and where?"*:

* Summary cards including **Trend** and **Flakiness** (share of installs that needed retries), plus a *"detection lies"* warning when installs reported success but the detection rule couldn't find the app afterwards.
* Charts: installs over time (stacked success/failure), install time over time, **failure rate by app version**, **median install time by app version** (measured installs only), and installer phase breakdown. Install times measure the final install attempt; in the Affected Sessions panel, an app the Intune Management Extension processed in multiple passes carries a small ×2 marker explaining why it finished long after it was first seen.
* A **duration regression banner** when a newly rolled-out version installs markedly slower than its predecessor — *"median install duration rose from 2.0 to 7.0 min after version X"*, with the sample counts behind both medians. The comparison uses medians over measured installs only and needs enough samples on both sides, so a single outlier never raises it; tenant admins also get a one-time [notification](../integrations/notifications.md#in-portal-alerts) per (app, version). The banner clears on its own once the version's median recovers or the version is superseded.
* **Top Failure Codes** with human-readable descriptions from a built-in error-code map, and **Device Model Correlation** with a *lift vs. baseline* multiplier that flags models failing disproportionately for this app.
* An **Affected Sessions** panel (filter Failed/All/Succeeded, configurable columns) linking straight to each session.

## Inventory

The normalized software inventory collected by the **Software Inventory & Vulnerability Analyzer** ([opt-in setting](../reference/settings.md#agent-analyzers)): every discovered title with version, publisher, session count, last-seen date, and whether it is CPE-mapped (the prerequisite for vulnerability matching).

## Vulnerabilities

The exposure panel correlates the inventory against **NVD CVEs**, the **CISA KEV** catalog, **MSRC**, and **FIRST EPSS**:

* KPI tiles — affected devices, distinct CVEs, and known-exploited (KEV) count — plus a severity breakdown (Critical/High/Medium/Low).
* **Top CVEs by affected devices**, each linked to its NVD entry with CVSS score, a KEV badge for actively exploited vulnerabilities, its EPSS score, a priority label, and sample affected software.

Two signals answer different questions, and the portal shows both:

* **CVSS** (severity) says how bad a successful exploit would be. The session report also shows the CVSS vector, so you can tell a network-reachable flaw (`AV:N`) from one that needs local access.
* **EPSS** (likelihood) is FIRST.org's estimated probability that the CVE is exploited in the wild within the next 30 days — `EPSS 12.3%` means 12.3 %. Scores are refreshed daily. A CVE without an EPSS pill has not been scored yet; that is unknown, not safe.

The **priority** label combines them into a remediation order: **Act** — listed in CISA KEV, exploitation is confirmed; **Attend** — EPSS of 10 % or more, or CVSS 9.0 and above; **Track** — everything else. A low-CVSS vulnerability with a high EPSS is still an *Attend*.

Critical findings also surface directly on the affected sessions via the built-in rule [ANALYZE-ID-003](../rules/analyze-rules/built-in-rules.md#identity-and-security).

{% hint style="warning" %}
An empty vulnerability list means **no detected CVEs** — not a verified-safe fleet. Scanning must be enabled per tenant, and results are a lower bound if a scan hit its collection cap.
{% endhint %}
