---
description: >-
  Collect custom evidence during enrollment — registry, WMI, files, event logs,
  and allow-listed commands, with strict security guardrails.
---

# Gather Rules

Gather rules define **what extra data the agent collects** from the device during enrollment. Each rule specifies a **collector type** (how to collect), a **target** (what to collect), optional **parameters**, and a **trigger** (when to collect). Results arrive as events in the session timeline — where [analyze rules](analyze-rules/README.md) can grade them automatically.

A set of maintained built-in and community gather rules ships with the product; your own rules are created on the **Gather Rules** page.

## Collector types

| Collector | Collects | Target example |
| --- | --- | --- |
| **Registry** | Values of a key (or subkey names with `listSubkeys`) | `HKLM\SOFTWARE\Microsoft\PolicyManager\current\device` |
| **Event Log** | Entries from classic or operational Windows event logs, filterable by event ID, source, and message | `Microsoft-Windows-Shell-Core/Operational`, event ID `62407`, filter `*ESPProgress*` |
| **WMI Query** | Results of a full WQL `SELECT` statement against allow-listed classes | `SELECT * FROM Win32_NetworkAdapterConfiguration` |
| **File** | File/directory existence, size, and optionally content (last 4,000 characters, files < 50 KB) | `C:\Windows\Panther\setuperr.log` with `readContent: true` |
| **Command (Allowlisted)** | Output of a **pre-approved** PowerShell/CLI command — exact allow-list match only | `Get-Tpm` |
| **Log Parser** | Regex matches (named capture groups) from log files — CMTrace or plain-text format, with position tracking and filename wildcards | `%ProgramData%\...\IntuneManagementExtension\Logs\AppWorkload*.log` |
| **JSON (JSONPath)** | Values extracted from a JSON file via JSONPath (file < 200 KB) | `$..DetectionScript` |
| **XML (XPath)** | Elements/attributes extracted from an XML file via XPath, with namespace support | `//ns:setting[@name='ComputerName']/@value` |

{% hint style="info" %}
**Targeting the user profile:** the agent runs as SYSTEM, so `%USERPROFILE%` resolves to the SYSTEM profile. Use the special token `%LOGGED_ON_USER_PROFILE%` to reach the signed-in user's profile (only `AppData\Local` and `AppData\Roaming` are allowed). Rules using the token are skipped automatically until a user session exists.
{% endhint %}

## Triggers

| Trigger | Runs | Use for |
| --- | --- | --- |
| **Startup** | Once, when the agent starts monitoring | Initial device state (BIOS, TPM, OS info) |
| **Interval** | Every 5–3600 seconds | Continuous monitoring (network status, policy changes) |
| **Phase Change** | When enrollment enters a specific ESP phase (`Start`, `DevicePreparation`, `DeviceSetup`, `AppsDevice`, `AccountSetup`, `AppsUser`, `FinalizingSetup`, `Complete`, `Failed`) | State checks at a defined point of enrollment |
| **On Event** | When a specific event type is emitted (e.g. `enrollment_complete`, `enrollment_failed`, `app_install_failed`) | Collecting "at the end" or in reaction to a failure |

## What the data looks like

Each execution emits one event with the rule's configured **output event type** and severity. The collected values live in the event's `data` field — and that's exactly what analyze rules reference via `dataField`:

| Collector | Data fields produced |
| --- | --- |
| Registry | Value names as keys, or `subkey_count` + `subkey_0`, `subkey_1`, … |
| Event Log | Event properties as key-value pairs |
| WMI Query | The WMI object's properties as keys |
| File | `exists`, `path`, `size_bytes`, `content` (if `readContent`) |
| Command | `output` (stdout, max 32 KB), `error_output` (stderr), `exit_code`, `command` — **raw text**, graded with `contains`/`regex` operators |
| Log Parser | Your regex's named capture groups as keys |
| JSON / XML | The matched values as key-value pairs |

See [Cookbook recipe 7](analyze-rules/cookbook.md#recipe-7-collect-your-own-data-and-grade-it-end-to-end) for the full gather → analyze pattern in action.

## Security guardrails

Custom collection on managed devices is a sensitive capability, so every collector enforces **allow-lists on the agent itself** — not just in the portal:

* **Registry** paths must fall under approved `HKLM\`/`HKCU\` prefixes (segment-bounded: subkeys of an allowed prefix are fine, sibling keys are not).
* **File / JSON / XML / Log Parser** paths must be within approved directories.
* **WMI** queries must target approved classes; **commands** must match the allow-list *exactly* — arbitrary commands are impossible.
* A rule targeting anything outside the allow-lists doesn't fail silently: the agent emits a **`security_warning` event** into the timeline instead, so misconfigurations are visible.

The current allow-lists are shown inline on the Gather Rules page in the portal (expandable under each collector type).

{% hint style="warning" %}
**Unrestricted Mode** relaxes the registry/WMI/command guardrails for environments that need broader collection. It is not generally available: the capability has to be **unlocked for your tenant by the platform operators on request** — only then does the toggle appear under Settings → Agent. Hard limits remain even with it enabled: `C:\Users` stays blocked and dangerous operations stay impossible. Leave it off unless a specific rule requires it.
{% endhint %}
