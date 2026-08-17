---
type: Troubleshooting
tags: [troubleshooting, hybrid-join, enrollment, entra-id, esp, windows-hello]
description: Hybrid Join devices that reach the desktop but never finish — the signed-in user never establishes Entra user affinity (no PRT), so Account Setup re-runs forever and Windows Hello never appears.
---

# Hybrid Join: stuck in Account Setup — the sign-in that never lands

Your Autopilot **Hybrid Entra Join** (HAADJ) device sails through Device Setup, the user signs in and even reaches the desktop — but the session never completes. In the portal the session parks in **Account Setup**, the timeline shows the warning **"Hybrid AAD Join: login overdue (placeholder still active)"** repeating across reboots, and the analysis panel raises **"Hybrid Sign-In Never Establishes Entra User Affinity"**. On the device, the Enrollment Status Page's account phase shows `0/x` apps — and comes back on every logon.

This is not the device failing to join. It is the **user** failing to register.

## How to recognize it

The pattern is distinctive because the device half looks perfectly healthy:

* `dsregcmd /status` on the device reports `AzureAdJoined : YES` **and** `DomainJoined : YES` — the hybrid device registration succeeded.
* The user signs in and gets a desktop; the session's timeline shows *Real user desktop detected*.
* Yet the **hybrid sign-in overdue** warning keeps firing, often across several reboots, and the account-phase apps never move past `0/x`.
* **The Windows Hello for Business enrollment wizard never appears** for the user.

The one number that confirms it: sign in as the affected user, run `dsregcmd /status`, and look under **SSO State**. `AzureAdPrt : NO` is the finding — the signed-in user has no **Primary Refresh Token**, so as far as Entra is concerned, this user session doesn't exist on this device.

## What is actually happening

On a hybrid deployment, joining the device is only half the contract. After the reboot, the user signs in with domain credentials and Windows silently runs a second registration: it exchanges the on-premises logon for an **Entra user token (PRT)**. Everything user-facing hangs off that token — user-targeted apps and policies in the Account Setup ESP, Windows Hello for Business provisioning, single sign-on to Microsoft 365.

When the PRT flow cannot finish, Windows falls back to **cached credentials**: the user gets a desktop, but a desktop without an Entra identity. The Account Setup ESP cannot settle and re-runs on every logon, Hello never offers its wizard (it needs the PRT to provision), and the enrollment session never reaches completion. Rebooting does not fix it — the same sign-in falls into the same hole every time.

The most common structural causes:

* **No domain-controller line of sight at the logon screen.** The PRT flow needs a reachable DC *at sign-in time*. If a VPN is required to reach one, a user-context tunnel that starts *after* sign-in is too late — this is the classic remote-worker / pre-logon VPN gap.
* **The user isn't syncing.** The signing-in user is outside the Entra Connect sync scope, or their UPN suffix isn't a verified domain in Entra ID (`.local` and mismatched suffixes are the usual suspects).
* **The computer object's registration isn't syncing.** The `userCertificate` attribute is filtered out, or the computer OU is outside the sync scope.
* **Network path problems from the user context** — a proxy or firewall blocking `login.microsoftonline.com`, `device.login.microsoftonline.com` or `enterpriseregistration.windows.net`, or significant time skew.

## What to check — in this order

1. **On the device, as the affected user: `dsregcmd /status`.**
   `AzureAdPrt : NO` under *SSO State* confirms the diagnosis. (`AzureAdJoined : YES` / `DomainJoined : YES` under *Device State* just confirms the device half is fine.)

2. **Event Viewer → Applications and Services Logs → Microsoft → Windows → User Device Registration → Admin.**
   Events **304**, **305** and **362** carry the PRT / user-registration failure details and name the blocked endpoint or the failing auth step.

3. **Can the logon screen reach a domain controller?**
   From the network the device enrolls on, verify a DC is reachable *before* any user-context VPN comes up. If a VPN is in the path, it must run as a **machine tunnel / pre-logon tunnel** (device tunnel in Always On VPN, pre-logon in GlobalProtect and comparable products).

4. **Entra Connect scope and UPN.**
   The user's OU and the computer's OU must both be inside the sync scope, the computer object's `userCertificate` attribute must sync, and the user's UPN suffix must be a verified Entra domain.

5. **Proxy/firewall from the user context.**
   `login.microsoftonline.com`, `device.login.microsoftonline.com`, `enterpriseregistration.windows.net` — reachable without interception from the signed-in user's context.

Autopilot Monitor captures the device-side half automatically: the built-in gather rule runs `dsregcmd /status` during Account Setup, and the finding on the session links the observed sign-in warnings to this diagnosis.

## Windows Hello on hybrid devices

Hello deserves its own note, because on these sessions it *looks* broken while actually being downstream:

* **The wizard never appearing is a symptom, not the fault.** Hello provisioning requires the user's PRT; restore user affinity and the wizard appears on the next sign-in.
* If you deploy Hello to hybrid-joined devices, use **cloud Kerberos trust**. It removes the on-premises certificate/key-trust dependencies that historically made Hello the slowest, most fragile step of hybrid enrollments.
* If you don't use Hello, **disable it explicitly** with a device-scoped Intune policy rather than leaving it unconfigured. An explicit "disabled" lets the enrollment resolve the Hello step deterministically instead of waiting for a wizard that will never come.

## Making hybrid enrollments structurally faster

Environments where this pattern never shows share a few habits:

* **Pre-provisioning (White Glove).** The device phase runs on a known-good technician network; the user-facing phase shrinks to sign-in, user ESP and Hello — minutes instead of hours, and far fewer moving parts at the user's desk.
* **Wired or stable network for the enrollment.** Sessions with this failure often show heavy network flapping; a docked/wired first sign-in removes a whole error class.
* **Defer Windows Update over the enrollment window.** A feature or quality update landing mid-enrollment adds forced reboots exactly where the identity flow is most fragile.

## Still stuck?

Collect a [diagnostics package](diagnostics-and-log-collection.md) and use **Report Session** on an affected session — the `dsregcmd` output and User Device Registration events above are exactly the context that makes the diagnosis quick. If your sessions instead start late or never appear at all, that is the other hybrid failure mode: [Hybrid Join: sessions start late — or never](hybrid-join-late-device-registration.md).
