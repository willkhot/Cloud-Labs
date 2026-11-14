# Lab 10 — Triggering a GuardDuty Finding: S3 Block Public Access Disabled

## Overview

This lab was a security scenario. A financial services company moved its customer records to Amazon S3 and turned on **Amazon GuardDuty** for threat detection. Then GuardDuty raised this finding:

```
Policy:S3/BucketBlockPublicAccessDisabled
```

My job was to show what causes it. I created an S3 bucket, turned off its **Block Public Access** settings, and also turned them off for the whole account. GuardDuty picked up both changes and created two findings: one for the bucket and one for the account.

## Objectives

- Create an S3 bucket with the default security settings
- Turn off Block Public Access on the bucket
- Turn off Block Public Access for the account
- Find and read the GuardDuty findings those changes caused
- Clean up by deleting the bucket

## Tools & Technologies

| Category | Tools |
|---|---|
| Cloud platform | Amazon Web Services (AWS) |
| Services | Amazon S3, Amazon GuardDuty (S3 Protection), AWS CloudTrail |
| Objects | S3 bucket, bucket and account Block Public Access settings, GuardDuty findings |
| Region | US East (N. Virginia) `us-east-1` |

## How GuardDuty Watches S3

GuardDuty **S3 Protection** keeps an eye on S3 by reading **CloudTrail** events. Every time someone calls an S3 API, like changing a bucket's settings or uploading, downloading, or deleting an object, CloudTrail records who did it and what changed. GuardDuty analyzes those events and creates a **finding** when something looks risky, such as a change that could lead to unauthorized access, data theft, or data deletion. An admin can then investigate and respond before the data is actually compromised.

S3 Protection works on its own once it's enabled. I didn't have to set up any logging or agents for it to catch the changes in this lab.

## Block Public Access: Two Layers

| Layer | Where | What it covers |
|---|---|---|
| Account level | S3 → Account and organization settings | Every bucket in the account, now and in the future |
| Bucket level | Bucket → Permissions | Just that one bucket |

Each layer has the same four settings, and **Block all public access** turns on all four. If either layer blocks something, it's blocked. So a bucket is only really open to the public when both layers are off and there's also a public bucket policy or ACL. Turning off Block Public Access doesn't make data public by itself, but it removes the safety net. That's why GuardDuty flags it.

## Architecture

This is the architecture.

![Block Public Access turned off at the bucket and account level, with CloudTrail and GuardDuty producing two findings](screenshots/00-architecture.png)

---

## Task 1 — Create the Bucket

### 1.1 Open S3

In the console, I searched for **S3** and opened **General purpose buckets**. The account didn't have any buckets yet.

![S3 console with no buckets](screenshots/01-s3-console.png)

![Create bucket button](screenshots/02-create-bucket-button.png)

### 1.2 Bucket settings

**Create bucket**, with these settings:

| Setting | Value |
|---|---|
| Bucket type | General purpose |
| Bucket name | `s3bucketumsl-wk` (bucket names have to be unique across all of AWS) |
| Object Ownership | ACLs disabled (bucket owner enforced), the default |
| Block Public Access | **Block all public access** on, the default |
| Bucket Versioning | Disabled, the default |
| Default encryption | SSE-S3, with Bucket Key enabled |

![Bucket type and name](screenshots/03-bucket-name.png)

The new bucket started out safe. ACLs were disabled and **Block all public access** was checked.

![Object Ownership and Block Public Access during creation](screenshots/04-object-ownership.png)

![Bucket Versioning left disabled](screenshots/05-bucket-versioning.png)

![Default encryption SSE-S3 with Bucket Key](screenshots/06-default-encryption.png)

The bucket was created.

![Bucket created](screenshots/07-bucket-created.png)

---

## Task 2 — Turn Off Block Public Access on the Bucket

**s3bucketumsl-wk → Permissions → Block public access (bucket settings) → Edit.** I unchecked **Block all public access**, which clears all four settings, and saved.

![Bucket-level Block Public Access unchecked](screenshots/08-bucket-block-public-access-off.png)

This change is the `PutBucketPublicAccessBlock` API call that leads to the `Policy:S3/BucketBlockPublicAccessDisabled` finding.

---

## Task 3 — Turn Off Block Public Access for the Account

The account-level setting was still on, and it overrides the bucket. To show the full risk, I turned it off too.

**S3 → Account and organization settings.** All four account-level settings were **On**.

![Account-level Block Public Access all On](screenshots/09-account-settings.png)

I clicked **Edit**, unchecked **Block all public access**, and clicked **Save changes**.

![Edit account settings](screenshots/10-account-edit.png)

![Account-level settings unchecked](screenshots/11-account-uncheck.png)

AWS warned that this affects every existing and new bucket in the account, and made me type `confirm`.

![Confirm the account-level change](screenshots/12-account-confirm.png)

All four account-level settings changed to **Off**.

![Account-level Block Public Access all Off](screenshots/13-account-block-public-access-off.png)

Then I uploaded a CSV file to the bucket as a stand-in for the customer records in the scenario.

![Uploading a file to the bucket](screenshots/14-upload.png)

---

## Task 4 — Review the GuardDuty Findings

**GuardDuty → Findings.** There were two new **Low** severity findings, one for each change.

![Two GuardDuty findings](screenshots/15-guardduty-findings.png)

### 4.1 Account finding

| Field | Value |
|---|---|
| Title | Amazon S3 Block Public Access was disabled for an account |
| Type | `Policy:S3/AccountBlockPublicAccessDisabled` |
| Caused by | `PutAccountPublicAccessBlock` |
| Severity | Low |
| Count | 2 (GuardDuty groups repeat events into one finding) |

![Account-level finding details](screenshots/16-finding-account.png)

### 4.2 Bucket finding

This is the finding from the scenario.

| Field | Value |
|---|---|
| Title | Amazon S3 Block Public Access was disabled for the S3 bucket s3bucketumsl-wk |
| Type | `Policy:S3/BucketBlockPublicAccessDisabled` |
| Caused by | `PutBucketPublicAccessBlock` |
| Resource | `s3bucketumsl-wk` |
| Severity | Low |

![Bucket-level finding details](screenshots/17-finding-bucket.png)

Both findings say the same thing: if this change wasn't expected, it could be a configuration mistake or a sign that someone's credentials were compromised.

### 4.3 What the security team should do

If this showed up in a real account like XYZ Corp's:

1. Check CloudTrail to see who made the change and whether it was approved
2. Turn **Block all public access** back on at the account and bucket level
3. Look for public bucket policies or ACLs on any bucket that holds customer data
4. If the change wasn't approved, treat the credentials that made it as compromised and rotate them

---

## Cleanup

I deleted the bucket so it wouldn't use any more lab credits.

![Deleting the bucket](screenshots/18-delete-bucket.png)

![Bucket deleted](screenshots/19-bucket-deleted.png)

---

## What I Learned

- Block Public Access exists at the account level and the bucket level, and either one can block public access
- Turning off Block Public Access doesn't make data public on its own, but it removes the guardrail. After that, one public bucket policy or ACL is all it takes to leak data.
- GuardDuty S3 Protection finds risky changes by analyzing CloudTrail events, with no extra setup on the bucket
- Each change produced its own finding: `Policy:S3/BucketBlockPublicAccessDisabled` for the bucket and `Policy:S3/AccountBlockPublicAccessDisabled` for the account
- A finding tells me the API call, the resource, and who made the change, which is where an investigation starts
- For buckets with sensitive data, Block Public Access should stay on, and any change to it should be treated as a security event

---

> **Note:** Screenshots are from a temporary AWS lab account. The AWS account ID (including where it appears in the finding text), my lab session username, the GuardDuty finding IDs, and the access key IDs in the findings have been redacted. The bucket was deleted after I finished.

[← Back to AWS Labs](../)
