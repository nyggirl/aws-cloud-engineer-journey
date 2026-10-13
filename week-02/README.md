# Week 2 — IAM & Security Foundations

## Overview

This week focused on AWS Identity and Access Management (IAM), the principle of least privilege, and IAM policy evaluation.

## Lab 2.1 — Least-Privilege S3 Policy

I created a custom IAM policy named `S3UploaderOnly-jinghan` and attached it to a dedicated IAM user named `s3-test-user`.

The policy allows two actions:

- **s3:PutObject:** Upload objects to the specified S3 bucket.
- **s3:GetObject:** Retrieve objects from the specified S3 bucket.

The permissions are restricted to `arn:aws:s3:::my-training-bucket-jinghan/*` to prevent access to objects in other buckets.

## Lab 2.2 — IAM Access Analyzer

I created an external access analyzer to identify resources that could be accessed by external principals.

The analyzer identified an existing IAM role used for GitHub Actions OIDC authentication in my Cloud Resume Challenge project. I reviewed the role's trust policy and archived the finding after confirming the intended access.

## Lab 2.3 — IAM Policy Simulator

I tested the permissions of `s3-test-user` using the IAM Policy Simulator.

| Action              | Result  |
| ------------------- | ------- |
| s3:PutObject        | Allowed |
| s3:GetObject        | Allowed |
| s3:ListAllMyBuckets | Denied  |

The results confirmed that the policy grants only the intended object-level permissions.

## Key Takeaways

I learned how IAM policies control access to AWS resources and how to apply the principle of least privilege. I also learned that AWS denies actions by default unless they are explicitly allowed, and that an explicit deny overrides an allow. Using IAM Access Analyzer and the Policy Simulator helped me understand how to identify and troubleshoot permission issues.
