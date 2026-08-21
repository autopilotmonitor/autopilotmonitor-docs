---
type: How-to Guide
tags: [entra-id, app-registration, migration, consent]
description: >-
  Moving your tenant from the previous Autopilot Monitor app registration to
  the new one: what the two applications are, the one-time consent, and when
  the previous app can be deleted from your tenant.
---

# App Registration Migration

Autopilot Monitor is moving to a new Microsoft Entra multi-tenant app registration. Tenants onboarded before the change still run on the previous registration — everything keeps working as-is, and both registrations run side by side during the migration period. Migrating is a single one-time admin consent done in the portal; everything else happens automatically.

This page explains what the two applications are, how to migrate, and what you can clean up in your tenant afterwards.

## The two applications

| | Application (client) ID | What it is |
| --- | --- | --- |
| **New app** | `886ab5e2-6144-442c-80cc-9b28e0667731` | The current Autopilot Monitor registration. All new tenants consent to this app, and it is the target of the migration. |
| **Previous app** | `1a400946-62c1-4ab4-aa37-f730ac89704d` | The original registration. Tenants onboarded before the migration still run on it until they migrate. It will be retired once the migration period ends. |

Both applications appear in the **Microsoft Entra admin center → Enterprise applications** with the display name **Autopilot Monitor** — always tell them apart by the Application ID, not the name.

The permission set does not change with the migration: the delegated sign-in permission (`User.Read`) and the same read-only Microsoft Graph application permission used for device validation (`DeviceManagementServiceConfig.Read.All`). Migrating does not add or broaden any permission. Optional add-on permissions you may have granted are covered [below](#after-the-migration).

## How to migrate

1. Sign in to the portal with an Entra ID account that can grant tenant-wide admin consent — **Global Administrator** or **Privileged Role Administrator**.
2. Open **Settings → Autopilot Validation**. If your tenant still runs on the previous app, a banner **"Please switch to the new Autopilot Monitor app registration"** is shown at the top.
3. Click **Grant consent for the new app** and complete the Microsoft consent prompt. If the new app has already been consented in your tenant (for example by another admin, or directly in the Entra admin center), use **Detect existing access** instead — no consent redirect needed.
4. The portal verifies the consent and switches your tenant to the new app automatically. A green confirmation banner appears: signed-in users keep working uninterrupted, and every next sign-in uses the new app on its own. Use **Sign in with the new app now** if you want your own browser to switch immediately.

{% hint style="info" %}
A freshly granted consent can take a minute to propagate on Microsoft's side. The portal retries the verification automatically — if it still reports missing permissions right after the consent, wait a minute and use **Detect existing access**.
{% endhint %}

Your end users do not need to do anything, and running enrollments are not affected: the on-device agent authenticates with the device's MDM client certificate, not with the app registration, so telemetry and monitoring continue seamlessly through the switch.

## After the migration

Two things do **not** carry over automatically, because they live on the previous application's service principal in your tenant:

* **Optional Graph add-on permissions.** If you granted opt-in add-on permissions — for example `CloudPC.Read.All` for [Windows 365 Cloud PC validation](../getting-started/windows-365-cloud-pcs.md) — re-grant them against the new app. Open **Settings → Optional Graph capabilities** and run the grant command again; it is pre-filled with the new app's client ID. See [Optional Graph Permissions](../reference/optional-graph-permissions.md).
* **Role assignments made directly on the enterprise application.** Most tenants assign portal roles inside Autopilot Monitor (**Settings → Access Management**) — those are unaffected. Only if you assigned roles on the Autopilot Monitor enterprise application in the Entra admin center do you need to re-create those assignments on the new application.

## Deleting the previous app

Once your tenant is migrated, the previous application serves no purpose in your tenant anymore and can be deleted:

1. Confirm the migration is complete: **Settings → Autopilot Validation** no longer shows the migration banner, and a fresh portal sign-in works normally.
2. In the **Microsoft Entra admin center**, open **Enterprise applications** and search for *Autopilot Monitor*.
3. Open the entry whose Application ID is `1a400946-62c1-4ab4-aa37-f730ac89704d` — this is the previous app.
4. Under **Properties**, choose **Delete**.

Deleting the enterprise application removes the consent your tenant granted to the previous app, including any add-on permissions still assigned to it.

{% hint style="warning" %}
Delete only the application with Application ID `1a400946-62c1-4ab4-aa37-f730ac89704d`, and only **after** your tenant has migrated. Deleting it while your tenant still runs on it breaks portal sign-in and device validation — agent uploads are rejected until you complete the migration to the new app.
{% endhint %}

## Frequently asked questions

<details>

<summary>Is there a deadline for the migration?</summary>

The previous app keeps working throughout the migration period — the banner in the portal is an invitation, not an outage warning. The retirement of the previous registration will be announced in advance via [Service Announcements](service-announcements.md) and the tenant contact address.

</details>

<details>

<summary>Why are there two "Autopilot Monitor" applications in my tenant?</summary>

Your tenant consented to the previous registration when it onboarded, and to the new registration when you migrated. During the migration window both are present and both are legitimate. After migrating, the previous one (`1a400946-...`) can be deleted as described above.

</details>

<details>

<summary>I granted the consent, but the portal still shows a permission error.</summary>

That is almost always the consent propagation window — Microsoft can take a minute to make a fresh tenant-wide consent visible to token requests. The portal retries automatically; if the error persists, use **Detect existing access** in **Settings → Autopilot Validation**, and check under **Enterprise applications → Autopilot Monitor (886ab5e2-…) → Permissions** that `DeviceManagementServiceConfig.Read.All` is listed as granted.

</details>

<details>

<summary>Does the migration change what data Autopilot Monitor can access?</summary>

No. The new registration carries the same permissions as the previous one: delegated `User.Read` for portal sign-in and the read-only `DeviceManagementServiceConfig.Read.All` application permission for device validation. Optional add-on permissions remain opt-in and are only present if you grant them again explicitly.

</details>
