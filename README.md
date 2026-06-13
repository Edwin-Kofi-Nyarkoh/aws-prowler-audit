# AWS Cloud Security Audit — Prowler

## Project Description

This project is a real security audit of an AWS account
using Prowler — an open source cloud security tool used
by professional security engineers worldwide to find
misconfigurations and compliance violations before
attackers do.

The audit scanned IAM, S3, CloudTrail, and VPC against
the CIS AWS Foundations Benchmark and AWS Foundational
Security Best Practices. Every finding is documented with
a plain-English explanation of what the problem is, what
risk it creates, and how it would be fixed.

This was done on Kali Linux as part of a self-directed
cloud security learning path targeting a career as a
Cloud Security Engineer.

---

## What Is Prowler?

Prowler is an industry-standard open source tool that
automatically scans AWS accounts for security weaknesses.
It runs hundreds of checks across IAM, S3, EC2,
CloudTrail, VPC, and more — flagging anything that does
not meet security best practices or compliance standards.

It is the same tool used by professional cloud security
teams and penetration testers in real companies.

---

## Scan Summary

| Total Checks | Passed | Failed | Pass Rate |
|---|---|---|---|
| 92 | 27 | 65 | 28.42% |

| Severity | Count |
|---|---|
| Critical | 1 |
| High | 23 |
| Medium | 14 |
| Low | 27 |
| Pass | 27 |

![Prowler Dashboard](screenshots/your-dashboard-screenshot.png)

Full findings report: [FINDINGS.md](FINDINGS.md)

---

## Services Scanned

| Service | Status | Critical | High | Medium | Low |
|---|---|---|---|---|---|
| IAM | FAIL (17) | 0 | 3 | 8 | 6 |
| S3 | FAIL (13) | 1 | 3 | 5 | 4 |
| CloudTrail | FAIL (37) | 0 | 17 | 1 | 19 |
| VPC | FAIL (1) | 0 | 0 | 1 | 0 |

---

## Key Findings

### Critical — S3 Bucket Publicly Accessible
The S3 bucket `edwin-kofi-nyarkoh.aws-static-site` is
publicly accessible via its bucket policy. This is
intentional as it serves a static website, but it is
documented here because any sensitive file accidentally
uploaded would be publicly readable. Public access is
scoped to GetObject only.

![S3 Finding](screenshots/your-s3-finding-screenshot.png)

### High — Root Account Used Recently
The AWS root account was used within the last 24 hours.
Root should only be used for account-level tasks. All
daily work should use a dedicated IAM user instead.

### High — No CloudTrail in 16 Regions
CloudTrail was only logging in one region. 16 other
regions had no coverage — meaning any attacker activity
in those regions would go completely undetected.

![CloudTrail Finding](screenshots/your-cloudtrail-screenshot.png)

### High — MFA Not Enabled on IAM User
The prowler-scanner IAM user has long-lived access keys
with no MFA enabled. If those keys were leaked, an
attacker would have immediate access with no second
factor to stop them.

### Medium — Weak IAM Password Policy
No password complexity or expiration policy is configured.
No requirements for uppercase, numbers, symbols, or
90-day rotation.

---

## What Was Correctly Secured

- Root account has hardware MFA enabled
- Root account has no access keys
- No IAM user has AdministratorAccess policy
- S3 bucket ACLs are disabled
- S3 bucket is not publicly writable
- No IAM policy with unrestricted admin access

---

## Environment

- Tool: Prowler v5.30.1
- OS: Kali Linux (VirtualBox)
- AWS Region: eu-west-1
- Date: June 2025
- Scan Command:

prowler aws --service iam s3 cloudtrail vpc 

--output-formats html json-ocsf 

--output-directory ~/prowler-results

---

## Tools Used

Prowler · AWS CLI · Python · AWS IAM · Kali Linux · Git