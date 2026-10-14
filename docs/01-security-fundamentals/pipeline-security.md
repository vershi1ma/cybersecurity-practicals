# DevSecOps: scanning and hardening a real pipeline

**Phase 1, parallel track.** The code lives in [Cloud-DevOps](https://github.com/vershi1ma/Cloud-DevOps); this is the security write-up.

**Goal:** add automated security checks to the pipeline that deploys my own apps (Docker images on k3s, Terraform for AWS), fix what they find, and make them able to fail a build.

## What was added

Three scanners on GitHub-hosted runners, read-only permissions, every action pinned to a full commit ID:

| Scanner | Looks at | Workflow |
|---|---|---|
| gitleaks | leaked secrets in the full git history | [secret-scan.yml](https://github.com/vershi1ma/Cloud-DevOps/blob/main/.github/workflows/secret-scan.yml) |
| Checkov | Terraform, Kubernetes manifests, Dockerfiles | [checkov-scan.yml](https://github.com/vershi1ma/Cloud-DevOps/blob/main/.github/workflows/checkov-scan.yml) |
| Trivy | known vulnerabilities in the three container images | [trivy-scan.yml](https://github.com/vershi1ma/Cloud-DevOps/blob/main/.github/workflows/trivy-scan.yml) |

## Results

| | Start | End |
|---|---|---|
| Checkov findings (Dockerfiles, Kubernetes, Terraform) | 6 + 66 + 16 = 88 | 0 unexplained |
| Trivy serious fixable flaws | 3 (nginx image) | 0 |

"Unexplained" matters: every remaining finding is either fixed or recorded with a reason in [.checkov.yaml](https://github.com/vershi1ma/Cloud-DevOps/blob/main/.checkov.yaml).

## What was fixed

- **Containers ran as root.** Backend and frontend now run as a non-root user, enforced in the image and by the cluster.
- **Kubernetes hardening:** read-only filesystem, no privilege escalation, all Linux capabilities dropped, resource limits, health probes, no auto-mounted API credential. Verified live: a file write fails and `id` shows the unprivileged user.
- **Three HIGH CVEs in the nginx image** cleared by upgrading packages at build time, confirmed by a clean re-scan. nginx itself still runs as root (it must bind port 80), recorded as an inline exception.

## Decisions a reviewer should know

- **Accepted-risk register:** 22 global skips, each with a reason (for example the S3 website bucket is public on purpose). Trade-off: a global skip covers every resource, so per-resource skips are the better practice.
- **Report first, block second:** the scanners ran report-only to set a baseline, then became blocking.
- **Alarms tested, not just configured:** a throwaway pull request with a Dockerfile lacking `USER` turned Checkov red, and an end-of-life Alpine image added to Trivy turned that job red. Branch deleted unmerged.
- **Preview before change:** every manifest went through `kubectl diff` before `kubectl apply`.

## Real problems hit

- **Drift:** the pipeline never applied the Kubernetes files, so repo and cluster could disagree. Hand-made copies on the server were saved aside and reconciled.
- **A check passed for the wrong reason:** "no public IP" passed only because the setting was missing from the file.
- **Zero findings is not always safe:** Trivy found almost nothing in an end-of-life OS because its distribution no longer publishes fixes.

## Not done yet

Network policies, a non-root nginx image, tighter EC2 settings, branch protection with required checks (a red run is visible today, not enforced), and a registry with fixed image tags.
