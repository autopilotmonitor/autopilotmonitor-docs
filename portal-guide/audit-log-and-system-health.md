---
type: Feature Guide
tags: [portal, audit-log, system-health]
description: >-
  The audit trail of configuration and data changes, and the live platform
  health status page.
---

# Audit Log & System Health

## Audit Log

Every configuration change, user-management action, and data-changing operation in your tenant is recorded — who did what, when, to which entity.

* **Columns:** timestamp, **action** (color-coded: create green, update blue, delete red), entity type, entity ID, performed by, and details. Rows with structured details expand into a key/value breakdown of exactly what changed.
* **Filters:** a from/to date window (defaults to the last 30 days), free-text search (user, entity, action, details), an action filter, and an entity-type dropdown. The default action filter *All (excl. deletions)* hides the noisy per-session retention bookkeeping.

{% hint style="info" %}
The log loads page by page — the free-text search and counts apply to the currently loaded page, not the entire history. Narrow the date window when hunting for something specific.
{% endhint %}

## System Health

The live status page for the platform, from your tenant's perspective. Checks run automatically on load; **Re-check** reruns them.

* An **overall status bar** — *All systems operational*, *Warnings detected*, or *System issues detected* — with a healthy-count badge and last-checked time.
* **Real-Time Connection** — the state of your SignalR live-update connection (the mechanism behind live dashboards and timelines), whether your tenant's update group is joined, and a *Test Group Join* button.
* **Backend Services** — each backend health check as a card with status, message, and expandable details.
* **MCP Server** — the [AI integration](../integrations/ai-integration-mcp.md) endpoint is probed on its own track (it scales to zero and may need a moment to wake), without blocking the other checks.

When something feels broken — dashboards not updating, sessions missing — this page is the first stop to distinguish *your device/agent problem* from *a platform problem*.
