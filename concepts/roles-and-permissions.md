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
| **Viewer** | Read-only role: sees everything in the portal — sessions, session details, rules, analytics, settings (secrets redacted), and reports — but cannot change anything or trigger actions. Ideal for auditors, security reviewers, or stakeholders who need full visibility without any write access. |
| **Member (no role)** | Only sees the **Progress Portal** — a simplified view for tracking a specific device by serial number. No access to session details, diagnostics, or configuration. Ideal for helpdesk staff or on-site technicians who just need to answer "how far along is this device?" |

Team members are added by UPN under **Settings → Access Management**, where admins can also enable/disable accounts and change roles. The same page adds a **service principal** (an application in your tenant, by its application ID) for unattended automation over the [MCP integration](../integrations/ai-integration-mcp.md#service-principals-and-automation); a service principal is always a Viewer.

## Admin Mode

Admin Mode is a safety toggle that gates destructive operations. It is only available to Tenant Admins and must be explicitly enabled before any destructive action — preventing accidental deletions during normal day-to-day use.

**How to enable:** click the **gear icon** in the top navigation bar and toggle **Admin Mode** under *Administration*. The toggle turns amber and shows **ON** while active.

| Action | Location | Description |
| --- | --- | --- |
| **Delete Session** | Dashboard → Actions column | Permanently deletes a session and all its event data. The Actions column only appears while Admin Mode is on. Use the **Select** button above the list to pick several sessions and delete them (or block their devices) in one step — at most 100 per action; deleting more than 10 at once requires typing `DELETE`. |
| **Mark as Failed** | Session detail page | Manually fails a stuck *In Progress*/*Pending* session that will never complete on its own. |
| **Mark as Succeeded** | Session detail page | Manually completes a session; also signals a still-running agent to finish up and clean the device. |

{% hint style="info" %}
Admin Mode is **not persistent** — it lives in the browser's local storage and resets when browser data is cleared. Keep it off unless you are actively performing administrative actions.
{% endhint %}

## MSP / fleet access

For managed service providers, Autopilot Monitor supports **delegated administration**: a delegated admin manages a defined set of customer tenants and gets a **Fleet** view with the same analytics (Fleet Health, Software, Geographic Performance, SLA, Usage) scoped across exactly those tenants — never more.

Delegated administration is a **Pro-only** capability: the managing (MSP) tenant must be on the [Pro plan](../plans.md). The managed customer tenants can start on any plan, including Community — if the managing tenant is not on Pro, the delegated scope is empty and no customer data is accessible. Pro includes **two managed tenants**; larger packages raise that limit on request. A pending invitation and a recently removed tenant (24 hours) each keep their slot.

**Managed tenants are on Pro.** A tenant managed by an organization on the Pro plan gets the Pro capabilities for as long as the delegation lasts — its plan badge reads **Pro (MSP)**, also when the tenant has a Pro plan of its own. Conferred Pro does not include delegated administration itself: a managed tenant cannot invite or manage tenants unless it is on Pro in its own right. When the delegation ends, the tenant returns to its own plan; a tenant that had raised its data retention keeps the 30-day grace described under [Plans](../plans.md) before the Community limit applies. A trial of the managing organization does not upgrade its customers.

How the model protects the customer:

| Guarantee | What it means |
| --- | --- |
| **Read-only access** | Delegated principals can view sessions, events, and analytics across their assigned tenants. Write and destructive operations are structurally unavailable — there is no path for a delegated principal to change configuration, delete sessions, or run device actions in a customer tenant. |
| **Secrets redacted** | Configuration is visible in redacted form: secrets such as SAS URLs, tokens, and webhook credentials are never exposed to a delegated reader. |
| **Customer-visible audit trail** | Every grant and revoke of delegated access is written to the audit log of the **managed customer tenant**, not the MSP's own tenant — so the customer can always see who was given access to their data, and when. |
| **AI usage follows the customer's plan** | MCP requests a delegated admin makes into a managed tenant count against **that tenant's** own MCP budget and plan, and the customer's admins see them, marked as delegated, on their MCP Usage page. See [AI Integration (MCP)](../integrations/ai-integration-mcp.md#rate-limits-and-usage-plans). |

### Setting it up yourself

A Pro tenant manages its delegations under **Settings → Tenant → Delegated Access**:

1. **Invite a customer.** Create an invitation link (valid 7 days, works once) and send it to an administrator of the customer tenant. Nothing is granted until they accept.
2. **The customer accepts.** Their tenant administrator opens the link, sees exactly what is granted (read-only, secrets redacted, AI usage on their own budget) and confirms. The grant appears in their audit log immediately.
3. **Assign your team.** Pick which members of your own tenant (Access Management) may read the managed tenants — always read-only.
4. **Remove a tenant.** Either side can end the delegation at any time; access stops immediately. The managing tenant's slot stays occupied for 24 hours after a removal, so a small allowance cannot be rotated through many customers.

Every tenant — including Community — sees under **Delegated Access** who can read it and can end a self-service delegation itself. Delegations provisioned by the platform operators are listed there too; contact support to change those.

{% hint style="info" %}
For the deeper explanation of tenant isolation and how delegated scopes are enforced, see [Trust & Security](../trust/security-faq.md).
{% endhint %}
