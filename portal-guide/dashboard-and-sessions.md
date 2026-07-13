---
type: Feature Guide
tags: [portal, dashboard, sessions]
description: >-
  The session browser — live stats, powerful filtering and search, and the
  admin actions on sessions.
---

# Dashboard & Sessions

The Dashboard is the landing page for admins and operators: every enrollment session in one live-updating table.

## Stats at the top

Five server-aggregated stat cards headline the page — **Active Sessions** (enrolling right now), **Success Rate** (last 7 days), **Avg. Duration** (last 7 days), **Total Today**, and **Failed Today**. A rotating *Tip of the Day* appears below them.

{% hint style="warning" %}
If **Autopilot Device Validation** is disabled, a red *action required* banner appears — agent ingestion is blocked until you enable it (see [Portal Setup](../getting-started/portal-setup.md)).
{% endhint %}

## The sessions table

Default columns: **Device** (name + serial), **Model**, **Status**, **Events**, **Duration**, **Started**. The **Columns** button adds more — Country, Agent Version, OS name/build/version/edition/language — and remembers your choice. Status badges carry qualifier pills where relevant: **Hybrid** (Hybrid Azure AD Join), **Self-Deploying** (kiosk/shared-device profile), and **Blocked**. Clicking a row opens the [session detail](session-details-and-diagnosis.md).

### Finding sessions

* **Search box** — matches device, serial, model, status, session ID, country, and even duration, including numeric expressions like `>30` for enrollments over 30 minutes. Suggestions rank exact matches first, then fuzzy matches; *Search all sessions* extends the search across the full server-side dataset.
* **Status pills** — Succeeded, In Progress, Pending, Stalled, Awaiting User, Failed, Incomplete — each with a live count; click to toggle. *Awaiting User* (Device Setup done, waiting on the user phase) and *Incomplete* (went silent without a completion or failure) come from the [timeout reclassification](../concepts/sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions).
* **Column filters** — most column headers sort, and many offer a filter funnel with checkbox value lists.
* **Global search** — from anywhere in the portal, press **Ctrl+K** (⌘K) to search sessions by serial, device name, or session ID.

Page size is adjustable (10–100), and a **full-width layout** toggle is available for large screens.

## Admin actions

With [Admin Mode](../concepts/roles-and-permissions.md#admin-mode) enabled, an **Actions** column appears with a delete button per session (confirmation required — deletion permanently removes the session and all its events).

For MSP/delegated admins the dashboard spans the managed tenants (indicated by a banner); regular users without a role never see this page — they are redirected to the [Progress Portal](progress-portal.md).
