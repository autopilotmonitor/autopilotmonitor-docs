---
type: Troubleshooting
tags: [diagnostics, logs, collection]
description: >-
  Collecting evidence when something goes wrong — the diagnostics package,
  additional log paths, session reports, and the agent's own logs.
---

# Diagnostics & Log Collection

## The diagnostics package

When enabled, the agent bundles the relevant device logs into a ZIP at the end of enrollment and uploads it — so the evidence is already waiting when you open a failed session (**Download Diagnostics** button on the session detail page).

Configuration lives under **Settings → Agent → Diagnostics Package** ([full reference](../reference/settings.md#diagnostics-package)):

* **Upload destination** — your own Azure Blob Storage (Container SAS URL; data never leaves your tenant) or the built-in hosted storage (short-lived, write-only upload tokens; no storage account needed).
* **Upload Mode** — `Always`, `On Failure Only` (recommended), or `Off`.
* **Additional Log Paths** — extend the default collection with your own files: wildcards in the last path segment, environment variables, and the `%LOGGED_ON_USER_PROFILE%` token for user-profile logs (limited to `AppData\Local`/`Roaming`), optionally with subfolders. Paths are validated against an agent-side allow-list of known log locations.

The default package already covers the essentials: agent logs, IME logs, Panther setup logs, SetupDiag, event logs, and related locations.

## The agent's own log

`%ProgramData%\AutopilotMonitor\Logs` on the device records the agent's startup, guard decisions, collection activity, and every backend interaction. Two settings matter when hunting an agent problem:

* **Keep Log File** — preserves the log through self-destruct.
* **Log Level** — raise to Debug/Verbose only while diagnosing; remember it applies to new enrollments.

For suspected IME-pattern issues there is a dedicated **IME Pattern Match Log** ([details](../rules/ime-log-patterns.md#contributing-patterns)).

## Reporting a session

The **Report Session** action on any session detail page sends a report including your comment and contact address, with optional screenshot and agent-log attachments — the session's timeline exports are attached automatically. Use it when you believe the *product* got something wrong (misclassified session, missing events, a rule misfiring); use the diagnostics package when you're troubleshooting *your* enrollment.
