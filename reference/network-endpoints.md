---
type: Reference
tags: [network, firewall, endpoints, requirements]
description: >-
  The outbound HTTPS endpoints Autopilot Monitor uses, for firewall, proxy, and
  gateway allow-lists. All traffic is outbound HTTPS on port 443 (TLS 1.2+).
---

# Network Endpoints

Autopilot Monitor communicates exclusively over **outbound HTTPS on port 443 (TLS 1.2 or newer)**. There is **no inbound connectivity** to your network and no non-standard ports. If your enrolling devices or admin workstations sit behind a firewall, proxy, or secure web gateway that filters egress, allow the hosts below.

## Autopilot Monitor endpoints

These four hosts are operated by Autopilot Monitor and are the ones you may need to add explicitly to an allow-list.

| Host | Purpose | Who connects |
| --- | --- | --- |
| `download.autopilotmonitor.com` | Agent package + integrity manifest download (bootstrap) | Enrolling devices |
| `autopilotmonitor-api-eu.azurewebsites.net` | Telemetry ingest and portal API | Enrolling devices **and** admin browsers |
| `www.autopilotmonitor.com` | Portal web app (sign-in and UI) | Admin browsers |
| `mcp.autopilotmonitor.com` | Read-only MCP telemetry server (optional AI integration) | AI/MCP clients |

{% hint style="info" %}
`autopilotmonitor.com` redirects to `www.autopilotmonitor.com`, and this documentation is served from `docs.autopilotmonitor.com` — allowing the `autopilotmonitor.com` parent domain and its subdomains covers all of them.
{% endhint %}

Note the two audiences usually live on different network segments: **enrolling devices** (agent) need `download.` and the API host; **administrators** (portal, AI clients) need `www.`, the API host, and — for AI integration — `mcp.`.

## Feature-dependent endpoint

| Host | Purpose | When needed |
| --- | --- | --- |
| `autopilotmonitoreu.blob.core.windows.net` | Diagnostics/log upload from the device via short-lived SAS | Only if you enable **diagnostics collection** for a tenant |

## Microsoft platform dependencies

The portal (in the browser) and Entra ID sign-in also rely on standard Microsoft/Azure endpoints. These are typically **already permitted** by the Microsoft 365 / Intune baseline allow-lists most organizations run, so you rarely need to add them for Autopilot Monitor specifically:

* `login.microsoftonline.com` — Microsoft Entra ID sign-in for the portal
* `*.service.signalr.net` (HTTPS **and** WebSocket / `wss`) — real-time portal updates
* `*.in.applicationinsights.azure.com`, `js.monitor.azure.com` — portal telemetry

The on-device agent does **not** use Entra ID — it authenticates with the device's MDM client certificate — so the agent network only needs the Autopilot Monitor hosts above.

## TLS inspection

{% hint style="warning" %}
If your network uses TLS inspection (break-and-inspect), **exclude `autopilotmonitor-api-eu.azurewebsites.net` and `download.autopilotmonitor.com` from interception on the enrolling-device path.** The agent authenticates with the device's **MDM client certificate**, and client-certificate authentication does not survive TLS interception. See [Requirements & Access](../getting-started/requirements-and-access.md) and [Agent Lifecycle & Security](../concepts/agent-lifecycle-and-security.md).
{% endhint %}
