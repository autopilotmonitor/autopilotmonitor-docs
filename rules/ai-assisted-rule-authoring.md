---
type: Guide
tags: [rules, ai, mcp, analyze-rules, gather-rules]
description: >-
  Let an AI assistant build gather and analyze rules for you — with built-in
  validation and a dry-run against real sessions from your tenant before
  anything goes live.
---

# AI-Assisted Rule Authoring

Building a [custom rule](overview.md) by hand means knowing the rule schema, the condition operators, the confidence model, and (for gather rules) the collection allowlists. With [AI Integration (MCP)](../integrations/ai-integration-mcp.md) connected, you can skip all of that: describe what you want to detect in plain language, and your AI assistant has everything it needs to produce a working, **tested** rule.

## What the assistant can do

Four building blocks are exposed through the MCP server:

* **Authoring resources** — the complete rule schemas, the gather-rule collection allowlists, and a full authoring guide (condition types, operators, confidence scoring, placeholder interpolation). The assistant reads the real contract instead of guessing.
* **`validate_rule`** — checks a draft rule without deploying anything: schema validation, allowlist checks for gather targets, and lint for rules that would be valid but could never fire (for example an unreachable confidence threshold).
* **`test_analyze_rule`** — the key step: **dry-runs a draft analyze rule against a real session from your tenant**. You get a full trace — which conditions matched (with the actual evidence from the session), how many events of each referenced type the session even contains, how the confidence score builds up, and whether the rule would have fired. Nothing is saved; sessions and analysis results are untouched.
* **`test_log_pattern`** — for log-parsing gather rules: tests a regex against sample lines from your log file **with the exact engine the agent uses on the device** (.NET, case-sensitive, CMTrace or plain-text mode). A pattern that works in an online regex tester can behave differently on the device — this tool removes that guesswork.

## The workflow

1. **Describe the scenario** to your assistant, e.g.:

   > *"Create an analyze rule that flags enrollments where the same app failed to install three or more times."*

   Many MCP clients also expose the ready-made **author-rule** prompt, which drives the steps below automatically.

2. The assistant drafts the rule and **validates** it (`validate_rule`), fixing schema or allowlist problems.

3. **Test against real sessions.** Give the assistant a session from your tenant — ideally one where the rule *should* fire and one where it should stay silent:

   > *"Test this rule against session 3f2a… — should it fire there?"*

   The dry-run trace shows exactly why the rule fired or didn't, and previews the explanation text with real values filled in. Iterate until both directions behave as expected.

4. **Create the rule** in the portal (**Settings → Analyze Rules** or **Gather Rules**), pasting the finished JSON — or export/import via the rule pages. Dry-running requires the Admin role (or platform read roles); creating rules requires Admin.

## Example: from a log file to timeline events

The most powerful pattern: hand the assistant a log file and let it build a **log-parsing gather rule** whose extracted values show up as events in the enrollment timeline.

1. **Attach the log file** (or paste representative lines) and describe what you want:

   > *"Here is our installer's log. Create a rule that shows every ERROR line with its component and error code in the enrollment timeline."*

2. The assistant reads the authoring guide and drafts a `logparser` gather rule. Named capture groups in the regex become data fields on the emitted event:

   ```json
   {
     "ruleId": "GATHER-APPS-101",
     "title": "CustomApp installer errors",
     "collectorType": "logparser",
     "target": "C:\\Windows\\Logs\\CustomApp\\install*.log",
     "parameters": {
       "pattern": "(?<level>ERROR|FATAL)\\s+(?<component>\\w+):\\s+(?<message>.+?)\\s+\\(code\\s+(?<errorCode>0x[0-9A-Fa-f]+)\\)",
       "format": "text",
       "trackPosition": "true"
     },
     "trigger": "interval",
     "intervalSeconds": 120,
     "outputEventType": "gather_customapp_errors",
     "outputSeverity": "Warning"
   }
   ```

3. `validate_rule` confirms the log path is on the collection allowlist and the rule is well-formed.

4. **`test_log_pattern` verifies the regex with the device's real engine.** The assistant sends the pattern plus sample lines from your file and gets back a per-line result: which lines matched, which didn't, and exactly which values (`level`, `component`, `errorCode`, …) each timeline event would carry. Two things it catches that generic regex testers miss:

   * Matching on the device is **case-sensitive** — `ERROR` does not match `error`.
   * IME/SCCM-style logs use the **CMTrace** format: the agent parses each line first and matches your regex against the message only. For plain-text logs the rule needs `"format": "text"` — the tool tells you when every line fails CMTrace parsing.

5. After deploying, each matching log line appears as a `gather_customapp_errors` event in the timeline. On top of it, the assistant can add an analyze rule (for example *"flag the session when any FATAL line appears"*) — and that one **can** be dry-run against real sessions with `test_analyze_rule`.

## Gather rules

Gather rules execute on enrolling devices, so there is no session dry-run for them. For log-parsing rules, `test_log_pattern` (above) covers the part that actually goes wrong — the regex. For all gather rules the assistant validates the target against the [collection allowlists](gather-rules.md) up front, and after deploying you can verify collection on a test device (the agent's gather-rule debug log setting shows every evaluation decision). A common pattern is a pair: a gather rule collects a new signal, and an analyze rule — which *can* be dry-run — turns it into a finding.

## Tips

* Dry-run against **recent** sessions: sessions that ran before a gather rule was deployed don't contain its events.
* If a condition unexpectedly doesn't match, check the trace's per-condition event count first — it tells you whether the session contains the referenced event type at all.
* Well-tested rules that would help other organizations are welcome as [community contributions](analyze-rules/built-in-rules.md) via the [GitHub repository](https://github.com/okieselbach/Autopilot-Monitor/tree/main/rules).
