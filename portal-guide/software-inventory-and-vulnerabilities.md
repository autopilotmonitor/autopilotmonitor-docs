---
description: >-
  App install health, the software inventory across enrolled devices, and
  CVE/KEV vulnerability correlation.
---

# Software Inventory & Vulnerabilities

The **Software** area is a three-tab hub with a shared 7/30/90-day time range.

## Installs

The app-install health view across all enrollments:

* Stat cards (total apps, total installs, average failure rate) and a **Delivery Optimization rollup** — peer offload percentage, bytes saved via peers and Connected Cache, and a sourcing breakdown.
* A sortable **apps table**: app (with type badge — Win32, MSI, WinGet, …), installs, succeeded/failed, failure rate (red at ≥ 20 %), average duration, and a **trend** arrow (improving/worsening in percentage points).

### Per-app deep dive

Clicking an app opens its detail page — the place to answer *"is this app hurting our enrollments, and where?"*:

* Summary cards including **Trend** and **Flakiness** (share of installs that needed retries), plus a *"detection lies"* warning when installs reported success but the detection rule couldn't find the app afterwards.
* Charts: installs over time (stacked success/failure), install duration over time, **failure rate by app version**, and installer phase breakdown.
* **Top Failure Codes** with human-readable descriptions from a built-in error-code map, and **Device Model Correlation** with a *lift vs. baseline* multiplier that flags models failing disproportionately for this app.
* An **Affected Sessions** panel (filter Failed/All/Succeeded, configurable columns) linking straight to each session.

## Inventory

The normalized software inventory collected by the **Software Inventory & Vulnerability Analyzer** ([opt-in setting](../reference/settings.md#agent-analyzers)): every discovered title with version, publisher, session count, last-seen date, and whether it is CPE-mapped (the prerequisite for vulnerability matching).

## Vulnerabilities

The exposure panel correlates the inventory against **NVD CVEs**, the **CISA KEV** catalog, and **MSRC**:

* KPI tiles — affected devices, distinct CVEs, and known-exploited (KEV) count — plus a severity breakdown (Critical/High/Medium/Low).
* **Top CVEs by affected devices**, each linked to its NVD entry with CVSS score, a KEV badge for actively exploited vulnerabilities, and sample affected software.

Critical findings also surface directly on the affected sessions via the built-in rule [ANALYZE-ID-003](../rules/analyze-rules/built-in-rules.md#identity--security).

{% hint style="warning" %}
An empty vulnerability list means **no detected CVEs** — not a verified-safe fleet. Scanning must be enabled per tenant, and results are a lower bound if a scan hit its collection cap.
{% endhint %}
