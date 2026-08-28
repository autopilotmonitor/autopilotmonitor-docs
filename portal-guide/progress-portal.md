---
type: Feature Guide
tags: [portal, progress, real-time]
description: >-
  The simple self-service status page — anyone can follow a device's enrollment
  by serial number, no admin role required.
---

# Progress Portal

The Progress Portal answers the most common question during a rollout — *"how far along is this device?"* — for people who don't need (and shouldn't have) access to session details: helpdesk staff, on-site technicians, or the end users themselves. It is the only page members **without a role** can access.

## How it works

1. Enter the device **serial number** (or its exact device name) and click **Check Status**. For users without a portal role, the serial number is also the access key: the **full, exact serial** is required, and every view of a device — including its live updates — is tied to it. Users who hold a portal role (Admin, Operator, or Viewer) can also search by a partial serial or device name.
2. The page shows a friendly, color-coded status — *Setting up your device…*, *Waiting for you to sign in*, *Setup complete!*, or *Setup encountered an issue* — with the device name and model, an **overall progress bar with percentage**, and the enrollment steps as a list (completed / current / failed). The steps follow the device's scenario: Autopilot with ESP shows the ESP phases (user phases are hidden when the user status page is skipped), [Device Preparation](../getting-started/autopilot-device-preparation.md) shows its own shorter flow, and a pre-provisioned device that is waiting for its user is shown as waiting rather than as a problem. A Windows 365 Cloud PC is found by the device name the Windows App shows and lists only the first-connect steps its user can watch.
3. During the app-installation phases, a live activity panel shows what is currently **downloading** (per-app progress, download rate, bytes) and **installing** (elapsed timer), plus a running *X of Y apps installed* counter.
4. On success the total setup time is shown; on failure a short reason or a *contact your IT department* fallback.

Everything updates live — no refreshing needed. The view is strictly read-only: no filters, no drill-downs, no admin controls, and no access to timelines or device internals.

{% hint style="info" %}
**Rollout tip:** during large deployments, give the field technicians the portal URL and the device serials — they can watch each device's progress without anyone granting portal roles or answering status calls. The serial doubles as the access key, so a technician can only follow devices whose serials they were given.
{% endhint %}
