# Log

## 2026-07-21

* **Creation**: Added a **Trust & Security** section — `trust/security-faq.md` and `trust/subprocessors.md` — written to be forwarded as-is to a customer's security or data protection reviewer. The FAQ covers what the agent collects, identity and access, tenant isolation including the delegated (MSP) boundary, data residency and encryption, retention and deletion, GDPR and contracting, and an explicit "What we do not do (yet)" section. The sub-processor page groups every third party by whether customer data actually reaches it. Both pages carry a `timestamp` and a visible "Last reviewed" date, and state that they describe the service as of that date rather than forming a contractual commitment. Records that glueckkanja AG operates the service and is the counterparty on both plans, with Oliver Kieselbach as the named creator and maintainer of the open-source project and the Community edition. Wired into `SUMMARY.md` and `index.md`.

## 2026-07-20

* **Creation**: Added `reference/network-endpoints.md` — customer-facing firewall/proxy allow-list of the outbound HTTPS (443, TLS 1.2+) hosts Autopilot Monitor uses: the four AM-operated endpoints (`download.`, `autopilotmonitor-api-eu.azurewebsites.net`, `www.`, `mcp.`), the feature-dependent diagnostics blob host, and a note on the Microsoft/Azure platform dependencies (Entra sign-in, SignalR, App Insights) already covered by standard M365/Intune baselines. Reused the mTLS TLS-inspection-exclusion warning. Wired into `SUMMARY.md` and `index.md` under Reference; added a cross-link from `getting-started/requirements-and-access.md`.

## 2026-07-18

* **Update**: Added a **Resolved** entry to `troubleshooting/service-announcements.md` — the scheduled infrastructure maintenance completed successfully ahead of the announced window on 18 Jul; platform fully operational, no data lost, irregularities → GitHub issue. Removed the temporary maintenance `warning` hint from the `README.md` Welcome page.

## 2026-07-14

* **Update**: Added a **Scheduled** entry to `troubleshooting/service-announcements.md` announcing planned infrastructure maintenance (Sat 18 Jul – Mon 20 Jul 2026, 00:00 CEST) during which the platform is unavailable; agents buffer locally and re-sync afterwards. Linked from the portal dashboard and landing-page maintenance banners. Also added a temporary `warning` hint at the top of the `README.md` Welcome page pointing to the announcement.

## 2026-07-13

* **Creation**: Added `reference/optional-graph-permissions.md` — customer-facing how-to for opt-in tenant-side Microsoft Graph add-on permissions (`appRoleAssignment` grants via `Grant-AutopilotMonitorAddOn.ps1`), migrated from the contributor tech-docs bundle where it did not belong. Wired into `SUMMARY.md` and `index.md` under Reference; added a pointer from `reference/settings.md`.

* **Creation**: Converted the customer documentation repository into an OKF v0.1 knowledge bundle — added `type` and `tags` YAML frontmatter fields to all 38 content pages (existing GitBook `description` fields kept as the OKF `description`), created `index.md` (bundle entry point) and this log. GitBook structure files (`README.md`, `SUMMARY.md`, `.gitbook.yaml`) are unchanged.
