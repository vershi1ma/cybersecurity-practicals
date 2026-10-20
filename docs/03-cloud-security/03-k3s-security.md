# Kubernetes (k3s) security review

**Scope:** a single-node k3s cluster on a small EC2 instance running a frontend, a backend and a demo web app, plus a self-hosted CI runner. Identifiers are left out on purpose.

## Findings

| Area | Finding | Severity | Outcome |
|---|---|---|---|
| API and kubelet | Anonymous requests refused (401) | Pass | None needed |
| RBAC | Only the built-in cluster-admin binding; the default service account can read nothing useful | Pass | None needed |
| Pod hardening | Frontend and backend run non-root with read-only filesystems; the demo nginx pod runs as root (confirmed with `id`, uid 0) | Medium | Deferred: nginx needs root for port 80; fix is a non-root image on 8080 |
| Pod network | Any pod could call the backend | Medium | **Fixed** with a NetworkPolicy |
| Secrets | Base64 only; encryption at rest is off (k3s default); no application secrets stored | Medium | Deferred: enable at rebuild |
| SSH | sshd running on both servers, unreachable (no inbound rule) and unused (no logins in 16 days on the web server) | Low | **Fixed**: stopped and disabled |

The root pod has compensating controls: internal-only service, no API token mounted, privilege escalation off, seccomp on, resource limits set.

## NetworkPolicy: before and after

| Test | Before | After |
|---|---|---|
| Unlabelled stranger pod calls the backend | Answered | Connection refused |
| Pod labelled as frontend calls the backend | Answered | Answered |

The policy file lives in the repo. It was applied by hand: the pipeline never applies manifests, so nothing restores the rule if it is deleted.

## What I learned

- A label is only a name tag. A policy stops a compromised pod, and RBAC stops someone creating a forged one, so each needs the other.
- Base64 is encoding, not encryption: I decoded a test secret with one command. Protect secrets by controlling who can read them, then encrypt the disk copy.
- Accepted risks are written down with a reason (in the Dockerfile skip comment and the scanner config), so a green scan means "known", not "unseen".

## Honest limits

Not covered: outgoing traffic rules, ingress rules for the other pods, a namespace-wide default deny, other namespaces. The real frontend-to-backend flow was not retested after the policy; a lookalike pod was used.
