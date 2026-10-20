# AWS account audit and detection

**Scope:** a single personal AWS account used for learning and lab work. Audited who can do what, what is recorded, and how fast a real mistake is detected. Account identifiers are left out on purpose.

## Findings

| Area | Finding | Severity | Outcome |
|---|---|---|---|
| IAM | The daily IAM user is full admin through a group; one CLI key lives on a tablet | High | Accepted and documented: hardware-key MFA, no other keys |
| Root | Used for daily work early on; MFA on, no access keys | Medium | Daily work moved to the IAM user |
| IAM | Orphan MFA device and a redundant read-only group | Low | Device deleted, group noted |
| CloudTrail | Trail healthy (multi-region, log validation on, bucket locked); console history only keeps 90 days | Info | The trail is the long-term record |
| GuardDuty | On in one region: 440 findings, 434 samples, 6 real (public S3 access, root usage) | Low | Reviewed |
| Config | Public-read rule: 5 buckets compliant, the website bucket non-compliant by design | Accepted | Documented |

## What I built

An alert path that emails within seconds of a serious finding:

`GuardDuty finding (severity 7+) -> EventBridge rule -> SNS topic -> email`

- The topic policy only accepts messages from that one rule, so nobody else can publish to it.
- An input transformer turns the raw JSON into one readable line.
- Tested twice with sample findings; both emails arrived in seconds.

## Detection race

I made a real mistake on a throwaway bucket (public read) and timed the tools:

| Tool | Time to notice |
|---|---|
| AWS Config | about 3.5 minutes |
| GuardDuty email | about 8.8 minutes |

Config also verified the fix. Takeaway: Config is the fast configuration check, GuardDuty the slower behaviour check, and both are needed.

## Cost

GuardDuty costs about $0.14 per 30 days here. A $5 monthly budget alerts at 85%, 100% and on forecast.

## Honest limits

- Alerts cover one region only, one inbox, and high severity only.
- No alert yet for someone switching CloudTrail off.
- CloudTrail does not record S3 object reads unless data events are enabled (they are off).
- Security Hub was taught but not enabled, to avoid cost.
