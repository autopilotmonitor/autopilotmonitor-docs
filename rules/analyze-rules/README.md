---
description: >-
  Automated analysis of every enrollment session — findings with severity,
  confidence, explanation, and remediation, produced by rules you can extend.
---

# Analyze Rules

Analyze rules are the analysis engine of Autopilot Monitor. Every enrollment session is automatically evaluated against all enabled rules — no one has to watch the dashboard for problems to surface.

A rule that matches produces a **finding** on the session:

* a **severity** (info / warning / high / critical) and a **confidence score**,
* an **explanation** of what was detected, with the actual evidence from this session interpolated in (app names, error codes, durations),
* **remediation steps** that tell you what to do about it,
* optionally, links to related documentation.

Findings appear as cards on the session detail page and feed the analysis summary, fleet statistics, and per-rule telemetry (how often each rule fires across your enrollments).

## In this section

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Concepts</strong></td><td>The anatomy of a rule: conditions, sources, operators, preconditions, and confidence scoring.</td><td><a href="concepts.md">concepts.md</a></td></tr><tr><td><strong>Built-in Rules Reference</strong></td><td>The catalog of all built-in rules — what each detects and when it fires.</td><td><a href="built-in-rules.md">built-in-rules.md</a></td></tr><tr><td><strong>Template Rules</strong></td><td>Configurable blueprints: fill in your environment-specific values and get a tailored rule.</td><td><a href="template-rules.md">template-rules.md</a></td></tr><tr><td><strong>Cookbook</strong></td><td>Seven worked recipes for building your own rules, from five-minute template to end-to-end custom detection.</td><td><a href="cookbook.md">cookbook.md</a></td></tr></tbody></table>

## Managing rules in the portal

The **Analyze Rules** page (requires the Admin or Operator role; editing requires Admin) shows all rules with search, severity/category filters, and type filters (Built-in / Community / Custom). Each rule card offers:

* **Enable/disable** per tenant — your choice is preserved across product updates.
* **Mark session as failed** — a per-rule toggle that makes a firing rule fail the whole session (a "knock-out criterion"). A few built-ins ship with this on by default; you can override it.
* **Fire statistics** — how often the rule fired and its hit rate over the last 30 days, so you can see which rules actually earn their keep in your environment.
* **Edit / Export** — custom rules are fully editable in a form or as raw JSON (the **Form ⇄ JSON** toggle); built-ins are read-only but can be exported as JSON to use as a starting point for your own.
