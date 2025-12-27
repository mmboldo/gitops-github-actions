# AWS Setup Guide

This document describes the **manual AWS preparation steps** required to run the pipelines in this repository.

No AWS resources are created automatically without explicit user action.

---

## 1. AWS Account Requirements

- An active AWS account
- Billing enabled
- Familiarity with basic AWS services

⚠️ **Costs apply** when running this project.  
Destroy resources after use.

---

## 2. IAM User for GitHub Actions

Create a dedicated IAM user for CI/CD.

### Minimum required permissions (example)
- EC2
- EKS
- IAM (limited)
- S3
- DynamoDB (state locking)
- ECR

Best practice:
- Create a **custom policy**
- Avoid using `AdministratorAccess`

---

## 3. S3 Bucket for Terraform State

Create:
- One S3 bucket for Terraform remote state
- Enable versioning
- Block public access

Optional:
- DynamoDB table for state locking

---

## 4. ECR Repository

Create an ECR repository to store application images.

---

## 5. EKS Cluster

You may:
- Create EKS manually
- Or allow Terraform workflows to create it

Ensure:
- kubectl access
- Proper IAM role bindings

---

## 6. Cleanup

After testing:
- Destroy Terraform-managed resources
- Delete ECR images
- Remove IAM users if no longer needed
