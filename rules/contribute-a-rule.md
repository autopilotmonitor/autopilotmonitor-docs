---
type: Guide
tags: [rules, community, contribution]
description: >-
  Submit your own gather or analyze rules for the community rule pool from the
  portal, choose how you are credited, and follow the review to publication.
---

# Contribute a Rule

Custom rules are yours alone. When one of them would help other organizations, you can submit it for the **community rule pool** straight from the portal. Accepted rules ship to every tenant as *Community* rules — with the credit you chose.

## Submitting

1. Open **Analyze Rules** or **Gather Rules** and click **Contribute a rule** in the community box above the rule list.
2. Pick one or more of your custom rules (up to 10 per submission). Built-in and community rules cannot be submitted.
3. Add a comment: what the rule detects and why it is useful beyond your organization. The reviewer reads it first. Optionally leave a contact email for questions about the submission — a sign-in account is not always a mailbox. The address is never published.
4. Choose the **credit** for the published rule:
   * **Community contribution** (default) — anonymous. Nothing about you or your organization is published.
   * **My organization** — the company name from your tenant settings.
   * **A name of my choice** — free text, for example your name.
   The credit becomes the public `author` field of the rule and is visible to every tenant.
5. Confirm the licence statement and submit.

Each rule receives a **submission id** (12 characters). Keep it if you want to ask about the submission.

The rule is frozen at submit time: later edits or deletions of your copy do not change what was submitted. Your own rule stays untouched — you can keep using it as before.

## What happens next

The submission appears in a **Community submissions** list on the same page. Pending submissions are shown by default; decided and withdrawn ones sit behind **Show all**. Withdrawn entries disappear after 30 days, declined ones after 90 days; approved and published entries stay as the record of where a community rule came from.

Statuses:

| Status | Meaning |
| --- | --- |
| Pending review | Waiting for the Autopilot Monitor team. You can withdraw it while it is pending. |
| Approved | Accepted. The reviewer's note and the future rule id are shown. The rule may be adapted before it ships — the note says so. |
| Published | The community rule is live for every tenant under the shown id. You can delete your custom copy if you no longer need it. |
| Declined | Not accepted, with the reviewer's reason. You can improve the rule and submit it again. |
| Withdrawn | Taken back by you before a decision. |

Tenant admins also receive an in-app notification when a submission is decided.

## What the review looks at

* Does the rule pass the same pre-flight checks the portal and the [AI-assisted authoring tools](ai-assisted-rule-authoring.md) apply — schema, collector guardrails, evaluable conditions?
* Does it fire where it should and stay quiet where it should not? Analyze rules are dry-run against real sessions; fire statistics from your tenant are part of the picture.
* Is it useful beyond one organization? Tenant-specific values are usually turned into a [template rule](analyze-rules/template-rules.md) so other admins can fill in their own.
* Does it overlap with an existing built-in rule?

Rules that are too specific to one environment, duplicate a built-in rule, or cannot be evaluated safely are declined with a reason.

## Good candidates

* A detection for a failure you hit more than once and that took real effort to diagnose.
* A gather rule that surfaces evidence the built-in collectors do not have, with an allow-listed target that exists on any device.
* Anything that turned a "failed" status into an answer for you.

Bugs, ideas and questions still go to the [GitHub issues](https://github.com/okieselbach/AutopilotMonitor/issues). IME log patterns follow their own path — see [IME Log Patterns](ime-log-patterns.md#contributing-patterns).
