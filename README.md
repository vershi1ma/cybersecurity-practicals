# Cybersecurity Practicals

## Overview
An evolving cybersecurity portfolio built alongside my cloud engineering work, aimed at cloud security and DevSecOps roles. Every phase is hands-on in a real AWS lab and documented with what was built and the real problems hit along the way. It grows one phase at a time, so the sections below only describe work that is finished. Companion to my [Cloud & DevOps portfolio](https://github.com/vershi1ma/Cloud-DevOps).

## Architecture
The lab is an Amazon Linux 2023 EC2 instance inside an AWS VPC, reached only through Systems Manager Session Manager (IAM role-based, no inbound SSH rule in the security group). Packets on its network interface are recorded with tcpdump and studied against real traffic: DNS lookups through the AWS VPC resolver (UDP 53), TCP connection setup, and plain HTTP compared with TLS-encrypted HTTPS. Later phases extend the same lab with hardening, cloud security services, and detection tooling.

## Skills demonstrated
DNS (resolution, TTL, UDP port 53) · TCP vs UDP · TCP three-way handshake · HTTP vs HTTPS (TLS encryption, SNI leakage) · Packet capture and analysis (tcpdump, capture filters) · dig and curl · OSI model mapped to real traffic · AWS Systems Manager Session Manager · Linux command line · nmap port scanning (filtered/open/closed) · HTTP header and method enumeration (curl -I, OPTIONS) · security findings write-up (version banner, TRACE/XST, missing security headers)

## Roadmap
- Phase 0: Networking & Systems Fundamentals *(in progress)*
- Phase 1: Security Fundamentals & Frameworks (CIA triad, MITRE ATT&CK, NIST CSF, CIS)
- Phase 2: Hardening (Linux, SSH, firewalls, logs, patching)
- Phase 3: Cloud Security (IAM, S3 exposure, GuardDuty, Security Hub, CloudTrail)
- Phase 4: Detection & Incident Response
- Phase 5: Offensive Essentials (own lab only)
- Phase 6: Capstone

Running alongside: DevSecOps scanning in CI pipelines and security automation scripts.

## Detailed write-ups
- Phase 0 — Networking & Systems Fundamentals
  - [Watching a web request travel (DNS, TCP, HTTP vs HTTPS)](docs/00-networking-fundamentals/01-packet-capture-lab.md)
  - [Reconnaissance against my own web server (nmap + curl)](docs/00-networking-fundamentals/02-recon-nmap-curl.md)

Each write-up covers what was done and the real problems hit and fixed along the way.
