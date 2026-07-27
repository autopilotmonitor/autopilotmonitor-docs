---
type: Overview
tags: [rules, analyze]
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
* **Fire statistics** — how often the rule fired and its hit rate over the last 30 days, plus a 30-day sparkline of daily fires, so you can see which rules actually earn their keep in your environment.
* **Regression badge** — a red **↑ Regression** badge appears while a rule is firing markedly more often than it used to (see below).
* **Edit / Export** — custom rules are fully editable in a form or as raw JSON (the **Form ⇄ JSON** toggle); built-ins are read-only but can be exported as JSON to use as a starting point for your own.

## Regression detection

A rule that suddenly starts firing far more often than it used to is a signal in itself: something changed in your environment — a Windows build, a driver, an app version, a policy — and it changed for the worse. Autopilot Monitor watches for that automatically, once a day, per rule and per tenant.

A rule is flagged when **all** of the following hold:

* its hit rate over the **last 7 days** is at least **twice** its rate over the **preceding 28 days**,
* the increase is **statistically separated** — the confidence intervals of the two rates do not overlap, so a quiet week followed by a busy one is not enough, and
* the window carries real volume: at least **5 sessions with a hit** and **20 evaluated sessions**.

Rules younger than about a month, rules you edited inside the window, and rules without a baseline are deliberately skipped — a new or changed rule has no "usual" to compare against.

When a rule is flagged you get a **bell notification** that links to the rule card, and the card shows the **↑ Regression** badge for as long as the episode lasts. Both carry the actual numbers (*"18 % of evaluated sessions in the last 7 days vs 4 % baseline"*), and where the affected sessions concentrate on one **OS build, device model, agent version, or IME version**, that share is reported too — as a correlation to investigate, not a cause. The episode ends when the rate falls back to normal, and only then can the same rule alert again, so an ongoing problem never turns into a stream of notifications.

Regression detection covers **analyze rules**; [gather rules](../gather-rules.md) collect rather than judge and are not part of it.
