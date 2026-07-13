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

1. Enter the device **serial number** (or device name) and click **Check Status**.
2. The page shows a friendly, color-coded status — *Setting up your device…*, *Setup complete!*, or *Setup encountered an issue* — with the device name and model, an **overall progress bar with percentage**, and the seven enrollment phases as a step list (completed / current / failed).
3. During the app-installation phases, a live activity panel shows what is currently **downloading** (per-app progress, download rate, bytes) and **installing** (elapsed timer), plus a running *X of Y apps installed* counter.
4. On success the total setup time is shown; on failure a short reason or a *contact your IT department* fallback.

Everything updates live — no refreshing needed. The view is strictly read-only: no filters, no drill-downs, no admin controls, and no access to timelines or device internals.

{% hint style="info" %}
**Rollout tip:** during large deployments, give the field technicians the portal URL and the device serials — they can watch each device's progress without anyone granting portal roles or answering status calls.
{% endhint %}
