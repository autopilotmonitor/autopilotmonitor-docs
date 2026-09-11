---
type: Troubleshooting
tags: [troubleshooting, install-progress, microsoft-365-apps, office, click-to-run]
description: Why Install Progress can list Microsoft 365 Apps twice — once as the package you deployed, once as the Office Click-to-Run installation itself — and how to read the two rows together.
---

# Microsoft 365 Apps appears twice in Install Progress

You open a session and Install Progress lists Office twice: one row with the name of your app, for example **Microsoft 365 Apps for Enterprise**, and one row called **Microsoft 365 Apps** with a **Click-to-Run** pill. Sometimes both rows are still running, sometimes one is green and the other is not.

Office was not installed twice, and the rows are not duplicates. **They are two different observers of the same installation.**

## The short answer

| Row | What it tells you |
| --- | --- |
| **Your app** — the name you gave the app in Intune, or a package with a **RealmJoin** pill | Did the *deployment package* run, and what did it report back? |
| **Microsoft 365 Apps** with a **Click-to-Run** pill | Did *Office itself* land on the device, and when did it finish? |

The first row tracks the package that started the Office setup. The second row watches the Office installation directly on the device, however that installation was started. Most of the time both agree. When they don't, that difference is useful to you, as described below.

## How Office gets onto a device — and which rows you see

Microsoft 365 Apps is always installed by the same engine, the Office **Click-to-Run** setup. What differs is *who starts it*:

| How you deploy Office | What happens on the device | Rows in Install Progress |
| --- | --- | --- |
| **Intune built-in app type** *Microsoft 365 Apps (Windows 10 and later)* | Intune sends the Office configuration to the device through MDM (the Office CSP), and the device starts the Click-to-Run setup with it. The Intune Management Extension is not involved. | **One row:** Microsoft 365 Apps · Click-to-Run |
| **Win32 app** — the Office Deployment Tool (`setup.exe` plus your `configuration.xml`) packaged as an `.intunewin` | The Intune Management Extension runs your package, and your package starts the Click-to-Run setup. | **Two rows:** your Win32 app **and** Microsoft 365 Apps · Click-to-Run |
| **RealmJoin package** that runs the Office Deployment Tool | The RealmJoin agent runs the package, and the package starts the Click-to-Run setup. | **Two rows:** your package · RealmJoin **and** Microsoft 365 Apps · Click-to-Run |
| **Office already on the device** — OEM image, or consumer Office that ships with Windows | Nothing is installed; Office may run a background update. | **One row:** Microsoft 365 Apps, marked **Preinstalled** (blue) — not an install and not a failure |

With the built-in app type, the Click-to-Run row is the **only** place Office appears. The Intune Management Extension never sees that installation, so there is no Intune row to show.

If the enrollment removes a preinstalled Office and installs your enterprise Office afterwards, the blue **Preinstalled** row turns into a normal install row. It stays one row.

## Why the Click-to-Run row exists

A package reporting *success* and Office being *installed* are not always the same moment. The deployment package's row ends when the package ends: the installer exits, the exit code is evaluated, the detection rule runs. Click-to-Run can keep downloading and laying down Office after that, or it can fail while the package still reports success.

So the agent watches Office directly: the Click-to-Run configuration in the registry, the Office download through Delivery Optimization and the Click-to-Run setup process. It marks the installation **complete only when the core Office apps (Word, Excel, PowerPoint or Outlook) are on disk**. That is why the Click-to-Run row usually starts and ends at different times than your package row. Different durations for the two rows are normal.

Once Office is installed, the `office_install_completed` event in the timeline also shows the Office version and how the download was delivered: from a Connected Cache server, from peers or from the Microsoft CDN.

## Reading the two rows together

| Your package row | Click-to-Run row | What it means |
| --- | --- | --- |
| ✓ Installed | ✓ Installed | Everything worked. The durations differ because the two rows measure different things. |
| ✓ Installed | ✗ Failed | The package reported success, but Office did not finish installing. The analyze rule **ANALYZE-OFFICE-001** flags this on the session. See [Office failed although the package succeeded](#office-failed-although-the-package-succeeded). |
| ✗ Failed | ✓ Installed | Office is on the device, but your package reported a failure. This almost always points at the package: its **detection rule** or the **exit code** of a wrapper script, not at Office. |
| any | **Incomplete** | The session ended before Office finished, so the agent stopped watching. This is neither a success nor a failure; the time shown is how long it was observed. |
| — (no package row) | any | You use the built-in app type, so the Click-to-Run row is the whole story. |

The Click-to-Run row is **observation only**. It never changes the enrollment result by itself. The Enrollment Status Page waits for the app as Intune reports it, and a Click-to-Run failure surfaces as an analyze-rule finding (warning), not as a failed enrollment.

## Office failed although the package succeeded

1. Open the `office_install_failed` event in the timeline. It lists the Office products, the update channel and — when Click-to-Run recorded one — an error code.
2. Check the Click-to-Run logs on the device. The setup runs as SYSTEM, so the logs are in `%windir%\Temp`, named after the computer name and a timestamp.
3. Check whether the device could reach the Office CDN or your Connected Cache during the enrollment.
4. Check the timeline for a reboot while Office was still installing. Click-to-Run runs in the background and does not stop the Enrollment Status Page from rebooting the device.

## Tip: make the rows easy to tell apart

The Click-to-Run row is always called **Microsoft 365 Apps**. Give your deployment package a name that differs from it, for example *M365 Apps – Win32 (Semi-Annual)*. Then it is obvious at a glance which row is your package and which one is Office.

## Still unsure?

Use **Report Session** on the session and describe how you deploy Office — built-in app type, Win32 package or RealmJoin. With that context, the two rows are quick to explain.
