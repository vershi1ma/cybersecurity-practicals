# Threat Model: Cloudlearner-server

A structured risk assessment of the EC2 instance hosting this project's lab work and portfolio site, built using the CIA triad, STRIDE, and MITRE ATT&CK.

## Methodology

Assets were scored on **likelihood × impact** (1–3 each, so 1–9 overall), using STRIDE (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege) to identify threats per asset. Integrity was treated as the most damaging loss for this server: its primary purpose is serving trusted content, so undetected tampering would do more harm than a temporary outage.

## Risk register

| # | Asset | Main threat | Score | Band |
|---|---|---|---|---|
| 3 | AWS account access | Stolen keys, account suspension | 9 | High |
| 5 | iPad credentials | Long-lived GitHub token, broad scope | 6 → **2** | High → Low (fixed) |
| 6 | Hostname / Elastic IP | Free DDNS hostname lapses without confirmation | 6 | High |
| 1 | Website content | Tampering (defacement) | 4 | Medium |
| 8 | The server itself | Compromise via unpatched service | 4 | Medium |
| 2 | Server's AWS role | EC2 + S3 full access, large blast radius | 3 | Medium |
| 9 | Docker server / self-hosted runner | Malicious PR running on the runner | 3 | Medium |
| 4 | Terraform config and state | Drift, accidental exposure | 2 | Low |
| 7 | Logs | Tampering, deletion | 2 | Low |

## Findings from real traffic, mapped to ATT&CK

Two genuine events, captured and analysed from the server's own Apache logs, map cleanly onto MITRE ATT&CK techniques:

- **Automated secret-hunting scan** — repeated requests for `.env`, `wp-config.php` and similar files, all returning 404. Maps to **T1595 – Active Scanning** (Reconnaissance). Untargeted, low signal on its own.
- **ThinkPHP exploit attempt** — a request crafted for a known ThinkPHP remote-code-execution vulnerability, returned a 200 with a body identical in size to the homepage. It failed only because the server runs no PHP, not because of any active defense. Maps to **T1190 – Exploit Public-Facing Application** (Initial Access). Worth treating as a real finding rather than a non-event, since the failure was incidental.

## NIST CSF coverage

Mapping current work against NIST's five functions surfaces a clear gap:

| Function | Status |
|---|---|
| Identify | Covered — this threat model |
| Protect | Covered — patching, secret scanning, least-privilege IAM, token hardening |
| Detect | Partial — manual log review only, no alerting |
| Respond | Not yet built |
| Recover | One reactive fix only (hostname renewal after an outage) |

Detect, Respond and Recover are the explicit focus of Phase 4 (Detection and Incident Response).

## Mitigations completed

- **GitHub token (asset 5):** replaced a broad, non-expiring classic token with a fine-grained, repo-scoped, 90-day-expiring token, moved out of plaintext notes into a credential manager. Re-scored 6 → 2.
- **TLS certificate renewal:** `certbot` was installed with working renewal logic but no schedule triggering it. Added a cron job to run renewal twice daily, closing a gap that would otherwise have caused an outage identical in kind to the hostname lapse that hit asset 6.
- **Secret scanning:** automated gitleaks scan added to CI across all three repositories in scope.

## Planned next

- Fine-grained token rotation reminder (set, pending first renewal)
- Replicate the gitleaks workflow pattern for Trivy / Checkov (Phase 1 parallel track)
- Formal Respond/Recover capability, built out in Phase 4
