---
description: Known failure patterns and how to resolve them, from silent agents to proxy trouble.
---

# Common Problems

## No sessions appear in the portal

The agent is deployed but the dashboard stays empty. Check in this order:

1. **Autopilot Device Validation enabled?** Without it, the backend rejects *all* agent uploads — the dashboard shows a red banner in that case. Enable it under Settings ([Portal Setup](../getting-started/portal-setup.md)).
2. **Is the device eligible?** It must be registered in Intune as an Autopilot device (or match your Corporate Identifiers), and its manufacturer/model must pass the [Hardware Whitelist](../reference/settings.md#hardware-whitelist).
3. **Did a bootstrap guard skip the device?** Already deployed once (registry marker), existing user profiles, prior interactive logon, or more than 12 hours of uptime all cause a silent skip. Run the [dry-run tester](../getting-started/deploy-the-agent.md#try-it-first-dry-run) on the device to see the exact decision.
4. **Network path clear?** The device needs outbound HTTPS to the backend. Watch for proxies requiring authentication during OOBE and for **TLS inspection** (next section).
5. **Agent log:** `%ProgramData%\AutopilotMonitor\Logs` on the device shows registration and upload errors in detail.

## TLS inspection breaks agent authentication

The agent authenticates with the device's **MDM client certificate** (mutual TLS). A proxy or firewall doing break-and-inspect terminates the TLS session and drops the client certificate — the backend then rejects the connection.

**Fix:** exclude the Autopilot Monitor backend endpoints from TLS inspection. The same applies to proxies that require user authentication during the device phase of OOBE (no user exists yet to authenticate).

## Sessions stuck "In Progress"

Usually the completion signal was missed (device powered off or rebooted at the wrong moment, connectivity lost). This self-heals: the backend marks the session **Failed – Timed Out** after the [session timeout](../reference/settings.md#data-management) (default 5 hours), and on the device the agent's own 6-hour lifetime plus the unconditional 48-hour emergency brake guarantee cleanup. If you know the outcome, mark the session succeeded/failed manually via [Admin Mode](../concepts/roles-and-permissions.md#admin-mode).

If it happens *consistently*, check whether something on the device kills connectivity mid-enrollment — security clients activating during ESP (VPN/SASE agents, e.g. Zscaler or Cloudflare One in strict modes) are the classic cause; give the Autopilot Monitor endpoints a gateway/split-tunnel exception.

## App failures dominate the timeline, but the apps are fine

Two classic Intune patterns the built-in rules recognize:

* **Detection rule mismatch** — the app installs, but its detection rule doesn't match, so Intune reports failure (HRESULT `0x87D1041C` when it kills the ESP). See the findings of [ANALYZE-APP-001/APP-013](../rules/analyze-rules/built-in-rules.md#apps) for concrete remediation.
* **Unmapped exit code** — the installer returns an exit code that isn't mapped in the app's return-code table; a later detection pass often flips it to success ([ANALYZE-APP-012](../rules/analyze-rules/built-in-rules.md#apps)). Fix the return-code mapping in Intune.

## Delivery Optimization metrics are missing

For devices running **IME 1.101 or later**, Microsoft removed DO telemetry from the IME log. Autopilot Monitor compensates by collecting DO data at the OS level, so current agents show peer-caching metrics again — but sessions recorded in the gap may lack them. Background in the [Service Announcements](service-announcements.md).

## An IME log line isn't recognized anymore

Microsoft occasionally changes IME log formats, which can silently break a parsing pattern. Enable the **IME Pattern Match Log** and follow the [contribution workflow](../rules/ime-log-patterns.md#contributing-patterns) — a fixed pattern helps every tenant.

## Live updates stopped working

If dashboards or timelines stop updating in real time, check [System Health](../portal-guide/audit-log-and-system-health.md#system-health): the **Real-Time Connection** card shows your SignalR state and offers a group-join test. A page reload re-establishes the connection; data is never lost by a dropped live connection — it's a display concern only.

## Still stuck?

Collect a [diagnostics package](diagnostics-and-log-collection.md) and use **Report Session** on the affected session — it bundles the context needed to investigate.
