---
type: Concept
tags: [security, privacy, gdpr, compliance, data-residency, trust]
timestamp: 2026-07-21
description: >-
  Security, privacy, and compliance answers for Autopilot Monitor — data
  residency, tenant isolation, encryption, retention and deletion, delegated
  (MSP) access, external services, and what we deliberately do not do.
---

# Security & Privacy FAQ

**Last reviewed: 21 July 2026 · Next review: 21 January 2027.**

This page answers the questions a security or data protection reviewer asks before Autopilot Monitor is approved for a production fleet. It is written to be forwarded as-is.

It describes how the service is built and operated **as of the review date above** — it is a technical description, not a contractual commitment. The binding documents are the [Terms of Use](https://autopilotmonitor.com/terms), the [Privacy Policy](https://autopilotmonitor.com/privacy), and, on the Enterprise plan, your signed agreement. Where a change here would matter to a customer's own assessment — data residency, external services, what is collected by default, the delegation model — it is announced through [Service Announcements](../troubleshooting/service-announcements.md) rather than quietly edited. If you are relying on a specific statement below, ask and you will get it confirmed for the current release.

Two principles run through every answer below:

* **Secure configuration is the default.** The settings that carry the most weight — hosted diagnostics upload, delegated access — are off, or point at your own storage, until an administrator explicitly turns them on. Where a default is *not* the most restrictive option, this page says so plainly rather than leaving you to find out; geolocation is the notable case.
* **Fail closed, never fail open.** When a check cannot be completed — a certificate chain that cannot be built, an entitlement lookup that errors, a route that nobody registered — the result is denial, not access.

{% hint style="info" %}
Something not answered here? Ask via a [GitHub issue](https://github.com/okieselbach/Autopilot-Monitor/issues) or [LinkedIn](https://www.linkedin.com/in/oliver-kieselbach). Questions asked once tend to end up on this page.
{% endhint %}

## Data Processing and Permissions

### What data does the agent collect from an enrolling device?

Technical enrollment telemetry only. Concretely:

* **Device identity:** serial number, device name, manufacturer, model, and the Entra ID tenant ID the device is enrolling into.
* **Enrollment progress:** phases, ESP stages, application and script install results, policy and CSP activity, reboots, timings, and failure codes.
* **Device context:** OS build, hardware characteristics, disk and network state, and the data your own [gather rules](../rules/gather-rules.md) request.
* **Approximate location**, if enabled — see [Is the device's IP address stored?](#is-the-devices-ip-address-stored) below.

### Does the agent collect the enrolling employee's identity or activity?

No. During Autopilot provisioning the user signs in once to start the process and then does not interact with the device while it provisions. There is no browsing history, no document or file content, no keystrokes, no screen capture, and no application usage tracking. The agent removes itself when enrollment finishes.

Two honest caveats:

* The agent reads the interactive session's user name from Windows to detect *"the desktop is now up and someone is signed in"* and to evaluate exclusion rules. That value drives state transitions; it is not stored as a session attribute.
* If the AutoLogon analyzer is active and the device is configured for automatic logon, the configured default user name can surface as a diagnostic finding — because a device that auto-logs-on is itself the security finding worth reporting.

### Can administrators make the agent collect more?

Yes, through [gather rules](../rules/gather-rules.md) — and that capability is fenced in. `C:\Users` is always blocked for privacy reasons, even in the relaxed guardrail mode. Downloading files, creating users, manipulating boot configuration, and establishing persistence are hard-blocked and cannot be enabled by configuration. Gather rules are authored by your own administrators, visible in the portal, and audited.

### Is the device's IP address stored?

Yes, if geolocation is enabled — and geolocation **is on by default**. Stated precisely, because this is the kind of detail that should not be discovered later:

* When geolocation is enabled, the agent asks a public geolocation service for the approximate location of its outbound connection. The **session record** stores only the derived **country, region, city, and approximate coordinates** — the IP is deliberately excluded from the location event that appears in the timeline.
* The **outbound public IP is additionally recorded once per session as a separate diagnostic event.** That event is hidden from the timeline view by default, but it is stored and queryable, and it is subject to your retention period like any other event.
* **The control that prevents IP collection is switching geolocation off** — a tenant setting, and also an agent command-line switch. With geolocation disabled, no IP and no location data is collected at all, and Geographic Performance simply has no data to show.
* **Request telemetry does not carry device IPs.** The service's own Application Insights masks `client_IP`, and the application never populates it.
* One further server-side case: a **distress report** — the emergency channel an agent uses to report that it is failing — records the reporting IP as part of that incident record.

{% hint style="info" %}
If your data protection assessment requires that no IP addresses are processed, disable geolocation for your tenant. Everything except Geographic Performance keeps working.
{% endhint %}

### Does anything about the service reach an AI or LLM provider?

**The platform makes no LLM API calls.** There is no model provider integration, no API key, and no outbound AI traffic. The semantic search used by the [MCP integration](../integrations/ai-integration-mcp.md) runs locally inside our own container on a small embedding model baked into the image.

What *does* happen: if you connect your own AI assistant through MCP, the data that assistant retrieves is delivered to **your** AI vendor, under **your** agreement with them. That transfer is initiated and controlled by you. MCP access is per-user and must be enabled by an administrator.

### What does the service log about administrator activity?

* An **audit log** per tenant recording action, entity type, entity ID, the acting user's UPN, and a timestamp — visible to you in the portal.
* **Request telemetry** in Application Insights carrying tenant ID, the acting user's UPN, role, correlation ID, and the API route. This is operational telemetry for running and supporting the service.
* **Operational events** for platform-level conditions (device blocked, kill signal delivered, certificate expiry approaching, capacity thresholds).

Portal front-end telemetry is **cookie-free** and carries tenant ID and UI preferences, but **not** user identity.

## Identity and Access

### How does a device prove it is allowed to send data?

With **mutual TLS using the Intune MDM client certificate** the device already holds — no shared secrets, no API keys, nothing that can leak from a script.

The Function App runs with `clientCertMode = Required`. Validation is intentionally strict:

* Trust is **pinned to embedded Intune root CAs** using custom root trust — the operating system trust store is deliberately ignored, so a compromised or overly permissive public CA cannot mint an accepted client certificate.
* The **Client Authentication EKU** is required; validity is checked in UTC.
* If no trust anchors load, validation **fails closed** — the service rejects everything rather than accepting anything.
* Rejections are recorded with structured reasons, so an enrollment that fails authentication is diagnosable without guesswork.

On top of the certificate, the device is checked against **Microsoft Graph** — only devices actually registered as Autopilot devices in your tenant are accepted — and optionally against a hardware allow-list you maintain.

### How do portal users authenticate?

Through **Microsoft Entra ID**, with multi-tenant JWT validation performed by the service. Notable hardening:

* The signing algorithm is restricted to an allow-list, which structurally blocks `alg:none` and HMAC-confusion attacks.
* Identity-library PII logging is switched off.
* Metadata retrieval is HTTPS-only, and the per-tenant metadata cache is bounded so an attacker cannot exhaust memory by presenting a stream of invented tenant IDs.

### What roles exist?

Tenant roles are **Admin**, **Operator**, and **Viewer**, plus a role-less **Member** who can only use the Progress Portal. Platform roles are **Global Admin** and **Global Reader** (cross-tenant read, configuration secrets redacted, no mutations, and raw table access explicitly withheld). Delegated roles for MSP scenarios are **Delegated Reader** and **Delegated Admin** — see [Delegated (MSP) access](#delegated-msp-access).

Role resolution is **table-first with claim fallback**, and a disabled assignment row is an explicit deny that overrides an Entra app-role claim. Revocation therefore wins.

See [Roles & Permissions](../concepts/roles-and-permissions.md) for what each role can do in the portal.

### How is authorization enforced across the API?

Through a single **endpoint access policy catalog**: every HTTP route must be registered with a policy tier and a tenant-scoping mode. **An unregistered route fails closed** — a new endpoint is unreachable until someone deliberately classifies it. There is no second, parallel list of exemptions that can drift out of sync; even the set of routes exempt from JWT validation is derived from the same catalog.

### Are there rate limits?

Yes, all sliding-window:

* **Per device**, keyed on the client certificate thumbprint. Misconfigured or zero limits are clamped upward so a bad configuration cannot fail open.
* **Per portal user**, keyed on UPN.
* **Per MCP user**, plus a daily and monthly quota tied to the tenant's plan.

## Tenant Isolation

### How is my tenant's data separated from other tenants'?

Isolation is enforced in the storage keys themselves, not by a filter that a query could forget. Every tenant-scoped table partitions either on the tenant ID directly or on a composite key prefixed with it, so a query for one tenant cannot structurally return another tenant's rows.

The tenant ID used for scoping comes from the **validated JWT**, not from a request header or body — it is not client-supplied and not forgeable.

Real-time channels are equally scoped. SignalR groups are per tenant, and joining one is authorized against **the group's tenant**, not the caller's home tenant — the distinction that keeps a cross-tenant caller from ever streaming a managed tenant's notifications on the strength of a role they hold somewhere else. Even a same-tenant user without member standing cannot subscribe to organization-wide activity; Progress Portal users see their own enrollment and nothing more.

### Delegated (MSP) access

Delegated administration is the one place where a tenant boundary is crossed — **deliberately, narrowly, and visibly to the customer.** It is an **Enterprise** capability: the managing partner's own tenant must be on the Enterprise plan, verified against the unforgeable tenant claim in their token. Managed customer tenants may be on any plan.

The guarantees:

* **Read-only.** Delegated grants apply only to read policy tiers. Write tiers are structurally unreachable for a delegated principal — this is not a checklist of blocked endpoints but a property of the authorization model. Should delegated write access ever be introduced, it would be an announced change to this page and to the delegation model, not a silent widening of an existing grant.
* **Explicitly scoped.** A delegated principal reaches exactly the tenants with an active, enabled grant — never a tenant beyond that set. Cross-tenant aggregate endpoints that cannot be filtered per tenant are simply not reachable for delegated principals.
* **Secrets redacted.** Configuration secrets are never served to a delegated reader.
* **Consent-aware.** A grant is either assigned centrally by platform operators or delegated by the customer's own tenant admin. Customer-initiated delegations enter a pending-approval state and confer nothing until approved. Only an active grant grants anything.
* **Audited in the customer's own trail.** Every grant, revocation, and disablement is written to the audit log of the **managed customer tenant** — so a customer can always answer *"who was given access to my data, by whom, and when?"* from their own portal. Customers can also query which parties currently hold delegated access to their tenant.
* **Revocation is fast.** Every grant change invalidates the cached scope immediately, and the cache is short-lived by design, so revoking access takes effect within seconds across all service instances — not at the next sign-in.
* **Removed on offboarding.** Delegated grants pointing at an offboarded tenant are deleted with it, so re-onboarding never silently restores someone's old access.

Certain routes are explicitly excluded from delegated reach even though they are read operations — deletion manifests and the customs archive among them.

## Data Protection

### Where is my data stored? Is it in the EU?

Yes, and specifically in Germany.

| Component | Region |
| --- | --- |
| Backend API (Azure Functions), Table Storage, Blob Storage, Queues | **Germany West Central** |
| SignalR (real-time updates) | **Germany West Central** |
| MCP server (Container App), Container Registry | **Germany West Central** |
| Application Insights, Log Analytics | **Germany West Central** |
| Portal front-end (Azure Static Web App) | **West Europe** — static assets only, no customer data |

All customer data — sessions, events, configuration, audit logs, diagnostics, backups — resides in **Germany West Central**. Your data is not replicated to another region, and the platform does not move it outside the EU. Should an additional regional deployment ever be offered (see [Can I run Autopilot Monitor in my own Azure subscription?](#can-i-run-autopilot-monitor-in-my-own-azure-subscription)), it would be a separate deployment a customer opts into — not a relocation of this one. See [Data Flows & External Services](data-flows.md) for the connections a customer can deliberately configure outward.

{% hint style="warning" %}
Some Azure resource names still carry an `…eu` suffix from before the July 2026 migration. Do not infer a region from a resource name — the deployed region is Germany West Central.
{% endhint %}

### Is data encrypted?

* **In transit:** HTTPS only, **TLS 1.2 minimum**, with a hardened cipher floor. Real-time updates use secure WebSockets; the MCP container refuses insecure ingress.
* **At rest:** Azure Storage encryption with **platform-managed keys**, and a storage-account-level TLS 1.2 floor. Customer-managed keys (CMK/BYOK) are **not** currently offered — see [What we do not do](#what-we-do-not-do-yet).

### How does the backend authenticate to its own storage?

With **managed identity**, not storage account keys. Container registry admin access is disabled and images are pulled with managed identity. Deployment pipelines authenticate to Azure with **OIDC federation** — there are no long-lived cloud credentials stored in the CI system.

### How long is data kept, and can I control it?

Retention is **per tenant and configurable by you**, default **90 days**. The permitted range is 7 to 90 days on Community and 7 to 365 days on Enterprise; the plan cap is applied when data is actually deleted, so an out-of-range stored value can never extend retention beyond your plan. Expired sessions are purged automatically.

You additionally control:

* **Delete session** — remove an individual monitoring session on demand.
* **Offboard tenant** — remove your tenant's data and configuration from the service entirely.

### How long is each kind of record kept?

Your retention setting governs enrollment data. The operational records around it have their own fixed lifetimes:

| Record | Kept for |
| --- | --- |
| Enrollment sessions and all their events, findings, and hosted diagnostics | **Your retention setting** — 90 days by default |
| Deletion manifests (what makes a deletion reversible) | **30 days** |
| Audit log of administrative actions | **180 days** |
| Operational events (platform health, security signals) | **90 days** |
| Portal sign-in activity and usage counters | **90 days** |
| Aggregated usage statistics | **180 days** |
| Distress reports (agent emergency channel) | **14 days** |
| Presence (who is currently online) | **1 day** |
| Configuration and authorization backups | Under a storage lifecycle policy, currently **90 days** |

Everything scoped to your tenant is removed when you offboard, regardless of these periods.

Two categories currently have **no time-based expiry** and are removed only on offboarding: product feedback you submitted, and reports an administrator explicitly submitted to us for support. Bringing enrollment bootstrap records under the same expiry as the sessions they belong to is in progress.

### What actually happens when data is deleted?

Deletion is **manifest-first**: before anything is removed, a manifest of exactly what will be deleted is built read-only, and that manifest is itself a faithful copy of the removed rows. Only then does the cascade run, and a verification pass confirms it completed.

That design exists so an accidental deletion is recoverable rather than final — recovery is an operator-assisted action **within 30 days**, while the manifest is retained. It is not a self-service undo button in the portal, and not a substitute for your own records.

Tenant offboarding is a staged cascade: enumeration of every session, a per-session cascade with a drain step that waits for completion, a safe wipe across every tenant-scoped table, blob cleanup, a second audit-log wipe as defence in depth, removal of the tenant configuration row, and a final completion record. Every stage is idempotent, so an interrupted offboarding resumes correctly rather than leaving residue.

Two things survive offboarding, by design and worth stating plainly:

* **Product feedback you submitted** is retained. Free-text comments are the signal that improves the product, and they are not tied to enrollment data.
* **Custom rules and IME log patterns you authored** are archived rather than deleted. This is the community idea working as intended: detection knowledge is what makes this product useful, and a rule that catches a real-world enrollment failure is worth more shared than discarded. Contributed rules and patterns are therefore subject to entering the community pool, so that the next organization hitting the same failure gets a diagnosis instead of a mystery. They are also kept so your own work can be restored if you come back.

Neither contains enrollment telemetry or personal data — a rule is a detection definition, not device data. If you would rather have yours removed, ask and it is done.

### Is data backed up? What is the recovery story?

Honestly and specifically:

* **Configuration, authorization, and rules tables are backed up daily** — exported to a private blob container with a manifest written last as a durability anchor, and aged out by a storage lifecycle policy. Restore is guarded: full-table restore is forbidden on authorization tables and replace-all is forbidden on audit tables, so a restore cannot be used to escalate privilege or erase an audit trail.
* **Session and event telemetry is not backed up.** It is time-bounded operational telemetry with a retention window measured in weeks, reproducible by the next enrollment. Treat Autopilot Monitor as a monitoring system, not a system of record.
* **Storage is locally redundant (LRS)** within Germany West Central. There is no cross-region disaster-recovery failover today — see [What we do not do](#what-we-do-not-do-yet).

### How do diagnostics uploads work? Do my logs leave my tenant?

By default, **no.** Diagnostics upload is **off by default**, and when enabled, the default destination is **your own Azure storage account** via a SAS URL you provide — the package never touches our infrastructure.

Hosted upload exists as an alternative but is **opt-in only** and requires an explicit administrator action in the settings UI behind a clearly marked *"data leaves your tenant"* disclosure. It is never enabled silently.

When hosted upload is used, the SAS we issue is **blob-scoped, write-and-create only, pinned to your tenant's exact path** — it cannot be redirected at another tenant's data — and is short-lived, measured in minutes rather than hours. On the device the SAS is fetched immediately before upload and never written to disk or configuration; log records keep only a truncated prefix, never the signature.

A diagnostics package contains agent logs, Intune Management Extension logs, and session information, with size caps and an explicit truncation record so you know if anything was omitted.

## Security by Design

### What is the general design posture?

* **Never trust the client.** Tenant scoping comes from validated tokens, never from headers. Where a header carries a tenant hint at all, it is honoured only on device routes that independently validate it.
* **Entitlements live in code, not in storage.** Plan limits are defined in an immutable code catalog specifically so that a storage failure can never widen access — a failed lookup resolves to the *least* privileged plan.
* **Prefer platform security to custom code.** Managed identity over secrets, App Service mutual TLS over hand-rolled device auth, Entra ID over local accounts.
* **Minimize blast radius.** Read tiers cannot write. Delegated principals cannot mutate. Global Readers cannot reach raw tables. Restores cannot rewrite audit history.

### Is the agent binary verifiable?

Yes, through four independent layers:

1. **Build provenance:** release packages carry a **Sigstore keyless GitHub artifact attestation**, verifiable with the GitHub CLI against the source repository and workflow that produced them (agent versions published from July 2026 onward).
2. **Publication integrity:** the SHA-256 computed at build time is published in the release manifest and checked by the bootstrapper and self-updater before anything executes.
3. **Independent cross-check:** the expected hash is also served over the authenticated agent configuration endpoint — a second, separate trust channel, so compromising the download alone is not sufficient.
4. **Runtime self-verification:** the running agent hashes itself and raises an emergency alert on mismatch.

Build numbers are reserved with an atomic compare-and-swap so two builds can never claim the same version, and the build fails on any executable/manifest/version inconsistency.

### Can you stop a misbehaving agent in the field?

Yes. There is a **kill switch** delivered over the configuration channel that stops agents by version or by device, plus device blocking for compromised or unauthorized hardware. Both raise operational events and are auditable.

### How is the code tested and reviewed?

Automated tests run in CI across the backend, agent, and MCP server — several hundred test files, including suites written specifically against the security boundaries: authentication middleware, policy enforcement, configuration redaction, revocation enforcement, SSRF protection, audit-log deletion exclusion, and adversarial tests against the MCP raw-data tools. Rule definitions are schema-validated on every push. Every change passes a structured code review before merge.

**Dependency vulnerability scanning is enabled** on the source repository, so vulnerable transitive dependencies are surfaced as they are published rather than discovered at audit time. Deployment pipelines authenticate to Azure with OIDC federation and hold no long-lived cloud credentials.

### How do I report a security vulnerability?

Through **[GitHub Security Advisories](https://github.com/okieselbach/Autopilot-Monitor/security/advisories/new)** — private vulnerability reporting is enabled on the repository, so a report reaches the maintainer without ever being public. Please use that rather than a public issue, LinkedIn, or email.

What to expect: an acknowledgement and an initial assessment of severity, then a fix or a mitigation plan, with a shared view of timing before anything is disclosed. Credit in the advisory if you want it — and none if you would rather stay anonymous. Reports are read by the maintainer directly; no response time is guaranteed.

Security research is welcome and is not a violation of the [Terms of Use](https://autopilotmonitor.com/terms). While testing, please do not access other tenants' data, degrade the service for others, or run automated scanners against production. There is no bug bounty — see [What we do not do](#what-we-do-not-do-yet).

### What happens when there is an incident?

The service raises operational events for the conditions that matter — devices blocked, kill signals delivered, certificate expiry approaching, capacity thresholds, backup and deletion failures — and those alert the operators directly, so problems surface without waiting for a customer to notice.

From there the sequence is: **contain** (kill switch, device block, or disabling the affected path), **investigate** using the audit log and operational telemetry, **remediate and deploy**, then **communicate**. Customer-visible incidents are published in [Service Announcements](../troubleshooting/service-announcements.md) — including what went wrong and what was done about it, not just that something happened. Where an incident affects personal data, notification follows the terms of your data processing agreement and the statutory obligations that apply.

### Does the service make automated decisions about people?

It makes automated decisions about **devices and requests**, not about people, and none of them produce legal or similarly significant effects on an individual:

* **Rate limiting** throttles a device, portal user, or MCP client that exceeds its window.
* **Device blocking** rejects telemetry from a device an administrator has blocked, or one that fails Autopilot validation.
* **The kill switch** stops agents by version or device when a release turns out to be faulty.
* **Analyze rules** classify an enrollment as failed, stalled, or successful — a diagnosis of a machine, which an administrator can always overrule by reading the underlying timeline.

### Are outbound webhooks protected against SSRF?

Yes. Notification webhook targets are screened by an SSRF guard before any request is made, so a webhook URL cannot be pointed at internal addresses or cloud metadata endpoints to make the service fetch something on an attacker's behalf.

## GDPR and Contracts

### Who is the controller and who is the processor?

For enrollment telemetry, **you are the controller** and Autopilot Monitor operates as your **processor**. You decide which devices are monitored, what additional data gather rules collect, how long it is retained, and when it is deleted.

For the operation of the service itself — account administration, the audit trail of portal actions, and operational telemetry — **glueckkanja AG** is the controller.

### Who operates the service, and who is my counterparty?

**glueckkanja AG**, a German company certified to ISO/IEC 27001, operates Autopilot Monitor and is your counterparty — **on both plans**. The service runs in infrastructure operated by glueckkanja AG; the plan is a setting within one system, not a different service or a different operator. Company details are published in the [Imprint](https://www.glueckkanja.com/en/imprint).

**Oliver Kieselbach** created Autopilot Monitor, maintains the open-source project, and is the contact for the Community edition. He acts in that role on behalf of glueckkanja AG and is not a separate contracting party — so the Community edition keeps a named maintainer and an open project, while the operator and the liable party stay unambiguous.

What differs between the plans is commercial, not technical:

* **Community** is free, maintained as an open community contribution, and provided **without commitments by glueckkanja AG** as to availability or support. Support is community-based via [GitHub issues](https://github.com/okieselbach/Autopilot-Monitor/issues).
* **Enterprise** is a written agreement with glueckkanja AG carrying support and reliability commitments, higher operating limits, extended retention, and delegated (MSP) administration.

### Can I get a data processing agreement (DPA / AVV)?

**Yes — available on request**, concluded with glueckkanja AG; a published version is planned. On the Enterprise plan it forms part of the written agreement. Get in touch through the [Imprint](https://www.glueckkanja.com/en/imprint) contact details, or via [LinkedIn](https://www.linkedin.com/in/oliver-kieselbach) or a [GitHub issue](https://github.com/okieselbach/Autopilot-Monitor/issues) for the project side.

The agreement is where the engaged parties and the terms of their engagement are set out. This documentation deliberately does not restate that contractually — it explains the architecture, so you can see what happens technically without waiting for a document. The technical data protection measures are identical on both plans: same region, same isolation, same retention and deletion controls, all described on this page.

### Is glueckkanja AG certified?

**glueckkanja AG is certified to ISO/IEC 27001.** That certification covers the company's information security management system. It is not a certification of this application: Autopilot Monitor itself is not separately certified, and the Azure platform it runs on carries Microsoft's own certifications. Stated this way so nobody mistakes the scope.

### Which external services are involved?

The **data processing agreement** is the authoritative document here — it names the parties engaged and the terms they are engaged on. It is available on request.

For the technical picture, [Data Flows & External Services](data-flows.md) maps every outbound connection. In short: Microsoft Azure (Germany West Central) is the only place your data is stored. Everything else is either a public reference-data source the service reads *from* — nothing about your environment goes out — or a destination **you** configure: notification channels, your own diagnostics storage, your own AI assistant.

### How do I exercise data subject rights?

Contact us and we will act on requests for access, correction, deletion, restriction, or portability within the statutory time limits. Practically, most requests resolve immediately in the portal: you hold delete-session and offboard-tenant controls yourself, and retention is your setting.

### Is there a service level agreement?

* **Community** is a Private Preview with **no availability guarantee** and **community support via GitHub issues**. It is fine for production fleets — with that trade-off understood.
* **Enterprise** is the plan that carries reliability and support commitments. Concrete figures are being finalized alongside pricing; ask and you will get the current draft rather than a placeholder.

Note that "SLA" elsewhere in this documentation ([SLA Compliance](../portal-guide/sla-compliance.md)) means *your* enrollment targets — how fast your Autopilot enrollments should complete — not a commitment about this service's uptime.

### Can I run Autopilot Monitor in my own Azure subscription?

**No, and it is not planned.** Autopilot Monitor is a SaaS service by design. The device agent authenticates with mutual TLS against a centrally pinned Intune CA trust chain, and the detection logic — analyze rules, IME log patterns, decision engine — is continuously updated centrally, which is precisely where the value sits. A self-hosted copy would be frozen at its install date and would need to reproduce the certificate trust model itself.

What we do instead, so that self-hosting is not the only path to data control:

* Data resides in **Germany West Central**, LRS, no replication outward.
* **Diagnostics stay in your own storage account by default** — the largest and most sensitive payload never reaches us unless you deliberately opt in.
* **Retention, session deletion, and full tenant offboarding are your controls**, not a support ticket.
* **Delegated access is opt-in, read-only, and audited in your own trail.**

**Local data residency is a different question, and the answer there is yes in principle.** Running the service in an additional region — a US deployment, for example — is something we can envisage, particularly in an Enterprise context where it is contracted. That is a deployment topology question rather than a change to the trust model, so it is realistic in a way that self-hosting is not.

If a self-hosted deployment or a specific residency region is a hard requirement for your organization, tell us — the requirement is tracked, and residency in particular is the kind of request that gets built when enough organizations ask.

## What we do not do (yet)

Stated explicitly, because a reviewer will find out anyway and the answer is better coming from us:

* **No external penetration test has been commissioned yet**, and there is no bug bounty program. Security work to date is design review, adversarial test suites against the authorization boundaries, dependency vulnerability scanning, and structured code review on every change.
* **No application-level certification.** glueckkanja AG, which operates the service, is ISO/IEC 27001 certified, and Azure carries Microsoft's certifications — but Autopilot Monitor as an application is not separately certified against SOC 2, ISO 27001, or a comparable scheme.
* **No static application security testing (SAST) in CI.** Dependency scanning is enabled; automated code analysis is not.
* **No customer-managed encryption keys (CMK/BYOK).** Encryption at rest uses platform-managed keys.
* **No cross-region disaster recovery.** Storage is locally redundant within Germany West Central; a regional outage means an outage.
* **Session and event telemetry is not backed up** — only configuration, authorization, and rules tables are.
* **No self-hosted deployment option** — see above for the reasoning and the compensating controls.

Several of these are on the path to Enterprise general availability. If one of them blocks an evaluation for you, say which one — that is the most useful signal we can get.

# Citations

* [Terms of Use](https://autopilotmonitor.com/terms) and [Privacy Policy](https://autopilotmonitor.com/privacy) — the binding documents; this page is the technical explanation behind them.
* [Data Flows & External Services](data-flows.md) — every outbound connection and what it carries.
* [Plans](../plans.md) — Community versus Enterprise.
* [Agent Lifecycle & Security](../concepts/agent-lifecycle-and-security.md) — how the agent installs, authenticates, and removes itself.
* [Roles & Permissions](../concepts/roles-and-permissions.md) — the portal permission model.
* [Network Endpoints](../reference/network-endpoints.md) — outbound hosts to allow.
* [Settings Reference](../reference/settings.md) — retention, geolocation, and diagnostics settings.
