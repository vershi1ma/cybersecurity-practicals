# Exploring My Own Server: Networking, Users, Processes, and Logs

**Goal:** understand the Linux server underneath my website from first principles, using it as a real lab rather than reading theory alone.

## Subnetting and routing

Found my server's address block (`172.31.32.0/20`, ~4,096 addresses) by reading `ip -4 -br addr`, then confirmed it against `ip route`: traffic to other machines in the block goes direct, everything else routes through the gateway. Matched this against AWS's actual VPC layout to check the two agreed.

## Users and permissions

Explored `whoami`, `id`, and the ownership/permission bits on `/etc/passwd` vs `/etc/shadow` (world-readable vs root-only, and why). Ran a full account lifecycle lab on a throwaway user: created it with no privileges, confirmed `sudo -l` denied it, granted sudo via the `wheel` group, confirmed the grant with `sudo -l` again, then removed the account with `userdel -r`. Confirms I can read and act on Linux's permission model rather than just describe it.

## Processes and listening ports

Used `ps`, `ss -tlnp`, and `systemctl` to build a picture of what's actually running (Apache, sshd, the SSM agent) and which ports are listening. Cross-checked `ss` output against the AWS security group rules directly: ports 80/443 open to the internet, nothing open on 22, matching what later recon work with nmap also confirmed from outside.

## Logs

Walked through `/var/log` and `journalctl`, then read Apache's real access log line by line. Found and investigated a genuine secret-hunting bot probing for `.env` and config files (all 404s, nothing exposed), plus routine background noise from automated scanners. This is the same log data that later fed the recon write-up's hidden-path checks.

## Why this matters

Together these four areas are the baseline every later phase builds on: you can't harden, detect, or investigate a server you don't already understand. Each piece here was verified hands-on against the real server, not assumed from documentation.
