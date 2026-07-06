---
description: >-
  How regex patterns turn Intune Management Extension log lines into structured
  timeline events — and how to help maintain them.
---

# IME Log Patterns

The Intune Management Extension (IME) log is the richest source of enrollment truth on a device — but it's a free-form text file. **IME Log Patterns** are regular expressions the agent applies to every log line in real time: when a pattern matches, **named capture groups** extract the data and an **action handler** turns it into a structured timeline event — app download progress, install state changes, ESP phase transitions, error detections.

This is the machinery behind most of what you see on a session timeline. Around 80 maintained patterns ship with the product; you normally never touch them — until Microsoft changes a log format.

## How a pattern works

1. **Match** — each log line is tested against the active patterns whose *category* applies to the current enrollment phase.
2. **Extract** — named capture groups (`(?<appId>…)`) pull values out of the line. The `{GUID}` placeholder expands to a standard GUID regex, so patterns matching app IDs stay readable.
3. **Act** — the pattern's action handler (e.g. `updateStateDownloading`, `espPhaseDetected`, `enrollmentCompleted`) processes the values and emits the structured event.

**Example** — tracking download progress:

```
Pattern:  \[StatusService\] Downloading app \(id = {GUID}.*?\) via (?<tech>\w+), bytes (?<bytes>\w+)/(?<ofbytes>\w+) for user
Action:   updateStateDownloading
```

Every matching line updates the app's state to *downloading* with live progress — including whether content comes via Delivery Optimization or CDN.

## Pattern anatomy

| Field | Purpose |
| --- | --- |
| `patternId` | Unique ID, e.g. `IME-DOWNLOADING` |
| `category` | When the pattern is evaluated: `always` (every line), `currentPhase` (only during the active ESP phase), `otherPhases` (only for non-active phases — used to filter apps already completed earlier) |
| `pattern` | The regex (.NET syntax) with named capture groups |
| `action` | The agent-side handler that processes the match |
| `parameters` | Optional extra configuration for the handler |
| `enabled` | Disabled patterns are skipped |

The **IME Log Patterns** page in the portal lists every pattern with its regex, action, category, and description — use **View as JSON** for the full definition.

## Contributing patterns

Microsoft occasionally changes IME log formats, and a pattern silently stops matching. The patterns are community-maintained on GitHub — here's the workflow when you suspect a break:

1. Enable the **IME Pattern Match Log** in Settings. The agent then writes every matched line to `%ProgramData%\AutopilotMonitor\Logs\ime_pattern_matches.log` on the device — so you can see exactly which patterns fire and which log lines go unmatched.
2. Run an enrollment and compare the match log against the raw IME log to find the changed line format.
3. Find the affected pattern on the IME Log Patterns page and copy its JSON (**View as JSON**).
4. Adjust the regex and submit a **pull request** to the [Autopilot Monitor repository](https://github.com/okieselbach/Autopilot-Monitor) — pattern files live in `rules/ime-log-patterns/`.
5. The pattern is validated against known log samples and merged; the fix reaches all tenants with the next update.

{% hint style="info" %}
This is the most valuable kind of community contribution: one fixed pattern restores telemetry for every organization using the product.
{% endhint %}
