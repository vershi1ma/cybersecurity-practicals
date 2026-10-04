# Tools

The tools this portfolio uses, what each is for, and where it fits. **Used** means I have run it hands-on and documented it (write-up numbers refer to `docs/00-networking-fundamentals/`). **Planned** names the phase it arrives in. The list grows with the work.

## Where tools run

- **AWS lab server** (Amazon Linux 2023) through browser Session Manager: packet capture, system inspection, patching, services, logs.
- **iPad (iSH, Alpine Linux):** AWS CLI, git, curl, nmap. iSH sandboxes networking, so nmap version and script detection (`-sV`, `-sC`) fail there; this is logged in write-up 02.
- **GUI tools and anything needing a real kernel** (Wireshark GUI, Kali, Docker) run on a cloud instance or in the browser, never on the iPad.

## Used so far

| Area | Tools | Purpose |
|---|---|---|
| Network analysis | `dig`, `curl`, `tcpdump`, `tshark` | DNS and reverse DNS, raw HTTP requests, packet capture, decoding and filtering captures (01, 02, 04) |
| Recon | `nmap`, `curl -I` | Port scan from outside vs inside, banners, HTTP methods (02) |
| Linux inspection | `ip`, `ss`, `ps`, `systemctl`, `journalctl`, `awk`, `sort`, `uniq` | Addresses and routes, listening ports, processes, services, log analysis (03) |
| Patching | `dnf`, `rpm` | Release and package queries, upgrades (05) |
| AWS | Session Manager, AWS CLI | Access without SSH; inspect and change resources, snapshots, security group checks |

## Planned

| Phase | Tools | Purpose |
|---|---|---|
| 1 Security fundamentals | `sha256sum`, `openssl`, `hashcat` / John, MITRE ATT&CK Navigator, CIS Benchmarks | Hashing vs encryption, certificates, password cracking on my own test hashes, mapping threats and hardening baselines |
| 2 Hardening | `iptables` / `ufw`, `fail2ban`, `logrotate`, Nessus or OpenVAS, `hping3` / `slowloris` | Host firewall, blocking abusive clients, log retention, vulnerability scan then re-scan, DoS lab between two isolated instances |
| 3 Cloud security | CloudTrail, GuardDuty, Security Hub, AWS Config, Trivy | API audit trail, threat detection, posture checks, container image scanning |
| 4 Detection and response | CloudTrail with Athena, Suricata or Snort, Zeek, TryHackMe / LetsDefend / CyberDefenders | Queryable audit logs, network detection rules, SOC practice, one written incident investigation |
| 5 Offensive essentials | OWASP Juice Shop, DVWA, Burp Suite, Gobuster, Nikto, `sqlmap` | Web attacks against deliberately vulnerable apps in my own lab, each with an exploit-then-fix write-up |
| Running alongside | GitHub Actions, Trivy, Checkov / tfsec, secret scanning, Python and Bash | Security scanning in CI, automation scripts |

## Safety rules

- Scan and attack only my own lab or CTF platforms.
- Vulnerable apps run only in an isolated lab VPC with inbound limited to my own IP.
- The DoS lab runs only between two isolated instances I control, never against a public server or a shared service.
