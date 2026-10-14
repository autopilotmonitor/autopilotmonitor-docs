---
type: Concept
tags: [sessions, limits, blocked, agent]
description: >-
  The limits that keep one misbehaving device from distorting your fleet data
  or overloading the platform: the upload rate limit, the event limit that
  pauses or ends a session's monitoring, and what you see when one applies.
---

# Protective Limits

Autopilot Monitor watches thousands of enrollments at once. A single device that sends far more data than an enrollment needs — a network connection that drops and reconnects every few minutes, a download that reports progress without pause — would distort your fleet statistics and take capacity away from every other enrollment. A few protective limits contain such a device by deciding how much of its telemetry Autopilot Monitor accepts.

## Which limits exist

| Limit | What it covers | What happens |
| --- | --- | --- |
| **Upload rate** | How many uploads a device may send per minute | Uploads above the limit are rejected until the window clears; the agent sends them again. See [Timeline gaps after adding gather rules](../troubleshooting/common-problems.md#timeline-gaps-after-adding-gather-rules-ingestion-rate-limit). |
| **Event limit** | How many events one enrollment session may send in total | The platform **pauses** the session's uploads for a limited time, or **ends monitoring** on the device. |
| **Agent lifetime** | How long the agent stays on the device | The agent stops after 6 hours of active time, and removes itself at its next start once its enrollment phase is older than 48 hours. See [Sessions & Statuses → Timeouts](sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions). |

The event limit sits far above what an enrollment normally produces — a typical session sends a few hundred events. It is set by the platform and applies to every tenant.

## When a session crosses the event limit

The platform checks the event volume of running sessions regularly. A session above the limit gets one of two measures:

* **Pause** — the session's uploads are refused for a limited time. The enrollment itself is not affected: the agent keeps running and holds its further events back on the device; if it is still running when the pause ends, it resumes uploading. The pause covers only this enrollment session: the device's next enrollment starts unblocked.
* **End monitoring** — the stronger measure. The agent stops monitoring and removes itself. Until the end time shown on the session page, the device stays unmonitored, including new enrollments on it.

The platform operator can apply the same measures by hand. The session page then says the platform operator paused or ended monitoring.

## What you see

* **Session Info** shows a card under the session's details: what happened, when, until when, and — for the event limit — how many events the session sent against which limit.
* **The session list** marks the session with a **Blocked** pill; its tooltip names the cause and the time.
* **The event timeline** carries a *device_blocked* event at the moment of the pause, so it is clear why the timeline stops there.
* **Analysis** shows the finding *Monitoring Paused by a Protective Limit*. Like every analyze rule, it can send a message to your notification channels: turn on **Channel notification** on its card in [Analyze Rules](../rules/analyze-rules/README.md#managing-rules-in-the-portal).
* **The audit log** records the block.

While the pause lasts, no events from the session reach Autopilot Monitor. Once the session has been silent long enough, it is classified like any silent session (see [Timeouts](sessions-and-statuses.md#timeouts-what-happens-to-stuck-sessions)), usually as *Incomplete*, which does not count as a failure. Its status reason names the pause instead of a silent agent.

## What to do

1. Open the event timeline and look at which event type repeats most often before the pause. That repetition is what the device was doing.
2. Fix the underlying condition on the device or in your network, for example a loose dock, a flaky cable or an unstable Wi-Fi connection.
3. Re-enroll the device if you need a complete record of the enrollment. After a pause, the new enrollment is monitored straight away; after monitoring was ended, wait for the end time shown on the session page.
4. If no event type stands out, report the session from its page so it can be looked at.
