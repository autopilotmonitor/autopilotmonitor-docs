---
description: >-
  A guided walkthrough — enroll a test device and watch Autopilot Monitor at
  work, from the first dashboard entry to the finished session.
---

# Your First Monitored Enrollment

With [Portal Setup](portal-setup.md) done and the [agent deployed](deploy-the-agent.md), it's time to see everything in action with a test device.

## 1. Start a test enrollment

Use any Autopilot-registered test device (a fresh VM registered for Autopilot works too, but note that some hardware-specific checks like Secure Boot analysis are skipped on VMs):

1. Reset the device or start it factory-fresh so it boots into OOBE.
2. Proceed through OOBE until the Autopilot enrollment starts and the **Enrollment Status Page (ESP)** appears.
3. Intune delivers the platform script during the device-preparation phase; the bootstrapper installs the agent seconds later.

## 2. Watch the session appear

Open the **Dashboard** in the portal. Within seconds of the agent starting, a new session appears with status **In Progress**, showing the device serial number, model, and the current enrollment activity — while the device is still at the ESP.

<!-- SCREENSHOT: dashboard-live-session — Dashboard session list with one freshly started In-Progress session (highlight the live status badge). Genuinely valuable here: this is the "it works!" moment for a new user. -->

{% hint style="info" %}
**No session appearing?** Check in this order: (1) Is **Autopilot Device Validation** enabled in Settings? Without it, all agent uploads are rejected. (2) Is the platform script assigned to the device's group? (3) Did a bootstrap guard skip the device — e.g. was it already enrolled once (registry marker), or has it been powered on for more than 12 hours? Run the [dry-run tester](deploy-the-agent.md#try-it-first-dry-run) on the device to see the exact decision.
{% endhint %}

## 3. Follow the live timeline

Click the session to open the **Session Detail** view:

* The **timeline** fills in real time: ESP phase transitions, every app download and installation with its outcome, PowerShell scripts, policies, and system events.
* Thanks to the agent's IME log **backfill**, the timeline starts at the very beginning of the enrollment — including everything that happened before the agent was installed.
* **Device metadata** (hardware, OS build, network, security posture) appears alongside the timeline.

This is the view you'll use later to answer "why is this device stuck at the ESP?" — for a deeper tour see [Session Details & Diagnosis](../portal-guide/session-details-and-diagnosis.md).

## 4. Read your first rule findings

While events stream in, [analyze rules](../rules/overview.md) evaluate the session automatically. Findings appear on the session as cards with a severity, a confidence score, an explanation of what was detected, and concrete remediation steps.

Even a perfectly healthy enrollment often produces a finding or two — for example a security posture note. That's the system working: every session gets the same automated review, so problems surface without anyone watching the dashboard.

## 5. Completion and cleanup

When the enrollment completes, the session status flips to **Succeeded** (or **Pending** first, in pre-provisioning scenarios — see [Sessions & Statuses](../concepts/sessions-and-statuses.md)). On the device, the agent performs its final upload and then **removes itself** — files and scheduled task are gone; only the small `Deployed` registry marker remains.

## What's next

* Understand the concepts behind what you just saw: [Sessions & Statuses](../concepts/sessions-and-statuses.md) and [Agent Lifecycle & Security](../concepts/agent-lifecycle-and-security.md)
* Explore the portal views: [Portal Guide](../portal-guide/dashboard-and-sessions.md)
* Set up [notifications](../integrations/notifications.md) so enrollment failures reach your team automatically
