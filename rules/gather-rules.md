---
type: Reference
tags: [rules, gather, diagnostics]
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
| **Registry** | Values of a key (or subkey names with `listSubkeys`) — reads the 64-bit view by default, with `emitOnlyIfExists` to only emit when the key is actually present | `HKLM\SOFTWARE\Microsoft\PolicyManager\current\device` |
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
| **Phase Start** | Once, when enrollment **enters** the selected phase (`Start`, `DevicePreparation`, `DeviceSetup`, `AppsDevice`, `AccountSetup`, `AppsUser`, `FinalizingSetup`, `Complete`) | State check at a defined point — "what did things look like when Account Setup began?" |
| **Phase End** | Once, when enrollment **leaves** the selected phase | Snapshot of a phase's end state — "what did Device Setup leave behind?" |
| **On Event** | When a specific event type is emitted (e.g. `enrollment_complete`, `enrollment_failed`, `app_install_failed`) | Collecting "at the end" or in reaction to a failure |

**Phase Start** and **Phase End** are the two bookends of a phase and both fire **exactly once** per phase — they are the way to build one-shot collections. Leave the phase empty to fire on *every* phase transition.

{% hint style="info" %}
**Picking the right one-shot trigger**

* *Beginning of enrollment* → **Startup**, or **Phase Start** on `Start`.
* *Beginning / end of a specific phase* (e.g. Account Setup) → **Phase Start** / **Phase End** on that phase.
* *End of the enrollment* → **On Event** with `enrollment_complete`, or **Phase Start** on `Complete`. A **Phase End** rule on `Complete` would need enrollment to move *past* Complete, which normally never happens.

**Phase End** also fires when a phase is left because the enrollment **failed** — so a rule that snapshots the end of Device Setup still captures the state at the failure point.
{% endhint %}

{% hint style="warning" %}
**Mind the ingestion rate limit.** Event ingestion is rate-limited **per device at 100 requests per minute** — a budget shared by *everything* the agent sends: its built-in telemetry, IME-derived events, *and* your gather rules. Noisy gather rules — several **Interval** rules firing every 60 seconds, or one rule producing many events per run (log parsers, broad registry reads) — can exhaust that budget. Uploads beyond the limit are rejected until the window clears, so telemetry arrives delayed or gaps appear in the timeline.

Rules of thumb: prefer the one-shot triggers — **Startup**, **Phase Start**, **Phase End**, or **On Event** — over intervals; when you do poll, use the widest interval that still answers your question (an enrollment rarely needs anything sampled more often than every few minutes); and cap what each run returns (`maxEntries`, `maxLines`, `maxResults`). For interval rules, combine [phase scoping](#phase-scoping) and [emit mode](#emit-mode) below — together they eliminate most polling noise.
{% endhint %}

## Phase scoping

{% hint style="info" %}
**Phase scoping and emit mode are for rules that can run more than once** — Interval, On Event, and the "any phase" variants of Phase Start/End. If your trigger already names a phase (e.g. Phase Start on Finalizing Setup), the rule fires exactly once at that moment and the portal hides both controls: a scope could only ever suppress that single collection, and there is no second result to compare against.

The one exception is **Startup**: a scope there doesn't suppress the collection, it *delays* it — the rule waits and collects once the scope is first reached instead of at agent start.
{% endhint %}

By default a rule is active during the **entire** enrollment. **Active During** restricts that — useful when the data only exists (or only matters) from a certain point, e.g. a registry key that software writes during Account Setup:

| Mode | Behavior |
| --- | --- |
| **All phases** (default) | The rule runs whenever its trigger fires — today's behavior. |
| **Only during specific phases** | The rule runs only while enrollment is in one of the selected phases. Outside them, interval timers idle, and phase/event triggers are ignored. |
| **From a phase onwards** | The rule activates when enrollment first reaches the selected phase and **stays active for the rest of the session** — even if a later failure occurs. |

Details worth knowing:

* Scoping applies to **all trigger types**. A scoped **Startup** rule doesn't run at agent start — it runs **once** when its scope first activates.
* Selectable phases are `Start` through `Complete`. Scoping never activates *via* the `Failed` state — but a "from" rule that already activated stays active through it.
* Before the first phase signal of a session, scoped rules are inactive.
* The `--run-gather-rules` diagnostic mode ignores phase scoping (it has no enrollment phase context) and executes scoped rules unconditionally.
* Older agent versions ignore these fields and run the rule in all phases — the benefit arrives with the agent rollout.

## Emit mode

**Emit mode** controls what happens *after* a collection:

| Mode | Behavior |
| --- | --- |
| **Always** | Every collection emits an event — today's behavior, and the mode of all pre-existing rules. |
| **On change** | The rule still collects on its trigger cadence, but only emits an event **when the collected result differs from the last emitted one**. New rules default to this. |

With **On change**, the first collection always emits (on an absent registry key, that first `exists: false` event is your confirmation the rule is polling). Afterwards the timeline stays silent until the value actually changes — the next emitted event carries `suppressedPolls` and `suppressedSinceUtc` in its data, so you can see how many identical polls were skipped and since when.

{% hint style="info" %}
**The zero-noise pattern for "wait until a key appears":** an interval registry rule with `emitOnlyIfExists: true`, phase scoping, and emit mode **On change** produces *no* events while the key is absent, exactly **one** event the moment it appears, and further events only when its values change.
{% endhint %}

Collectors whose output is inherently volatile (event-log entries with timestamps, log parsers with position tracking) change on every poll — for those, **On change** effectively behaves like **Always**.

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
