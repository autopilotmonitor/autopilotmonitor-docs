---
type: Reference
tags: [rules, analyze, built-in]
description: >-
  The complete catalog of built-in analyze rules — what each one detects and
  when it fires.
---

# Built-in Rules Reference

Autopilot Monitor ships with 43 maintained rules. All are enabled by default except the three [template rules](template-rules.md) (marked *off by default*), which need your environment-specific values first.

Most device-phase rules are additionally evaluated **when WhiteGlove pre-provisioning completes** — findings from the technician phase (app failures, low disk, tampering indicators, firmware and clock problems, critical vulnerabilities) appear at the seal, while the technician is still at the device, instead of days later when the user finishes the enrollment. A handful of rules also evaluate the moment their triggering event arrives (e.g. a failed app install, a failed Windows Update, a TPM attestation error). Early findings behave as described in [Concepts → Evaluation triggers](concepts.md#evaluation-triggers-when-a-rule-runs): they are preliminary, notify at most once, and are confirmed or resolved by the final analysis. Rules whose evidence only exists at the end of a session (explicit failure, timeout, user-phase rules) stay enrollment-end only.

Built-in rules are updated with the product — fixes and improvements arrive automatically while your enable/disable choices are preserved. Rules that are retired are cleanly removed from all tenants.

## Apps

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-APP-001** · Win32 App Detection Script Failure | warning | An app installed but its Intune detection rule reported *NotDetected*, so it counts as "not installed" despite the successful install. Suppressed if a later install of the same app succeeded. |
| **ANALYZE-APP-002** · App Installation Error | high | An install failed with a recognizable error code — common MSI exit codes (1603, 1618, 1619, 1625, …) or HRESULTs — and explains what the specific code means. |
| **ANALYZE-APP-003** · App Dependency Chain Failure | high | An app failed because a dependency in its chain failed first. |
| **ANALYZE-APP-004** · Insufficient Disk Space | critical | Free disk space dropped below 5 GB during enrollment. Confidence rises when apps actually failed while space was low. |
| **ANALYZE-APP-005** · Content Download Timeout | high | App content download timed out or failed (Delivery Optimization timeout, download error) — typically network, proxy, or CDN trouble. |
| **ANALYZE-APP-006** · App Install Retry Loop | high | The same app was started 3+ times — an unhonored reboot request, repeated failures, or a detection mismatch causing retries. |
| **ANALYZE-APP-007** · Multiple App Installation Failures | high | Two or more different apps failed in one session — a systemic problem (network, disk, policy) rather than a single bad app. |
| **ANALYZE-APP-008** · Slow App Installation | warning | A single app took longer than 10 minutes to install. |
| **ANALYZE-APP-010** · App Reverted to Error After Successful Install | warning | An app completed successfully but later reported failed for the same app — the post-install re-evaluation flipped it back (e.g. 0x87D13B9C). |
| **ANALYZE-APP-012** · Enforcement Error Resolved by Successful Detection | warning | An "unmapped exit code" failure that a subsequent detection pass resolved — the app is fine, but the Intune return-code mapping should be fixed. |
| **ANALYZE-APP-013** · App Detection Failure During ESP (0x87D1041C) | critical | The classic ESP killer: an app installed fine, but its detection rule didn't match, failing the whole ESP with HRESULT 0x87D1041C. Includes deep remediation (detection-rule bitness, WOW6432Node, testing as SYSTEM). |
| **ANALYZE-APP-014** · MSIX/Store App Failure During ESP (AppX Deployment) | warning | The ESP failed on the Apps subcategory and the AppX deployment log scan identified a suspected MSIX/Store package. Store apps install outside the Win32 pipeline, so regular per-app tracking sees nothing — this correlation names the likely culprit. |
| **ANALYZE-APP-015** · Required App Never Settled — ESP Completion Deferred | high | Intune reported the user session as done, but a required app stayed pending forever — without failing or even starting to install. Typically an assignment or requirement-rule problem (requirements never met, unsatisfied dependency, or an app that needs something unavailable during provisioning, e.g. a VPN client waiting for a portal sign-in). |
| **ANALYZE-CORR-003** · Proxy Configuration Causing Download Failure | high | A proxy/PAC is configured **and** app content downloads failed with download-specific errors — pointing at the proxy blocking Intune content endpoints, with the bypass list to fix it. |
| **ANALYZE-OFFICE-001** · Microsoft 365 Apps Install Failed | warning | The Office Click-to-Run background install never finished — a failure the Intune app status *hides*, because IME reports the M365 app "done" minutes before C2R actually finishes streaming. |

## Enrollment Status Page (ESP)

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-ESP-001** · ESP Blocking App Timeout | high | The DeviceSetup phase exceeded 30 minutes — a blocking app is stuck, slow, or looping. Confidence rises beyond 60 minutes. |
| **ANALYZE-ESP-002** · ESP Subcategory Failed (Certificates) | high | The ESP reported the *Certificates* subcategory as failed — catches certificate/connector problems even when the user clicked "Continue anyway". |
| **ANALYZE-ESP-004** · ESP Timeout with 'Continue Anyway' (Soft Failure) | warning | The ESP hit its terminal timeout, but the profile allows *Continue anyway* — the user most likely reached the desktop on a working, not fully provisioned device. A cluster of these usually means one slow blocking app is holding the fleet at the timeout wall. |
| **ANALYZE-ESP-005** · MDM Policy Forced a Mid-ESP Reboot (Second Sign-In) | warning | Attributes an actually-observed enrollment reboot to device-assigned MDM policies that Windows flagged as reboot-required — the classic "why did the user have to sign in twice?" explained, with the responsible policy URIs named. |
| **ANALYZE-ESP-006** · ESP Failure Recovered After User Retry | info | The recovery story: the ESP reported a terminal failure, the user pressed *Try again*, the failed step re-ran and completed. The earlier failure did not stick — useful context when a session looks worse in the timeline than it ended. |

## Enrollment

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-ENRL-001** · Enrollment Failed | critical | The anchor rule: the enrollment explicitly failed. Extracts the failure reason, failed app, ESP subcategory, and HRESULT into one card, and cross-references the more specific rules that fired alongside it. |
| **ANALYZE-ENRL-002** · Session Timed Out | high | The session stopped sending evidence and hit the backend session timeout — device powered off mid-enrollment, lost connectivity, or genuinely hung. Deliberately doesn't fire when the session already failed or completed. When the evidence shows the user desktop was reached and all tracked apps completed, the finding points at the user phase never signaling completion (see ANALYZE-ID-004). |
| **ANALYZE-ENRL-003** · TPM Attestation Failure — Known Error Code | high | A TPM-attestation error code from Microsoft's Autopilot known-issues list was observed (0x800705B4, 0x801C03EA, 0x81039001, …) — these block self-deploying and pre-provisioning deployments during "Securing your hardware", each with a documented fix. |
| **ANALYZE-ENRL-004** · MDM Enrollment Blocked — Known Error Code | high | An enrollment-blocking error code from Microsoft's known-issues pages was observed (0x80180014, 0x80180018, 0xC1036501, 0x801C03F3, 0x80070774) — device-reuse blocks, licensing/enrollment limits, multiple MDM configurations, deleted Entra device objects, or an Intune Connector domain mismatch. |
| **ANALYZE-ENRL-005** · Hybrid Join Timeout 0x80004005 — Fixed by Windows Update | high | A hybrid-join deployment hit the documented build-dependent 0x80004005 timeout — resolved by specific cumulative updates per Windows version; the finding names the fixed-build thresholds to compare against. |

## Device

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-DEV-001** · Windows Hello Provisioning Timeout | warning | Windows Hello for Business provisioning didn't start within the expected time after ESP — policy, TPM prerequisite, or Key Registration Service connectivity. |
| **ANALYZE-DEV-002** · Sustained High Memory Usage During Enrollment | warning | Memory stayed above 90 % across at least 3 performance snapshots — sustained pressure, not a transient spike. |
| **ANALYZE-DEV-003** · Unsupported Windows Version | warning | The device enrolled with an OS build past end of support — Windows 11 22H2 or older, including every Windows 10 build (LTSC caveat documented in the rule). |
| **ANALYZE-DEV-004** · Windows Update Failed During Enrollment | high | A Windows quality/cumulative update **failed** to install during OOBE/ESP (WindowsUpdateClient EventID 20) — a known enrollment-breaker that the Intune console doesn't surface. Decodes the HRESULT and the update title; confidence rises when the enrollment also failed or a reboot is pending. |
| **ANALYZE-DEV-005** · Windows Update Installed During Enrollment | info | Informational: a Windows quality/cumulative update **installed** mid-enrollment (WindowsUpdateClient EventID 19). Useful context when correlating enrollment duration, an extra reboot, or later device behavior with a specific KB. |
| **ANALYZE-DEV-006** · OS Build Changed During Enrollment | info | Informational: the OS build changed across a reboot during enrollment — deterministic proof that an update was installed, even when the Windows Update event channel shows nothing. |
| **ANALYZE-DEV-007** · System Clock Skew — TPM Attestation and ESP Timeout Risk | warning | The device clock deviates from NTP by more than two minutes — a documented cause of TPM attestation errors (self-deploying/pre-provisioning) and ESP timeouts. |
| **ANALYZE-DEV-008** · Device Clock Wrong During Enrollment — Jump or Sustained Offset | warning | The device clock jumped five or more minutes mid-session, or ran five or more minutes off server time for the whole session — the during-enrollment complement of ANALYZE-DEV-007, catching devices where the NTP check could not run or the clock changed after startup. Clocks off by that much break Kerberos, Entra ID token validation and fresh certificate validity. |
| **ANALYZE-DEV-009** · Battery Critically Low During Enrollment | high | The battery dropped to 15 % or less while the device was enrolling on battery power (detected live by the agent's power watcher). A device dying mid-enrollment is left half-provisioned and usually needs a reset — this fires while there is still time to plug it in. |
| **ANALYZE-DEV-010** · Enrollment Switched From AC to Battery Power | warning | The device lost AC power mid-enrollment and switched to battery. Enrollments drain batteries quickly and Windows may throttle on battery; the timeline shows whether power was restored or the battery kept draining (50/30/15 % threshold events). |

## Identity and Security

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-ID-001** · Expected Machine Certificate Not Found | warning · *off by default (template)* | A certificate with **your configured subject** is missing from `LocalMachine\My` — SCEP/PKCS deployment failure detection. Requires the GATHER-ID-002 gather rule. |
| **ANALYZE-ID-002** · Unexpected Local Admin Accounts Detected | high | The Local Admin Analyzer found admin accounts outside your allow-list — the classic BypassNRO / manipulation signal on a fresh Autopilot device. |
| **ANALYZE-ID-003** · Critical Vulnerability Detected | critical | The software inventory correlated installed software with critical CVEs (CVSS ≥ 9.0) or CISA KEV entries — the device arrives with actively exploited software. |
| **ANALYZE-ID-004** · Hybrid Sign-In Never Establishes Entra User Affinity | high | On a hybrid-joined device the user reached the desktop, but the sign-in never registered with Entra (no Primary Refresh Token) across multiple reboots — the Account Setup page re-appears on every logon and the enrollment never completes. Points at Entra Connect sync scope, UPN mismatch, or missing domain-controller line of sight at logon. Also evaluated *while the enrollment is still running* (see [evaluation triggers](concepts.md#evaluation-triggers-when-a-rule-runs)). |
| **ANALYZE-SEC-001** · Secure Boot UEFI CA 2023 certificate not deployed | warning | Secure Boot is on, but the Windows UEFI CA 2023 certificate is not in the Secure Boot DB. The 2011 certificates expired in June 2026 — affected devices need active remediation to keep receiving Secure Boot updates. The agent verifies this directly against the firmware (UEFI `db`/`KEK` variables): when the firmware already contains the 2023 certificate, the warning is suppressed even if the Windows servicing status has not caught up yet. |
| **ANALYZE-SEC-003** · AutoLogon plaintext password stored in registry | high | A plaintext `DefaultPassword` sits in the Winlogon registry key — a real credential exposure, precisely scoped so normal ESP auto-logon does not trigger it. |
| **ANALYZE-SEC-004** · AutoLogon enabled for an unexpected user | warning · *off by default (template)* | AutoLogon is configured for a user that is not on your approved kiosk-account list. |
| **ANALYZE-SEC-005** · Unexpected Provisioning Package | warning | A provisioning package (PPKG) outside the built-in allow-list (Windows OS-inbox + common OEM factory-preload families) was found — legitimate in bulk-enrollment/OEM-recovery scenarios, a tamper indicator on a pure-Autopilot device. |
| **ANALYZE-SEC-006** · Unexpected Provisioning Package (Custom Allow-List) | warning · *off by default (template)* | The configurable twin of SEC-005: extend the allow-list with your own known-good packages, then disable SEC-005. |
| **ANALYZE-SEC-007** · Provisioning Package Scan Incomplete | warning | The PPKG scan hit its cap and was truncated — coverage honesty for SEC-005/006: some packages may not have been evaluated. |
| **ANALYZE-SEC-008** · Secure Boot is disabled | warning | The device enrolled with Secure Boot turned off — a prerequisite for VBS/Credential Guard, device health attestation, and common compliance policies. |

## Reading a rule's full definition

Every rule's complete definition — conditions, confidence model, full explanation and remediation text — is visible in the portal (expand the rule card, or use **Export** for the raw JSON). The rule sources are also public in the [GitHub repository](https://github.com/okieselbach/Autopilot-Monitor/tree/main/rules/analyze) — community contributions welcome.
