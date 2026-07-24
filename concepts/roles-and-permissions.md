---
type: Concept
tags: [roles, permissions, rbac]
description: >-
  Who can see and do what: tenant roles, the Admin Mode safety toggle, and how
  MSP fleet access fits in.
---

# Roles & Permissions

Access to the portal is controlled by role-based permissions. Everyone signs in with their Microsoft Entra ID account; what they see is determined by the role assigned within Autopilot Monitor.

## Tenant roles

| Role | Permissions |
| --- | --- |
| **Tenant Admin** | Full access to all tenant configuration, sessions, diagnostics, and settings. Manages team members via **Settings → Access Management**, can enable **Admin Mode** for destructive operations. The **first user to sign in** for a tenant is automatically granted this role. |
| **Operator** | Day-to-day monitoring role: views sessions and analytics, sees the tenant's settings read-only (secrets are redacted, and there are no save controls), manages Bootstrap Tokens (if granted), executes device actions such as on-demand log collection, and can submit diagnostic files to support. Configuration changes, member management, validation gates, and offboarding remain with Tenant Admins. Cannot enable Admin Mode and cannot perform destructive operations such as deleting sessions. |
| **Member (no role)** | Only sees the **Progress Portal** — a simplified view for tracking a specific device by serial number. No access to session details, diagnostics, or configuration. Ideal for helpdesk staff or on-site technicians who just need to answer "how far along is this device?" |

Team members are added by UPN under **Settings → Access Management**, where admins can also enable/disable accounts and change roles.

## Admin Mode

Admin Mode is a safety toggle that gates destructive operations. It is only available to Tenant Admins and must be explicitly enabled before any destructive action — preventing accidental deletions during normal day-to-day use.

**How to enable:** click the **gear icon** in the top navigation bar and toggle **Admin Mode** under *Administration*. The toggle turns amber and shows **ON** while active.

| Action | Location | Description |
| --- | --- | --- |
| **Delete Session** | Dashboard → Actions column | Permanently deletes a session and all its event data. The Actions column only appears while Admin Mode is on. |
| **Mark as Failed** | Session detail page | Manually fails a stuck *In Progress*/*Pending* session that will never complete on its own. |
| **Mark as Succeeded** | Session detail page | Manually completes a session; also signals a still-running agent to finish up and clean the device. |

{% hint style="info" %}
Admin Mode is **not persistent** — it lives in the browser's local storage and resets when browser data is cleared. Keep it off unless you are actively performing administrative actions.
{% endhint %}

## MSP / fleet access

For managed service providers, Autopilot Monitor supports **delegated administration**: a delegated admin manages a defined set of customer tenants and gets a **Fleet** view with the same analytics (Fleet Health, Software, Geographic Performance, SLA, Usage) scoped across exactly those tenants — never more.

Delegated administration is an **Enterprise-only** capability: the managing (MSP) tenant must be on the [Enterprise plan](../plans.md). The managed customer tenants can be on any plan, including Community — if the managing tenant is not on Enterprise, the delegated scope is empty and no customer data is accessible.

How the model protects the customer:

| Guarantee | What it means |
| --- | --- |
| **Read-only access** | Delegated principals can view sessions, events, and analytics across their assigned tenants. Write and destructive operations are structurally unavailable — there is no path for a delegated principal to change configuration, delete sessions, or run device actions in a customer tenant. |
| **Secrets redacted** | Configuration is visible in redacted form: secrets such as SAS URLs, tokens, and webhook credentials are never exposed to a delegated reader. |
| **Customer-visible audit trail** | Every grant and revoke of delegated access is written to the audit log of the **managed customer tenant**, not the MSP's own tenant — so the customer can always see who was given access to their data, and when. |

Delegated access is provisioned by the platform operators; if you are an MSP interested in this model, get in touch via the [usual channels](../getting-started/requirements-and-access.md#how-to-request-access).

{% hint style="info" %}
For the deeper explanation of tenant isolation and how delegated scopes are enforced, see [Trust & Security](../trust/security-faq.md).
{% endhint %}
