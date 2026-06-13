# AWS Security Audit — Findings Report

Tool: Prowler v5.30.1
Date: June 2025
Account: [REDACTED]
Services Scanned: IAM, S3, CloudTrail, VPC

---

## Summary

| Total Checks | Passed | Failed |
|---|---|---|
| 92 | 27 | 65 |

| Severity | Count |
|---|---|
| Critical | 1 |
| High | 23 |
| Medium | 14 |
| Low | 27 |

---

## Finding 1 — S3 Bucket Publicly Accessible
**Severity:** Critical
**Service:** S3
**Check:** s3_bucket_public_access

**What it means:**
The S3 bucket is accessible to anyone on the internet
via its bucket policy. This is intentional for a static
website but is flagged because any sensitive file
accidentally uploaded would be immediately public.

**Risk:**
Accidental exposure of credentials, config files, or
private data. This is the same misconfiguration behind
major cloud data breaches.

**Fix:**
Scope the bucket policy to only allow GetObject on
specific file paths. Never store sensitive files in a
public bucket. Enable access logging to monitor who
reads what.

**Note:** This is an intentional architectural decision
for a static website. Public access is scoped to
GetObject only via bucket policy.

---

## Finding 2 — Root Account Used Recently
**Severity:** High
**Service:** IAM
**Check:** iam_avoid_root_usage

**What it means:**
The AWS root account was used within the last 24 hours.
Root has unlimited access to the entire account and
should almost never be used for daily tasks.

**Risk:**
If root credentials are ever stolen, an attacker gains
complete and unrestricted control of the entire AWS
account — no IAM policy can stop them.

**Fix:**
Create a dedicated IAM admin user for daily work. Lock
away root credentials. Only use root for account-level
tasks like changing billing settings or closing the
account.

---

## Finding 3 — No CloudTrail Logging in 16 Regions
**Severity:** High
**Service:** CloudTrail
**Check:** cloudtrail_multi_region_enabled

**What it means:**
CloudTrail is AWS's audit log. It records every API
call made in the account. 16 out of 18 regions have no
CloudTrail trail enabled — meaning activity in those
regions is completely invisible.

**Risk:**
An attacker who gains access could launch EC2 instances
to mine cryptocurrency, create backdoor IAM users, or
steal data in an uncovered region — with no log of
any of it.

**Fix:**
Create one multi-region CloudTrail trail — a single
trail can cover all regions at once. Store logs in a
separate S3 bucket with Object Lock enabled so logs
cannot be deleted or tampered with.

---

## Finding 4 — MFA Not Enabled on IAM User
**Severity:** High
**Service:** IAM
**Check:** iam_user_hardware_mfa_enabled

**What it means:**
The prowler-scanner IAM user has active long-lived
access keys but no MFA device enabled.

**Risk:**
If the access keys were ever leaked in a GitHub commit,
config file, or stolen laptop, an attacker would have
immediate AWS access with no second factor required.
Attackers scan GitHub continuously for leaked keys and
exploit them within minutes.

**Fix:**
Enable MFA on all IAM users. Use temporary credentials
via IAM roles and STS instead of long-lived access keys
where possible. Rotate access keys every 90 days.

---

## Finding 5 — Weak IAM Password Policy
**Severity:** Medium
**Service:** IAM
**Check:** Multiple password policy checks

**What it means:**
The account has no strong password policy. No
requirements for uppercase letters, numbers, symbols,
minimum length of 14 characters, or 90-day expiration.

**Risk:**
Weak passwords are easier to crack through brute force
or guess through credential stuffing attacks where
attackers use lists of previously leaked passwords.

**Fix:**
Set minimum length to 14 characters. Require uppercase,
lowercase, numbers, and symbols. Set expiration to 90
days. Prevent reuse of the last 24 passwords.

---

## Finding 6 — S3 Bucket Allows Insecure HTTP
**Severity:** Medium
**Service:** S3
**Check:** s3_bucket_secure_transport_policy

**What it means:**
The S3 bucket policy does not enforce HTTPS. Requests
over plain HTTP are allowed.

**Risk:**
Data transferred over HTTP is unencrypted. An attacker
on the same network could intercept and read the data
in transit.

**Fix:**
Add a bucket policy condition that denies any request
where aws:SecureTransport is false. This forces all
traffic to use HTTPS only.

---

## What Was Correctly Configured

- Root account has hardware MFA enabled
- Root account has no access keys
- No IAM user has AdministratorAccess policy attached
- prowler-scanner access keys are less than 90 days old
- S3 bucket ACLs are disabled (modern best practice)
- S3 bucket is not publicly writable via ACL
- S3 bucket is not publicly listable
- No IAM policy grants unrestricted admin access
- AWSCloudShellFullAccess not attached to any user

---

## Conclusion

The account has a good security foundation — root MFA
is enabled, no admin policies are over-assigned, and
access keys are fresh. However, three areas need
immediate attention: the lack of multi-region CloudTrail
coverage leaves most of the account's activity invisible,
the root account is being used for daily tasks which
creates unnecessary risk, and no MFA is on the scanning
IAM user. Fixing these three High findings would
significantly improve the account's security posture.