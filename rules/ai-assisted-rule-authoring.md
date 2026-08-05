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

Three building blocks are exposed through the MCP server:

* **Authoring resources** — the complete rule schemas, the gather-rule collection allowlists, and a full authoring guide (condition types, operators, confidence scoring, placeholder interpolation). The assistant reads the real contract instead of guessing.
* **`validate_rule`** — checks a draft rule without deploying anything: schema validation, allowlist checks for gather targets, and lint for rules that would be valid but could never fire (for example an unreachable confidence threshold).
* **`test_analyze_rule`** — the key step: **dry-runs a draft analyze rule against a real session from your tenant**. You get a full trace — which conditions matched (with the actual evidence from the session), how many events of each referenced type the session even contains, how the confidence score builds up, and whether the rule would have fired. Nothing is saved; sessions and analysis results are untouched.

## The workflow

1. **Describe the scenario** to your assistant, e.g.:

   > *"Create an analyze rule that flags enrollments where the same app failed to install three or more times."*

   Many MCP clients also expose the ready-made **author-rule** prompt, which drives the steps below automatically.

2. The assistant drafts the rule and **validates** it (`validate_rule`), fixing schema or allowlist problems.

3. **Test against real sessions.** Give the assistant a session from your tenant — ideally one where the rule *should* fire and one where it should stay silent:

   > *"Test this rule against session 3f2a… — should it fire there?"*

   The dry-run trace shows exactly why the rule fired or didn't, and previews the explanation text with real values filled in. Iterate until both directions behave as expected.

4. **Create the rule** in the portal (**Settings → Analyze Rules** or **Gather Rules**), pasting the finished JSON — or export/import via the rule pages. Dry-running requires the Admin role (or platform read roles); creating rules requires Admin.

## Gather rules

Gather rules execute on enrolling devices, so there is no server-side dry-run for them. The assistant instead validates the target against the [collection allowlists](gather-rules.md) up front, and after deploying you can verify collection on a test device (the agent's gather-rule debug log setting shows every evaluation decision). A common pattern is a pair: a gather rule collects a new signal, and an analyze rule — which *can* be dry-run — turns it into a finding.

## Tips

* Dry-run against **recent** sessions: sessions that ran before a gather rule was deployed don't contain its events.
* If a condition unexpectedly doesn't match, check the trace's per-condition event count first — it tells you whether the session contains the referenced event type at all.
* Well-tested rules that would help other organizations are welcome as [community contributions](analyze-rules/built-in-rules.md) via the [GitHub repository](https://github.com/okieselbach/Autopilot-Monitor/tree/main/rules).
