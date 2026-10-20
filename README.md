# Cybersecurity Practicals

## Overview
An evolving cybersecurity portfolio built alongside my cloud engineering work, aimed at cloud security and DevSecOps roles. Every phase is hands-on in a real AWS lab and documented with what was built and the real problems hit along the way. It grows one phase at a time, so the sections below only describe work that is finished. Companion to my [Cloud & DevOps portfolio](https://github.com/vershi1ma/Cloud-DevOps).

## Architecture
The lab is an Amazon Linux 2023 EC2 instance inside an AWS VPC, reached only through Systems Manager Session Manager (IAM role-based, no inbound SSH rule in the security group). Packets on its network interface are recorded with tcpdump, read back with tshark, and studied against real traffic, including unsolicited internet scanners: DNS lookups through the AWS VPC resolver (UDP 53), TCP connection setup, and plain HTTP compared with TLS-encrypted HTTPS. The server is kept patched against Amazon Linux's dated release model, with a snapshot taken before each change. Later phases extend the same lab with hardening, cloud security services, and detection tooling. The same security checks also run in the CI pipeline of my Cloud and DevOps portfolio: secret scanning, Checkov and Trivy, set to block on new findings.

## Skills demonstrated
DNS (resolution, TTL, UDP port 53) · TCP vs UDP · TCP three-way handshake · HTTP vs HTTPS (TLS encryption, SNI leakage) · Packet capture and analysis (tcpdump, tshark display filters) · traffic attribution (reverse and forward-confirmed DNS) · dig and curl · OSI model mapped to real traffic · AWS Systems Manager Session Manager · Linux command line · nmap port scanning (filtered/open/closed) · HTTP header and method enumeration (curl -I, OPTIONS) · security findings write-up (version banner, TRACE/XST, missing security headers) · Linux user and permission management (sudo, groups, account lifecycle) · process and service inspection (ps, ss, systemctl, enabled-vs-default baseline) · log analysis (journalctl, Apache access logs) · Linux patching (dnf, Amazon Linux 2023 release model, snapshot-before-change, kernel reboot verification) · CIA triad and threat modeling (risk scoring, STRIDE) · MITRE ATT&CK technique mapping (T1595, T1190) · NIST CSF function mapping and gap analysis · DevSecOps pipeline security (gitleaks, Checkov, Trivy, accepted-risk register, container and Kubernetes hardening, scanners proven with a failing test pull request) · AWS account audit (IAM, CloudTrail, GuardDuty, Config) with a tested alert path · attack-then-fix labs (S3 exposure, IAM privilege escalation, permissions boundary, security groups) · Kubernetes security review (RBAC, NetworkPolicy, secrets, pod hardening)

## Roadmap
- Phase 0: Networking & Systems Fundamentals *(complete)*
- Phase 1: Security Fundamentals & Frameworks (CIA triad, MITRE ATT&CK, NIST CSF, CIS) *(complete)*
- Phase 2: Hardening (Linux, SSH, firewalls, logs, patching)
- Phase 3: Cloud Security (IAM, S3 exposure, GuardDuty, Security Hub, CloudTrail, Kubernetes security) *(complete)*
- Phase 4: Detection & Incident Response
- Phase 5: Offensive Essentials (own lab only)
- Phase 6: Capstone

Running alongside: DevSecOps scanning in CI pipelines (secret scan, Checkov and Trivy now blocking, see the Phase 1 write-up) and security automation scripts.

## Detailed write-ups
- Phase 0 — Networking & Systems Fundamentals
  - [Watching a web request travel (DNS, TCP, HTTP vs HTTPS)](docs/00-networking-fundamentals/01-packet-capture-lab.md)
  - [Reconnaissance against my own web server (nmap + curl)](docs/00-networking-fundamentals/02-recon-nmap-curl.md)
  - [Exploring my own server: networking, users, processes, and logs](docs/00-networking-fundamentals/03-linux-exploration.md)
  - [Identifying unknown traffic in a packet capture (tshark)](docs/00-networking-fundamentals/04-traffic-analysis-tshark.md)
  - [Patching Amazon Linux 2023: why "no updates" isn't the whole answer](docs/00-networking-fundamentals/05-patching-amazon-linux.md)

- Phase 1 — Security Fundamentals & Frameworks
  - [Threat model: Cloudlearner-server (CIA, STRIDE, ATT&CK, NIST CSF)](docs/01-security-fundamentals/threat-model.md)
  - [DevSecOps: scanning and hardening a real pipeline (gitleaks, Checkov, Trivy)](docs/01-security-fundamentals/pipeline-security.md)

- Phase 3 — Cloud Security
  - [AWS account audit: who can do what, and how fast a mistake is detected](docs/03-cloud-security/01-aws-account-audit.md)
  - [Attack-then-fix labs: S3 exposure, IAM escalation, security groups](docs/03-cloud-security/02-attack-then-fix-labs.md)
  - [Kubernetes (k3s) security review: RBAC, NetworkPolicy, secrets](docs/03-cloud-security/03-k3s-security.md)

Each write-up covers what was done and the real problems hit and fixed along the way.

Full tool inventory, used so far and planned by phase: [TOOLS.md](TOOLS.md)
