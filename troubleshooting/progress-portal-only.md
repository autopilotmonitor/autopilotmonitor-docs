---
type: Troubleshooting
tags: [troubleshooting, roles, access, progress-portal]
description: >-
  You sign in and see only the Progress Portal: you have no portal role yet.
  How to get one, and what to do when nobody in your organization looks after
  Autopilot Monitor anymore.
---

# Signed in, but only the Progress Portal

You sign in to Autopilot Monitor and see only **Device Setup Progress**, a search box for a single device, instead of the dashboard. Below the search, a note points you to your organization's Autopilot Monitor admin.

Nothing is broken. Your organization is set up, and you are signed in to it. You just have no portal role yet.

## Why you see only the Progress Portal

What you see depends on your [portal role](../concepts/roles-and-permissions.md). The first person from your organization to sign in became its **Tenant Admin**. Everyone who signs in after that starts without a role, and members without a role see only the [Progress Portal](../portal-guide/progress-portal.md).

## Get a role

Ask your organization's Autopilot Monitor admin to add you under **Settings → Access Management** as Admin, Operator, or Viewer. Then reload the page.

Not sure who your admin is?

* Ask the team that runs Intune or device enrollment in your organization.
* An Entra administrator can see who signs in to Autopilot Monitor: **Microsoft Entra admin center → Enterprise applications → Autopilot Monitor → Sign-in logs**. Entra keeps these logs only for a limited time, depending on your license.

## The note says no device has been monitored recently

The note says this when your organization signed up more than two weeks ago and the portal holds no enrollment session. Then Autopilot Monitor is often left over from a first look: someone signed up, perhaps without the rights to grant the admin consent it needs, and nobody followed up.

Ask your IT department first. If no one looks after it anymore, write to **support@autopilotmonitor.com** from your work address and name your organization's domain:

* If your organization never monitored a device, we add you as its admin and let the current admin know.
* If it did, access stays with its current admins.

To start monitoring after that, someone with the **Global Administrator** or **Privileged Role Administrator** role grants the admin consent. See [Portal Setup](../getting-started/portal-setup.md#2-enable-autopilot-device-validation).

## You have a role, but still see only the Progress Portal

* **The role is brand new.** Reload the page after a minute.
* **The role was assigned in Entra.** If your organization assigns portal roles as app roles on the Autopilot Monitor enterprise application, the role arrives with your next sign-in. Sign out and sign in again.
* **Your entry is disabled.** A disabled entry under **Settings → Access Management** blocks access, even when Entra assigns you a role. Ask your admin to enable it.
