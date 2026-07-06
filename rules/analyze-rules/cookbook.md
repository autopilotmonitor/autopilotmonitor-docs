---
description: >-
  Seven worked recipes for building your own analyze rules — from a five-minute
  template to a full custom detection pipeline with your own collected data.
---

# Cookbook: Build Your Own Rules

Each recipe follows the same shape: **the goal → the rule → why it's built that way**. Recipes are ordered by difficulty; later ones assume you've read [Concepts](concepts.md). Create custom rules on the **Analyze Rules** page via **Create Custom Rule** — simple rules work in form mode, recipes using correlation or arrays need the **JSON mode** toggle.

{% hint style="info" %}
**Testing tip:** you don't need to wait for a live enrollment to test a rule. Replay a captured IME log with the agent's `--replay-log-dir` parameter (see [Agent Command-Line Parameters](../../reference/agent-command-line.md)) to create a real session in fast-forward, then check whether your rule fires on it.
{% endhint %}

## Recipe 1: Your first rule in five minutes (template)

**Goal:** warn when the SCEP certificate your Wi-Fi depends on is missing after enrollment.

No JSON needed — this is a [template rule](template-rules.md):

1. Make sure the gather rule **GATHER-ID-002** (machine certificate store) is enabled on the Gather Rules page.
2. On the Analyze Rules page, open the **Templates** tab and enable **ANALYZE-ID-001**.
3. Enter your certificate subject, e.g. `CN=Contoso Issuing CA` (case-insensitive substring match), and save.

That's it: a custom copy `ANALYZE-ID-001-CUSTOM` now checks every enrollment and warns when the certificate didn't arrive — usually the first sign of a broken SCEP profile or NDES connector.

## Recipe 2: Alert when a specific critical app fails

**Goal:** your VPN client is business-critical — if *it* fails, you want a high-severity finding immediately, not a generic app-failure card.

```json
{
  "ruleId": "ANALYZE-APP-101",
  "title": "Contoso VPN Client failed to install",
  "description": "The business-critical VPN client failed during enrollment.",
  "severity": "high",
  "category": "apps",
  "enabled": true,
  "baseConfidence": 90,
  "conditions": [
    {
      "signal": "vpn_client_failed",
      "source": "event_type",
      "eventType": "app_install_failed",
      "dataField": "appName",
      "operator": "contains",
      "value": "Contoso VPN",
      "required": true,
      "suppressByEvent": {
        "eventType": "app_install_completed",
        "joinField": "appId"
      }
    }
  ],
  "confidenceThreshold": 40,
  "explanation": "The **Contoso VPN Client** failed to install during this enrollment. Without it the user cannot reach internal resources.\n\n- **App:** `{{appName}}`",
  "remediation": [
    {
      "title": "Check the VPN client installation",
      "steps": [
        "Open the app_install_failed event for the exact error code",
        "Verify the install succeeded on retry (this rule suppresses itself if it did)",
        "If the failure is systemic, check the generic app-failure findings on this session"
      ]
    }
  ],
  "tags": ["apps", "vpn", "business-critical"]
}
```

**Why it's built this way:**

* `contains` on `appName` survives display-name suffixes like version numbers; use `equals` only if your app names are perfectly stable.
* `suppressByEvent` keeps the rule quiet when a later retry of the **same app** (joined on `appId`) succeeded — you only hear about failures that *stayed* failures.
* `{{appName}}` interpolates the actual matched app name into the finding.
* Want a failure of this app to fail the whole session? Add `"markSessionAsFailedDefault": true`.

## Recipe 3: Thresholds — "this is taking too long"

**Goal:** flag enrollments where any single app takes more than 15 minutes, and separately, where the AccountSetup phase drags beyond 20 minutes.

**Slow app** — the `app_install_duration` source measures start → completion per app:

```json
{
  "signal": "very_slow_app",
  "source": "app_install_duration",
  "operator": "gt",
  "value": "900",
  "required": true
}
```

**Slow phase** — `phase_duration` measures how long the session spent in an ESP phase. The idiomatic pattern (borrowed from the built-in ESP-001) gates firing through a confidence factor:

```json
{
  "conditions": [
    {
      "signal": "account_setup_phase",
      "source": "phase_duration",
      "eventType": "esp_phase_changed",
      "dataField": "espPhase",
      "operator": "equals",
      "value": "AccountSetup",
      "required": true
    }
  ],
  "baseConfidence": 50,
  "confidenceFactors": [
    { "signal": "long_phase", "condition": "phase_duration > 1200", "weight": 40 }
  ],
  "confidenceThreshold": 70
}
```

**Why it's built this way:** the condition alone matches whenever the phase *exists* (base 50 — below the threshold of 70). Only when the factor `phase_duration > 1200` also matches does the score reach 90 and the rule fire. This base-below-threshold trick is *the* pattern for "X exists AND X is extreme".

## Recipe 4: Counting — repeats and sustained states

**Goal 1:** the same app failed twice or more (per-app counting):

```json
{
  "signal": "same_app_failed_twice",
  "source": "event_count",
  "eventType": "app_install_failed",
  "dataField": "appId",
  "operator": "count_per_group_gte",
  "value": "2",
  "required": true
}
```

`dataField` is the **grouping key** — the condition matches when any single group (here: any one app) reaches the count.

**Goal 2:** disk space stayed low — at least 3 performance snapshots under 10 GB free. A single low reading during a large install can be normal; a *sustained* state is a finding. This uses the count **value filter**:

```json
{
  "signal": "sustained_low_disk",
  "source": "event_count",
  "eventType": "performance_snapshot",
  "operator": "count_gte",
  "value": "3",
  "filterField": "disk_free_gb",
  "filterOperator": "lt",
  "filterValue": "10",
  "required": true
}
```

**Why it's built this way:** `filterField`/`filterOperator`/`filterValue` restrict *which* events are counted (only snapshots below 10 GB), while `operator`/`value` set the count threshold (at least 3 of them). The built-in ANALYZE-DEV-002 (sustained high memory) uses exactly this construction — export it for a complete working example.

## Recipe 5: Allow-lists over arrays

**Goal:** alert when something appears on the device that is *not* on your approved list — the pattern behind the provisioning-package rules, transferable to any array payload.

```json
{
  "signal": "ppkg_not_allowlisted",
  "source": "event_data_array",
  "eventType": "provisioning_package_scan",
  "dataField": "artifacts",
  "itemField": "identity",
  "operator": "not_regex",
  "value": "^(?:Microsoft\\.Windows\\.Cosa\\b|Contoso\\.BulkEnroll\\b|Contoso\\.Recovery\\b)",
  "required": true
}
```

**How it reads:** iterate the `artifacts` array; for each element take its `identity` field; the condition matches if **any** element fails the allow-list regex. The finding's evidence lists every non-matching item.

**Why it's built this way:**

* The regex is **anchored** (`^`) with per-alternative word boundaries (`\b`) — an unanchored substring list would let `Evil.Contoso.BulkEnroll.Backdoor` pass because it *contains* an allowed name.
* Extend the list by inserting alternatives **inside** the group, before the closing `)`.
* For provisioning packages specifically, don't build this from scratch — use the [ANALYZE-SEC-006 template](template-rules.md#analyze-sec-006-unexpected-provisioning-package-custom-allow-list), which ships the maintained OS/OEM defaults.

## Recipe 6: Correlation — "A happened, then B, for the same thing"

**Goal:** an app reported *completed*, but later reported *failed* — the same app flipped back after success (the built-in APP-010 scenario, worth studying as the reference).

```json
{
  "signal": "reverted_after_success",
  "source": "event_correlation",
  "eventType": "app_install_completed",
  "correlateEventType": "app_install_failed",
  "joinField": "appId",
  "timeWindowSeconds": 3600,
  "required": true
}
```

**How it reads:** find an `app_install_completed` (event A) followed by an `app_install_failed` (event B) **for the same `appId`**, no more than an hour apart.

**Refinements:**

* Filter which A events qualify with `eventAFilterField` / `eventAFilterOperator` / `eventAFilterValue` — e.g. only completions of a particular app.
* Add `suppressByEvent` to discard pairs that a third, resolving event fixed.
* Correlation conditions require **JSON mode** in the editor.

## Recipe 7: Collect your own data and grade it (end-to-end)

**Goal:** the most powerful pattern in the product: gather evidence the agent doesn't collect by default, then let an analyze rule turn it into an automatic verdict. Example: verify the TPM is ready at the end of every enrollment.

**Step 1 — the Gather Rule** (Gather Rules page → create rule):

| Field | Value |
| --- | --- |
| Collector Type | **Command (Allowlisted)** |
| Target | `Get-Tpm` (must match the allow-list exactly) |
| Trigger | **On Event** → `enrollment_complete` |
| Output Event Type | `gather_tpm_status` |

The command's stdout lands in the event's `output` field (commands always produce `output`, `error_output`, `exit_code` — see [Gather Rules](../gather-rules.md#what-the-data-looks-like)).

**Step 2 — the Analyze Rule** that grades the output:

```json
{
  "ruleId": "ANALYZE-DEV-101",
  "title": "TPM not ready after enrollment",
  "description": "Get-Tpm output shows the TPM is not ready — BitLocker and Windows Hello provisioning will fail on this device.",
  "severity": "high",
  "category": "device",
  "enabled": true,
  "baseConfidence": 85,
  "conditions": [
    {
      "signal": "tpm_status_gathered",
      "source": "event_type",
      "eventType": "gather_tpm_status",
      "operator": "exists",
      "value": "",
      "required": true
    },
    {
      "signal": "tpm_not_ready",
      "source": "event_data",
      "eventType": "gather_tpm_status",
      "dataField": "output",
      "operator": "not_regex",
      "value": "TpmReady\\s*:\\s*True",
      "required": true
    }
  ],
  "confidenceThreshold": 40,
  "explanation": "The TPM on this device reports **not ready** at the end of enrollment. BitLocker escrow and Windows Hello for Business depend on a ready TPM.",
  "remediation": [
    {
      "title": "Investigate the TPM state",
      "steps": [
        "Open the gather_tpm_status event and read the full Get-Tpm output",
        "Check for TPM firmware issues or a cleared/disabled TPM in UEFI",
        "Verify the device model's TPM against your hardware baseline"
      ]
    }
  ],
  "tags": ["device", "tpm", "gather"]
}
```

**Why it's built this way:**

* The first condition (`exists`) makes the rule fire **only when the gather rule actually ran** — without it, `not_regex` on a missing event would never match anyway, but the explicit gate documents the dependency and keeps the evidence clean.
* Command output is raw text, so text operators (`contains`, `not_contains`, `regex`, `not_regex`) on `dataField: "output"` are the right tools. The regex tolerates PowerShell's column whitespace.
* This same two-step shape works for anything: a registry value your image writes, a vendor agent's log line, a WMI property — collect it with the matching [collector type](../gather-rules.md#collector-types), grade it with an analyze rule.

{% hint style="info" %}
Gather rule targets are validated against **security allow-lists** on the agent. If your target isn't covered, the agent emits a `security_warning` event instead of collecting — see [Gather Rules → Security guardrails](../gather-rules.md#security-guardrails).
{% endhint %}

## General tuning advice

* **Start specific.** A rule that fires on half your sessions is noise, not insight — check the fire statistics on the rule card after a few days and tighten conditions or thresholds.
* **Prefer suppression over silence.** If a state can self-heal (retries!), use `suppressByEvent` rather than lowering severity.
* **Steal from the built-ins.** Every built-in rule can be exported as JSON. ANALYZE-APP-013 (preconditions + evidence extraction), ANALYZE-ESP-004 (interpolation done well), and ANALYZE-ID-002 (working confidence factors) are the best ones to learn from.
