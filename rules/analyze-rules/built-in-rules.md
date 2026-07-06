---
description: >-
  The complete catalog of built-in analyze rules — what each one detects and
  when it fires.
---

# Built-in Rules Reference

Autopilot Monitor ships with 30 maintained rules. All are enabled by default except the three [template rules](template-rules.md) (marked *off by default*), which need your environment-specific values first.

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
| **ANALYZE-CORR-003** · Proxy Configuration Causing Download Failure | high | A proxy/PAC is configured **and** app content downloads failed with download-specific errors — pointing at the proxy blocking Intune content endpoints, with the bypass list to fix it. |
| **ANALYZE-OFFICE-001** · Microsoft 365 Apps Install Failed | warning | The Office Click-to-Run background install never finished — a failure the Intune app status *hides*, because IME reports the M365 app "done" minutes before C2R actually finishes streaming. |

## Enrollment Status Page (ESP)

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-ESP-001** · ESP Blocking App Timeout | high | The DeviceSetup phase exceeded 30 minutes — a blocking app is stuck, slow, or looping. Confidence rises beyond 60 minutes. |
| **ANALYZE-ESP-002** · ESP Subcategory Failed (Certificates) | high | The ESP reported the *Certificates* subcategory as failed — catches certificate/connector problems even when the user clicked "Continue anyway". |
| **ANALYZE-ESP-004** · ESP Timeout with 'Continue Anyway' (Soft Failure) | warning | The ESP hit its terminal timeout, but the profile allows *Continue anyway* — the user most likely reached the desktop on a working, not fully provisioned device. A cluster of these usually means one slow blocking app is holding the fleet at the timeout wall. |

## Enrollment

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-ENRL-001** · Enrollment Failed | critical | The anchor rule: the enrollment explicitly failed. Extracts the failure reason, failed app, ESP subcategory, and HRESULT into one card, and cross-references the more specific rules that fired alongside it. |
| **ANALYZE-ENRL-002** · Session Timed Out | high | The session stopped sending evidence and hit the backend session timeout — device powered off mid-enrollment, lost connectivity, or genuinely hung. Deliberately doesn't fire when the session already failed or completed. |

## Device

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-DEV-001** · Windows Hello Provisioning Timeout | warning | Windows Hello for Business provisioning didn't start within the expected time after ESP — policy, TPM prerequisite, or Key Registration Service connectivity. |
| **ANALYZE-DEV-002** · Sustained High Memory Usage During Enrollment | warning | Memory stayed above 90 % across at least 3 performance snapshots — sustained pressure, not a transient spike. |
| **ANALYZE-DEV-003** · Unsupported Windows Version | warning | The device enrolled with an OS build past end of support — Windows 11 22H2 or older, including every Windows 10 build (LTSC caveat documented in the rule). |

## Identity & Security

| Rule | Severity | What it detects |
| --- | --- | --- |
| **ANALYZE-ID-001** · Expected Machine Certificate Not Found | warning · *off by default (template)* | A certificate with **your configured subject** is missing from `LocalMachine\My` — SCEP/PKCS deployment failure detection. Requires the GATHER-ID-002 gather rule. |
| **ANALYZE-ID-002** · Unexpected Local Admin Accounts Detected | high | The Local Admin Analyzer found admin accounts outside your allow-list — the classic BypassNRO / manipulation signal on a fresh Autopilot device. |
| **ANALYZE-ID-003** · Critical Vulnerability Detected | critical | The software inventory correlated installed software with critical CVEs (CVSS ≥ 9.0) or CISA KEV entries — the device arrives with actively exploited software. |
| **ANALYZE-SEC-001** · Secure Boot UEFI CA 2023 certificate not deployed | warning | Secure Boot is on, but the Windows UEFI CA 2023 certificate is not in the Secure Boot DB. The 2011 certificates expired in June 2026 — affected devices need active remediation to keep receiving Secure Boot updates. |
| **ANALYZE-SEC-003** · AutoLogon plaintext password stored in registry | high | A plaintext `DefaultPassword` sits in the Winlogon registry key — a real credential exposure, precisely scoped so normal ESP auto-logon does not trigger it. |
| **ANALYZE-SEC-004** · AutoLogon enabled for an unexpected user | warning · *off by default (template)* | AutoLogon is configured for a user that is not on your approved kiosk-account list. |
| **ANALYZE-SEC-005** · Unexpected Provisioning Package | warning | A provisioning package (PPKG) outside the built-in allow-list (Windows OS-inbox + common OEM factory-preload families) was found — legitimate in bulk-enrollment/OEM-recovery scenarios, a tamper indicator on a pure-Autopilot device. |
| **ANALYZE-SEC-006** · Unexpected Provisioning Package (Custom Allow-List) | warning · *off by default (template)* | The configurable twin of SEC-005: extend the allow-list with your own known-good packages, then disable SEC-005. |
| **ANALYZE-SEC-007** · Provisioning Package Scan Incomplete | warning | The PPKG scan hit its cap and was truncated — coverage honesty for SEC-005/006: some packages may not have been evaluated. |

## Reading a rule's full definition

Every rule's complete definition — conditions, confidence model, full explanation and remediation text — is visible in the portal (expand the rule card, or use **Export** for the raw JSON). The rule sources are also public in the [GitHub repository](https://github.com/okieselbach/Autopilot-Monitor/tree/main/rules/analyze) — community contributions welcome.
