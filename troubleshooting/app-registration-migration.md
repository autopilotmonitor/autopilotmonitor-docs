---
type: How-to Guide
tags: [entra-id, app-registration, migration, consent]
description: >-
  Moving your tenant from the previous Autopilot Monitor app registration to
  the new one: what the two applications are, what users see when signing in
  from a new browser, the one-time consent, and when the previous app can be
  deleted from your tenant.
---

# App Registration Migration

Autopilot Monitor is moving to a new Microsoft Entra multi-tenant app registration. Tenants onboarded before the change still run on the previous registration — everything keeps working as-is, and both registrations run side by side during the migration period. Migrating is a single one-time admin consent done in the portal; everything else happens automatically.

This page explains what the two applications are, what your users see while your tenant has not migrated yet, how to migrate, what to check in your tenant before and after, and what you can clean up afterwards.

## The two applications

| | Application (client) ID | What it is |
| --- | --- | --- |
| **New app** | `886ab5e2-6144-442c-80cc-9b28e0667731` | The current Autopilot Monitor registration. All new tenants consent to this app, and it is the target of the migration. |
| **Previous app** | `1a400946-62c1-4ab4-aa37-f730ac89704d` | The original registration. Tenants onboarded before the migration still run on it until they migrate. It will be retired once the migration period ends. |

Both applications appear in the **Microsoft Entra admin center → Enterprise applications** with the display name **Autopilot Monitor** — always tell them apart by the Application ID, not the name.

The permission set does not change with the migration: the delegated sign-in permission (`User.Read`) and the same read-only Microsoft Graph application permission used for device validation (`DeviceManagementServiceConfig.Read.All`). Migrating does not add or broaden any permission. Optional add-on permissions you may have granted are covered [below](#optional-graph-add-on-permissions).

## Signing in from a new browser or device

The portal signs everyone in with the **new app** by default. A browser that has signed in before remembers which app your tenant uses, so nothing changes for your existing browsers and devices. A **new** browser or device — a colleague's first sign-in, a phone, a helpdesk workstation opening the [Progress Portal](../portal-guide/progress-portal.md) — starts with the new app, and while your tenant still runs on the previous one, that first sign-in shows one of two Microsoft pages:

* **A consent prompt for "Autopilot Monitor"** asking for sign-in and profile access (`User.Read`). Accept it: the sign-in completes, the browser learns your tenant's app, and every later sign-in on that browser is silent.
* **"Need admin approval"** when your organization does not allow users to consent to applications. Click **Return to the application without granting consent** (at the bottom of the page): the portal retries the sign-in with the previous app automatically, and the browser remembers it from then on. There is no need to request approval through that page.

Neither page appears after your tenant has migrated. Migrating (the one-time admin consent below) is therefore the way to give every user a prompt-free first sign-in.

{% hint style="info" %}
If a **Conditional Access** policy in your tenant blocks the new app entirely, the sign-in ends on Microsoft's block page instead. Open the portal once with `?authapp=legacy` appended — for example `https://portal.autopilotmonitor.com/progress?authapp=legacy` — to sign that browser in with the previous app; it remembers the choice. Then update the policy for the new app (see [Before you migrate](#before-you-migrate-settings-on-the-enterprise-application)) and migrate.
{% endhint %}

## How to migrate

1. Sign in to the portal with an Entra ID account that can grant tenant-wide admin consent — **Global Administrator** or **Privileged Role Administrator**.
2. If your tenant still runs on the previous app, the **Dashboard** shows a blue banner **"Please switch to the new Autopilot Monitor app registration"** with a **Switch in Settings** button; the same banner sits at the top of **Settings → Autopilot Validation**. The dashboard banner can be hidden for the current browser tab and returns in the next one until the tenant has migrated.
3. In **Settings → Autopilot Validation**, click **Grant consent for the new app** and complete the Microsoft consent prompt. If the new app has already been consented in your tenant (for example by another admin, or directly in the Entra admin center), use **Detect existing access** instead — no consent redirect needed.
4. The portal verifies the consent and switches your tenant to the new app automatically. A green confirmation banner appears: signed-in users keep working uninterrupted, and every next sign-in uses the new app on its own. Use **Sign in with the new app now** if you want your own browser to switch immediately.
5. Only if your tenant granted the previous app [optional Graph add-on permissions](#optional-graph-add-on-permissions): instead of the green banner, the portal shows **One more step** with the list of those permissions and a PowerShell command pre-filled with the new app's ID. Run it once, then click **Detect existing access** — the switch completes automatically.

The switch only happens once the new app holds every Microsoft Graph application permission the previous app holds in your tenant, so no capability is lost along the way. A sign-in consent alone (the `User.Read` prompt described above) never switches the tenant — only the admin consent from Settings does.

{% hint style="info" %}
A freshly granted consent can take a minute to propagate on Microsoft's side. The portal retries the verification automatically — if it still reports missing permissions right after the consent, wait a minute and use **Detect existing access**.
{% endhint %}

Your end users do not need to do anything, and running enrollments are not affected: the on-device agent authenticates with the device's MDM client certificate, not with the app registration, so telemetry and monitoring continue seamlessly through the switch.

## Before you migrate: settings on the enterprise application

Most tenants configure nothing on the Autopilot Monitor enterprise application itself and can skip this section. Settings you made **on the previous application's enterprise application** in the Entra admin center live on that application only and do not carry over — replicate them on the new application (`886ab5e2-…`) before or right after migrating:

* **Conditional Access policies scoped to the application.** Policies that target *all cloud apps* already cover both applications. A policy (or an exclusion) that names the previous Autopilot Monitor app explicitly needs the new app added.
* **Assignment required.** If you set **Properties → Assignment required? = Yes** on the previous application to limit who can sign in, set it on the new application as well and assign the same users or groups — otherwise, after the migration, any member of your tenant can sign in to the portal (they still only see what their [portal role](../concepts/roles-and-permissions.md) allows; members without a role reach the Progress Portal only).
* **App role assignments** (`Admin` / `Operator` roles assigned on the enterprise application, used only by tenants that opted into Entra app roles). Re-create the assignments on the new application; until then, users signing in with the new app have no role.

The new enterprise application appears in your tenant as soon as the first user or admin consents to it — that is the moment these settings can be replicated.

### Optional Graph add-on permissions

[Optional Graph permissions](../reference/optional-graph-permissions.md) — for example `DeviceManagementScripts.Read.All` for script display names or `CloudPC.Read.All` for [Windows 365 Cloud PC validation](../getting-started/windows-365-cloud-pcs.md) — are tenant-side grants on the service principal of the app your tenant runs on. They are not part of the admin consent, so consenting to the new app does not carry them over, and the portal does **not** switch your tenant while the new app lacks a permission the previous app holds: the feature it powers would silently stop working.

You do not have to work this out yourself. After the consent, **Settings → Autopilot Validation** shows **One more step** with exactly the permissions the new app still lacks and a PowerShell command that grants them on the new app (`886ab5e2-…`). Run it once with an account that can assign application permissions (Global Administrator, Privileged Role Administrator or Cloud Application Administrator — Azure Cloud Shell is the easiest place), then click **Detect existing access**. The switch completes automatically.

{% hint style="info" %}
Use the command from that banner, not the one on **Settings → Optional Graph capabilities**: until the switch, that page pre-fills the client ID of the app your tenant currently runs on — the previous one. After the switch it targets the new app for any further grants.
{% endhint %}

## After the migration

One thing does **not** carry over automatically, because it lives on the previous application's service principal in your tenant:

* **Role assignments made directly on the enterprise application.** Most tenants assign portal roles inside Autopilot Monitor (**Settings → Access Management**) — those are unaffected. Only if you assigned roles on the Autopilot Monitor enterprise application in the Entra admin center do you need to re-create those assignments on the new application (see [above](#before-you-migrate-settings-on-the-enterprise-application)).

Optional Graph add-on permissions are already on the new app at this point — the portal does not switch without them (see [above](#optional-graph-add-on-permissions)). **Settings → Optional Graph capabilities** now targets the new app for anything you grant later.

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

Your tenant consented to the previous registration when it onboarded, and to the new registration when you migrated — or when a user accepted the new app's sign-in prompt from a new browser. During the migration window both are present and both are legitimate. After migrating, the previous one (`1a400946-...`) can be deleted as described above.

</details>

<details>

<summary>A user accepted the new app's sign-in prompt — is my tenant migrated now?</summary>

No. That prompt only grants the sign-in permission (`User.Read`) and creates the new enterprise application in your tenant; it does not move the device-validation permission, and the tenant keeps running on the previous app. Migrating is the admin consent from **Settings → Autopilot Validation**, which the portal verifies before switching the tenant.

</details>

<details>

<summary>Users in my organization keep seeing "Need admin approval" when they sign in.</summary>

Your tenant has not migrated yet and does not allow users to consent to applications, so every first sign-in from a new browser lands on that page; the link at its bottom returns them to the portal, which then signs them in with the previous app. Migrating the tenant removes the page for everyone. Until then, tell users to click **Return to the application without granting consent** rather than requesting approval.

</details>

<details>

<summary>I granted the consent, but the portal still shows a permission error.</summary>

That is almost always the consent propagation window — Microsoft can take a minute to make a fresh tenant-wide consent visible to token requests. The portal retries automatically; if the error persists, use **Detect existing access** in **Settings → Autopilot Validation**, and check under **Enterprise applications → Autopilot Monitor (886ab5e2-…) → Permissions** that `DeviceManagementServiceConfig.Read.All` is listed as granted.

</details>

<details>

<summary>I granted the consent, but the portal shows "One more step" and lists Graph permissions.</summary>

Your tenant granted the previous app one or more [optional Graph add-on permissions](#optional-graph-add-on-permissions), and the new app does not hold them yet. The portal holds the switch so the feature they power keeps working. Copy the command shown in the banner (it targets the new app's ID `886ab5e2-…` and exactly those permissions), run it once with an account that can assign application permissions, then click **Detect existing access**. If the banner persists right after running the script, wait a minute — Microsoft's grant can take a moment to reach token requests — and click **Detect existing access** again.

</details>

<details>

<summary>Does the migration change what data Autopilot Monitor can access?</summary>

No. The new registration carries the same permissions as the previous one: delegated `User.Read` for portal sign-in and the read-only `DeviceManagementServiceConfig.Read.All` application permission for device validation. Optional add-on permissions remain opt-in and are only present if you grant them again explicitly.

</details>
