# Attack-then-fix labs on my own AWS account

**Scope:** three deliberate misconfigurations on disposable resources in my own account. Each was attacked, observed in the logs, fixed and re-tested. Nothing live was touched.

## Results

| Lab | What I broke | What happened | Fix |
|---|---|---|---|
| S3 exposure | Public read on a throwaway bucket | Switching off Block Public Access alone changed nothing; adding a public read policy exposed the file | Lock back on, delete the policy |
| IAM escalation | A test user holding one permission: attach policies to itself | It gave itself full admin in under a minute | Detach, then add a permissions boundary |
| Network exposure | Audit of every security group | Only the web group faces the internet (80 and 443, no SSH); two orphan rules sat in a default group | Rules revoked, default group now has no inbound rules |

## What each lab proved

**1. S3 needs two mistakes to leak.** The lock and the bucket policy are separate controls. Exposure took both failing, so keeping the lock on protects even when a policy is wrong. The window was 9 minutes 39 seconds and CloudTrail recorded 5 management events, but no object reads: reads need data events, which are off by default.

**2. One permission can equal admin.** "May attach policies to myself" is the power to grant any other power. After adding a permissions boundary, the same attack failed with an explicit boundary error. CloudTrail names the caller, the target and the policy for the allowed attempt; the blocked one lacks the target details.

**3. Old rules outlive their reason.** The orphan rules (a database port, and SSH from one single address) pointed at things that no longer exist. They were harmless today and dangerous the day something reused those groups.

## Accepted and open

- The website bucket is public on purpose: its policy allows only reading objects, never listing.
- Every group still allows all outbound traffic.
- Some unused groups remain, kept for a planned isolated lab network.
