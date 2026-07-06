---
description: >-
  The agent CLI reference for testing, debugging, and advanced scenarios —
  including replaying captured enrollments.
---

# Agent Command-Line Parameters

In production the agent needs no parameters — the bootstrap script installs and starts it with everything it needs, and behavior is controlled through [tenant settings](settings.md). The command line exists for **testing, debugging, and advanced scenarios**, launching `AutopilotMonitor.Agent.exe` directly.

## Session & lifecycle

| Parameter | Description |
| --- | --- |
| `--new-session` | Deletes existing session data and starts a fresh session — restart the agent on a device without carrying over stale state. |
| `--no-cleanup` | Suppresses self-destruct after enrollment; agent files and the scheduled task remain for post-enrollment debugging. |
| `--keep-logfile` | Preserves the log directory during self-destruct. |
| `--reboot-on-complete` | Reboots the device after enrollment completes (default delay 10 s, configurable via remote configuration). |
| `--disable-geolocation` | Disables the IP-based geo-location lookup for this run. |

## Authentication & bootstrap

| Parameter | Description |
| --- | --- |
| `--bootstrap-token <token>` | Authenticate with a [Bootstrap Token](bootstrap-script-and-tokens.md#bootstrap-tokens) during OOBE, before an MDM client certificate exists. |
| `--await-enrollment` | Wait for the MDM client certificate to become available before monitoring starts. |
| `--await-enrollment-timeout <minutes>` | Maximum wait for the MDM certificate. Default: `480` (8 hours). |

## Testing & replay

| Parameter | Description |
| --- | --- |
| `--ime-log-path <path>` | Use IME log files from a custom path — test with logs collected from other devices. |
| `--replay-log-dir <path>` | **Replay** real IME logs from a directory, simulating a complete enrollment in fast-forward. Creates a real session in the backend: device info comes from the current machine, enrollment events from the logs. Ideal for testing rules, demos, and analyzing past enrollments. |
| `--replay-speed-factor <n>` | Time compression for replay. Default `50` — a 50-minute enrollment replays in about a minute. Delays between events are divided by the factor, capped at 5 s each. |

### Example — replay a captured enrollment

```powershell
AutopilotMonitor.Agent.exe --replay-log-dir "C:\Logs\IME" --replay-speed-factor 100
```

Replays the captured enrollment at 100× speed and produces a full session on the dashboard — including [analyze rule](../rules/analyze-rules/README.md) evaluation, which makes replay the fastest way to test a custom rule.

{% hint style="warning" %}
Replay and test parameters are for test/development environments — don't use them in production deployments.
{% endhint %}
