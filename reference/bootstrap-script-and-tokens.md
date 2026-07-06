---
description: >-
  How the bootstrap script decides to install, and how Bootstrap Tokens enable
  pre-MDM scenarios.
---

# Bootstrap Script & Tokens

## The bootstrap script

`Install-AutopilotMonitor.ps1` is the only thing you deploy (as an Intune platform script — see [Deploy the Agent](../getting-started/deploy-the-agent.md)). On each device it:

1. Evaluates the **pre-requisite guards** — registry deployment marker, no real user profiles, no previous interactive logon, device uptime under 12 hours, agent not already present. All must pass, otherwise the script exits silently without changing anything.
2. **Downloads the agent package** from the distribution endpoint and verifies its **SHA-256 hash** against the published value — a mismatch aborts the installation.
3. Installs the agent, creates a **scheduled task running as SYSTEM**, and starts monitoring.

The full guard list with the exact checks is on the [Deploy the Agent](../getting-started/deploy-the-agent.md#safe-to-assign-broadly) page, together with the read-only **dry-run tester** (`Test-ShouldBootstrapAgent.ps1`) that reports the install decision for any machine without modifying it.

{% hint style="info" %}
The script is **fail-soft by design**: any error path exits without breaking the enrollment. Monitoring is an observer — a bootstrap problem must never cost you a device rollout.
{% endhint %}

Both scripts are public: [`scripts/Bootstrap/` on GitHub](https://github.com/okieselbach/Autopilot-Monitor/tree/main/scripts/Bootstrap).

## Bootstrap Tokens

*(Part of the **Bootstrap Sessions** optional feature — unlocked per tenant on request; the section then appears under Settings → Tenant.)*

Normally the agent authenticates with the device's **MDM client certificate**, and the backend validates the device against your Autopilot registration. Two situations don't fit that model:

* **Very early OOBE** — the agent should start before Intune has issued the MDM certificate.
* **Pre-validation scenarios** — devices that need to register before device validation applies to them (lab devices, special provisioning flows).

Bootstrap Tokens bridge this gap: a token authenticates the session start instead of the certificate, and the agent switches to certificate authentication as soon as the MDM certificate arrives (`--await-enrollment`, default wait up to 8 hours).

### Managing tokens

Under **Settings → Bootstrap Sessions**, admins (and Operators granted token management) can:

* **Create** a token with a validity of 1 h, 4 h, 8 h, 24 h, 48 h, or 7 days. Each token gets a unique **short code and URL**; the copy button provides the URL and a ready-to-use **PowerShell command** for technicians or provisioning scripts.
* **Track** each token's status (Active / Expired / Revoked), creation date, expiry, creator, and **usage count**.
* **Revoke** a token at any time — revocation takes effect immediately.

{% hint style="warning" %}
Treat bootstrap tokens like credentials: choose the shortest validity that covers the job, and revoke tokens that are no longer needed. Devices registered via token still appear as normal sessions and are subject to all other tenant policies.
{% endhint %}
