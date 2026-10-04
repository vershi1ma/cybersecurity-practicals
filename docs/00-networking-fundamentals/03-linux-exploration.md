# Exploring My Own Server: Networking, Users, Processes, and Logs

**Goal:** understand the Linux server underneath my website from first principles, using it as a real lab rather than reading theory alone.

## Subnetting and routing

Found my server's address block (`172.31.32.0/20`, ~4,096 addresses) by reading `ip -4 -br addr`, then confirmed it against `ip route`: traffic to other machines in the block goes direct, everything else routes through the gateway. Matched this against AWS's actual VPC layout to check the two agreed.

## Users and permissions

Explored `whoami`, `id`, and the ownership/permission bits on `/etc/passwd` vs `/etc/shadow` (world-readable vs root-only, and why). Ran a full account lifecycle lab on a throwaway user: created it with no privileges, confirmed `sudo -l` denied it, granted sudo via the `wheel` group, confirmed the grant with `sudo -l` again, then removed the account with `userdel -r`. Confirms I can read and act on Linux's permission model rather than just describe it.

## Processes, services, and listening ports

Used `ps`, `ss -tlnp`, and `systemctl` to build a picture of what's actually running (Apache, sshd, the SSM agent) and which ports are listening. Cross-checked `ss` output against the AWS security group rules directly: ports 80/443 open to the internet, nothing open on 22, matching what later recon work with nmap also confirmed from outside.

Then went through the services layer. `systemctl list-units` showed 20 running services, recorded as the baseline for later comparison. There are no hand-made service files in `/etc/systemd/system`, and comparing enabled services against the distribution's defaults showed only `httpd` and `cloud-init` as non-default, which is what I expected. Practised stopping and starting a harmless service (`atd`) to see the lifecycle. Noted several services that may be unnecessary here (`sshd`, `atd`, `gssproxy`, `libstoragemgmt`, `systemd-homed`) as hardening candidates to test and disable in Phase 2. Nothing was disabled yet.

## Logs

Walked through `/var/log` and `journalctl`, then read Apache's real access log line by line, summarising it with `awk`, `sort`, and `uniq`. Two findings about what logs do and don't show: Session Manager sessions do not appear in `wtmp`, so AWS CloudTrail is the real record of who connected; and `sudo` commands are visible in `journalctl`.

The access log held real hostile and routine traffic:
- A secret-hunting bot asking for `.env`, `wp-config.php` and similar files. All returned `404`, nothing exposed.
- A research scanner (Modat) whose reverse DNS matched its claimed identity, so harmless.
- A request for a known PHP-framework exploit path that returned `200` with only 142 bytes, the same size as my home page. PHP isn't installed here, so nothing could execute: Apache simply served the default page. A `200` status alone doesn't mean an attack worked, so I compared response size against the normal page.

This is the same log data that later fed the recon write-up's hidden-path checks.

## Why this matters

Together these four areas are the baseline every later phase builds on: you can't harden, detect, or investigate a server you don't already understand. Each piece here was verified hands-on against the real server, not assumed from documentation.
