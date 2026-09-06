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

* **Search box** — matches device, serial, manufacturer, model, status, session ID, country, agent version, and the OS fields, plus duration expressions like `>30` for enrollments over 30 minutes. Several terms narrow each other down, and `model=` or `manufacturer=` pins a term to one field — see [Search syntax](#search-syntax). Suggestions rank exact matches first, then fuzzy matches; *Search all sessions* extends the search across the full server-side dataset with the same syntax.
* **Status pills** — Succeeded, In Progress, Pending, Stalled, Awaiting User, Failed, Incomplete — each with a live count; click to toggle. *Awaiting User* (Device Setup done, waiting on the user phase) and *Incomplete* (went silent without a completion or failure) come from the [timeout reclassification](../concepts/sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions).
* **Column filters** — most column headers sort, and many offer a filter funnel with checkbox value lists.
* **Global search** — from anywhere in the portal, press **Ctrl+K** (⌘K) to search sessions by serial, device name, or session ID.

### Search syntax

The search box follows the conventions you know from other search boxes — the same ones as the [event timeline search](session-details-and-diagnosis.md#filtering-the-event-timeline):

| You type | You get |
| --- | --- |
| `surface` | Sessions where any searched field contains *surface* |
| `surface failed` | Sessions matching **both** terms — several terms are combined with AND |
| `model=surface` | Only the model field is searched for *surface* |
| `manufacturer="Contoso Ltd"` | A quoted value keeps its spaces; only the manufacturer field is searched |
| `model="EliteBook 840" -failed` | A qualified term combined with an exclusion — a leading minus hides matching sessions |
| `"exit code -1"` | A quoted term is taken literally, leading minus included |
| `>30` | Enrollments longer than 30 minutes (`<`, `>=`, `<=` work too) |

Field names for `field=value` (a colon works as well): `device`, `serial`, `manufacturer`, `model`, `status`, `session`, `country`, `region`, `city`, `agent`, `os`, `build`, `osversion`, `edition`, `language`. Matching is case-insensitive and by substring — `model=elitebook` finds every EliteBook variant; for an exact value use the column filter. Anything that is not a known field name stays an ordinary search term, so a time like `14:30` still works.

Clicking a model on the [Fleet Health](fleet-health.md) page opens the dashboard with exactly this search — `manufacturer=… model=…` — pre-filtered to failed enrollments.

Page size is adjustable (10–100), and a **full-width layout** toggle is available for large screens.

## Admin actions

With [Admin Mode](../concepts/roles-and-permissions.md#admin-mode) enabled, an **Actions** column appears with a delete button per session (confirmation required — deletion permanently removes the session and all its events).

For MSP/delegated admins the dashboard spans the managed tenants (indicated by a banner); regular users without a role never see this page — they are redirected to the [Progress Portal](progress-portal.md).
