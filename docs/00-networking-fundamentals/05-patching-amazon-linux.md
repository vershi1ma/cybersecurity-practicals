# Patching Amazon Linux 2023: Why "No Updates" Isn't the Whole Answer

**Goal:** bring my live web server up to date safely, and understand how Amazon Linux 2023's update model can hide a gap.

**Setup:** `Cloudlearner-server` (Amazon Linux 2023, Apache), reached through Session Manager; AWS CLI from my iPad for the snapshot.

## Finding: clean check, six weeks behind

`dnf updateinfo summary` returned nothing, even after forcing a fresh catalog download, which reads as "no pending security advisories". But Amazon Linux 2023 ships dated releases, and a server stays on its release until someone chooses to move it. My server was on `2023.12.20260817`; `2023.12.20260930` was available, with five more in between.

The advisory check only compares the server against its own release's frozen snapshot, so it cannot show fixes that live in newer releases. A silent result meant "current within an old release", not "current".

## Change process

1. **Snapshot first.** Backed up the 8 GB root volume with the AWS CLI and confirmed it showed Completed in the console before touching anything. A failed upgrade becomes a rollback instead of a rebuild.
2. **Preview.** `dnf upgrade --releasever=2023.12.20260930` prints its plan and waits for confirmation. It listed 89 packages to upgrade and 3 to install (188 MB), including `systemd` and a new kernel (`6.18.51`).
3. **Pre-reboot check.** Confirmed Apache was both running and enabled at boot, so the site would return without manual action.
4. **Reboot.** The new kernel is installed beside the old one and only takes over at restart. Until then the kernel fixes are on disk but not active.
5. **Verify.** Running kernel (`uname -r`), release number (`rpm -q system-release`), Apache active, and the site returning `200 OK` from outside the server.

## Why this matters

- Attackers read the same public advisories as defenders, so a server that falls behind keeps flaws everyone already knows about.
- The "no updates" signal was misleading precisely because it looked reassuring. Verifying the release number is the real check.
- A patch is not finished until the running system proves it: the kernel only counts after the reboot.

## Limits and follow-ups

- I did not read the per-release notes, so I cannot yet say which specific vulnerabilities this upgrade fixed. Next time I will read them before upgrading.
- The Kernel Livepatch repository is enabled on this system but I have not studied it yet.
- Routine habit going forward: compare the installed release against the newest one monthly.

## Terms used

**Patching:** installing vendor fixes for known flaws. **Release (Amazon Linux 2023):** a dated, frozen edition of the whole package set. **Deterministic upgrade:** the system only moves when you choose a new release. **Advisory:** a published notice describing one flaw and its fix. **Kernel:** the core of the operating system; a new one needs a reboot. **Snapshot:** a point-in-time copy of a disk, used here as a rollback point.
