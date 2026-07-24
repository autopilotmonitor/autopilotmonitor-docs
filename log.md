# Log

## 2026-07-24

* **Update**: `concepts/roles-and-permissions.md` — the **Operator** role now sees the tenant's settings read-only: all values are visible with secrets redacted, and the pages carry no save controls. Bootstrap Token management (if granted) is unchanged, and Operators can submit diagnostic files to support. Configuration changes, member management, validation gates, and offboarding remain with Tenant Admins. The `reference/settings.md` introduction states the same split.

* **Update**: `portal-guide/session-details-and-diagnosis.md` — documented **On-demand log collection**: the **Collect Logs** header action on a running session asks the agent to build and upload a diagnostics package immediately (delivered with the agent's next check-in, tracked live on the button, `-server-requested` file-name suffix). Available to Admins and Operators once diagnostics upload is configured; for unconfigured tenants the button is visible but disabled, and a Tenant Admin gets a one-step dialog that enables hosted storage with mode *On Failure Only* and collects right away. `reference/settings.md` (Diagnostics Package → Upload Mode) cross-references the feature and states that on-demand collection works in every enabled mode, including *On Failure Only*.

## 2026-07-23

* **Update**: `rules/gather-rules.md` — the approved registry prefixes now include the Autopilot/OOBE enrollment tracking state (`SOFTWARE\Microsoft\Windows\Autopilot`, covering the ESP's `EnrollmentStatusTracking` policy-provider registrations), so custom rules can target e.g. the presence or absence of a specific ESP policy provider. Enforcement is on the agent; the addition takes effect with the next agent release.

* **Update**: `reference/settings.md` — **Allowed Local Accounts** (Local Admin Analyzer) entries support wildcards: `*` matches any sequence of characters, `?` exactly one (e.g. `adm-*` for generated admin accounts). Matching is case-insensitive and covers account names and profile folders.

## 2026-07-22

* **Update**: Rewrote **Security guardrails** in `rules/gather-rules.md`. A gather rule is stated as a declarative collector definition rather than a script, with a per-collector table of what is enforced, the standing output/count/time limits, and a `danger` hint listing the hard blocks that hold under every configuration — `C:\Users`, the SAM/SECURITY/SYSTEM hives, the Security and PowerShell event logs, downloads, user creation, boot configuration, persistence, and destructive operations. Event Log targets are now allow-listed and documented as such. Points at `rules/guardrails.json` in the public repository as the definitive list. Removed the claim that the allow-lists are displayed inline in the portal; the duplicate Unrestricted Mode hint is gone, and the remaining one states that a tenant administrator cannot enable the mode alone.

* **Update**: `trust/security-faq.md` — expanded *Can administrators make the agent collect more?* along the same lines, added *How do you know the isolation actually holds?* under Tenant Isolation (the endpoint-enumerating catalog test, the tenant-scoping test, and the fail-closed assertions), and rewrote *How is the code tested and reviewed?* into four layers: the suite gating every pull request and merge, CodeQL on both languages, automated dependency updates, and the recurring per-component architecture and security reviews. Review date advanced to 22 July 2026.

* **Update**: `troubleshooting/faq.md` — the data-location answer now leads with Germany West Central as the answer and explains the West Europe front-end as a static-asset host that stores no customer data, rather than presenting the two regions side by side.

## 2026-07-21

* **Update**: This bundle is now the corpus behind the MCP server's `search_docs` tool, which answers product questions from the documentation and cites the page it used. Documented the tool in `integrations/ai-integration-mcp.md` (tool table, how it differs from `search_knowledge`, example prompt). Added a convention to `index.md`: the corpus is built into the MCP container image, so changes here reach AI clients only after an MCP redeploy, and `/health` on the MCP server reports the bundle commit it was built from.

* **Update**: Documented the new **Tenant → Contact** setting — the address used to reach a tenant about the service itself (technical problem, security matter, change needing attention), purpose-limited to service communication and never used for marketing or shared. Added to `reference/settings.md` under Tenant, and to the recommended-configuration list in `getting-started/portal-setup.md`.

* **Creation**: Added a **Trust & Security** section — `trust/security-faq.md` and `trust/data-flows.md` — written to be forwarded as-is to a customer's security or data protection reviewer. The FAQ covers what the agent collects, identity and access, tenant isolation including the delegated (MSP) boundary, data residency and encryption, retention and deletion, contracting, and an explicit "What we do not do (yet)" section. The data-flows page maps every outbound connection by what actually happens to customer data; the contractual side stays with the data processing agreement, which these pages point to rather than restate. Both pages carry a `timestamp` and a visible "Last reviewed" date, and state that they describe the service as of that date rather than forming a contractual commitment. Records that glueckkanja AG operates the service and is the counterparty on both plans, with Oliver Kieselbach as the named creator and maintainer of the open-source project and the Community edition. Wired into `SUMMARY.md` and `index.md`.

## 2026-07-20

* **Creation**: Added `reference/network-endpoints.md` — customer-facing firewall/proxy allow-list of the outbound HTTPS (443, TLS 1.2+) hosts Autopilot Monitor uses: the four AM-operated endpoints (`download.`, `autopilotmonitor-api-eu.azurewebsites.net`, `www.`, `mcp.`), the feature-dependent diagnostics blob host, and a note on the Microsoft/Azure platform dependencies (Entra sign-in, SignalR, App Insights) already covered by standard M365/Intune baselines. Reused the mTLS TLS-inspection-exclusion warning. Wired into `SUMMARY.md` and `index.md` under Reference; added a cross-link from `getting-started/requirements-and-access.md`.

## 2026-07-18

* **Update**: Added a **Resolved** entry to `troubleshooting/service-announcements.md` — the scheduled infrastructure maintenance completed successfully ahead of the announced window on 18 Jul; platform fully operational, no data lost, irregularities → GitHub issue. Removed the temporary maintenance `warning` hint from the `README.md` Welcome page.

## 2026-07-14

* **Update**: Added a **Scheduled** entry to `troubleshooting/service-announcements.md` announcing planned infrastructure maintenance (Sat 18 Jul – Mon 20 Jul 2026, 00:00 CEST) during which the platform is unavailable; agents buffer locally and re-sync afterwards. Linked from the portal dashboard and landing-page maintenance banners. Also added a temporary `warning` hint at the top of the `README.md` Welcome page pointing to the announcement.

## 2026-07-13

* **Creation**: Added `reference/optional-graph-permissions.md` — customer-facing how-to for opt-in tenant-side Microsoft Graph add-on permissions (`appRoleAssignment` grants via `Grant-AutopilotMonitorAddOn.ps1`), migrated from the contributor tech-docs bundle where it did not belong. Wired into `SUMMARY.md` and `index.md` under Reference; added a pointer from `reference/settings.md`.

* **Creation**: Converted the customer documentation repository into an OKF v0.1 knowledge bundle — added `type` and `tags` YAML frontmatter fields to all 38 content pages (existing GitBook `description` fields kept as the OKF `description`), created `index.md` (bundle entry point) and this log. GitBook structure files (`README.md`, `SUMMARY.md`, `.gitbook.yaml`) are unchanged.
